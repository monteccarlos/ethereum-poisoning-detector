-- ==========================================
-- Address poisoning detection on Arbitrum One
-- Dataset: bigquery-public-data.crypto_arbitrum_mainnet_us
-- Types: A (dust), B (zero), C (fake-token-like)
-- Window: October 2025
-- ==========================================

-- ------------------------
-- 1) DECLARE FIRST
-- ------------------------
DECLARE usdt_addr   STRING DEFAULT "0xfd086bc7cd5c481dcc9c85ebe478a1c0b69fcbb9"; -- USDT
DECLARE usdc_addr   STRING DEFAULT "0xaf88d065e77c8cc2239327c5edb3a432268e5831"; -- native USDC
DECLARE usdc_e_addr STRING DEFAULT "0xff970a61a04b1ca14834a43f5de4533ebddb5cc8"; -- bridged USDC.e

DECLARE dust_threshold INT64 DEFAULT 1000000;  -- 1.0 token (6 decimals)
DECLARE transfer_sig   STRING DEFAULT "0xddf252ad1be2c89b69c2b068fc378daa952ba7f163c4a11628f55a4df523b3ef";

-- ------------------------
-- 2) CREATE TEMP FUNCTIONS
-- ------------------------

-- Convert hex character to integer (0–15)
CREATE TEMP FUNCTION hex_char_to_int(c STRING) AS (
  CASE
    WHEN c BETWEEN '0' AND '9' THEN SAFE_CAST(c AS INT64)
    ELSE 10 + (UNICODE(LOWER(c)) - UNICODE('a'))
  END
);

-- Convert a 12-hex-digit string to INT64
CREATE TEMP FUNCTION hex12_to_int64(h STRING) AS (
  hex_char_to_int(SUBSTR(h, 1, 1)) * 17592186044416 +  -- 16^11
  hex_char_to_int(SUBSTR(h, 2, 1)) * 1099511627776  +  -- 16^10
  hex_char_to_int(SUBSTR(h, 3, 1)) * 68719476736    +  -- 16^9
  hex_char_to_int(SUBSTR(h, 4, 1)) * 4294967296     +  -- 16^8
  hex_char_to_int(SUBSTR(h, 5, 1)) * 268435456      +  -- 16^7
  hex_char_to_int(SUBSTR(h, 6, 1)) * 16777216       +  -- 16^6
  hex_char_to_int(SUBSTR(h, 7, 1)) * 1048576        +  -- 16^5
  hex_char_to_int(SUBSTR(h, 8, 1)) * 65536          +  -- 16^4
  hex_char_to_int(SUBSTR(h, 9, 1)) * 4096           +  -- 16^3
  hex_char_to_int(SUBSTR(h,10, 1)) * 256            +  -- 16^2
  hex_char_to_int(SUBSTR(h,11, 1)) * 16             +  -- 16^1
  hex_char_to_int(SUBSTR(h,12, 1))                     -- 16^0
);

-- ------------------------
-- 3) MAIN QUERY
-- ------------------------

WITH raw_transfers AS (
  -- 1) Pull all ERC-20 Transfer logs in October 2025
  SELECT
    l.block_timestamp,
    l.block_number,
    l.transaction_hash,
    l.log_index,
    LOWER(l.address) AS token_address,
    LOWER(CONCAT("0x", SUBSTR(l.topics[OFFSET(1)], 27, 40))) AS from_address,
    LOWER(CONCAT("0x", SUBSTR(l.topics[OFFSET(2)], 27, 40))) AS to_address,
    LPAD(SUBSTR(COALESCE(l.data, "0x0"), 3), 64, "0") AS vhex
  FROM `bigquery-public-data.goog_blockchain_arbitrum_one_us.logs` AS l
  WHERE ARRAY_LENGTH(l.topics) >= 3
    AND l.topics[OFFSET(0)] = transfer_sig
    AND l.block_timestamp >= TIMESTAMP("2025-10-30")
    AND l.block_timestamp <  TIMESTAMP("2025-11-01")
),

decoded_transfers AS (
  -- 2) Decode ERC-20 value (just enough for 0 / dust / ≥1)
  SELECT
    block_timestamp,
    block_number,
    transaction_hash,
    log_index,
    token_address,
    from_address,
    to_address,
    vhex,
    CASE
      WHEN vhex IS NULL OR vhex = REPEAT('0', 64) THEN 0
      WHEN REGEXP_CONTAINS(SUBSTR(vhex, 1, 52), r'^0+$') THEN
        hex12_to_int64(SUBSTR(vhex, 53, 12))
      ELSE dust_threshold + 1
    END AS value_raw
  FROM raw_transfers
),

classified_transfers AS (
  -- 3) Canonical stablecoins vs everything else
  SELECT
    block_timestamp,
    block_number,
    transaction_hash,
    log_index,
    token_address,
    CASE
      WHEN token_address = LOWER(usdt_addr) THEN "USDT"
      WHEN token_address IN (LOWER(usdc_addr), LOWER(usdc_e_addr)) THEN "USDC"
      ELSE "OTHER"
    END AS token_symbol,
    from_address,
    to_address,
    value_raw,
    SAFE_DIVIDE(CAST(value_raw AS FLOAT64), 1000000.0) AS value_tokens,
    token_address IN (
      LOWER(usdt_addr),
      LOWER(usdc_addr),
      LOWER(usdc_e_addr)
    ) AS is_canonical
  FROM decoded_transfers
),

canonical_transfers AS (
  SELECT * FROM classified_transfers WHERE is_canonical
),
fake_transfers AS (
  SELECT * FROM classified_transfers WHERE NOT is_canonical
),

-- 4) Legit + suspect (dust/zero) for canonical tokens
canonical_legit AS (
  SELECT *
  FROM canonical_transfers
  WHERE value_raw >= dust_threshold
),

canonical_suspect AS (
  SELECT
    *,
    CASE
      WHEN value_raw = 0 THEN "zero"
      WHEN value_raw < dust_threshold THEN "dust"
      ELSE "other"
    END AS suspect_type,
    CASE
      WHEN value_raw = 0 THEN from_address
      ELSE to_address
    END AS victim_addr,
    CASE
      WHEN value_raw = 0 THEN to_address
      ELSE from_address
    END AS phishing_addr
  FROM canonical_transfers
  WHERE value_raw < dust_threshold
),

-- 5) Fake-token-like suspect (type C)
fake_suspect AS (
  SELECT
    f.*,
    "fake_token" AS suspect_type,
    from_address AS victim_addr,
    to_address   AS phishing_addr
  FROM fake_transfers f
  WHERE value_raw > 0
),

-- 6) Match canonical suspicious transfers (dust/zero) to prior legit canonical transfers
matched_official AS (
  SELECT
    s.transaction_hash AS spoof_tx_hash,
    s.log_index        AS spoof_log_index,
    s.block_timestamp  AS spoof_time,
    s.block_number     AS spoof_block_number,
    l.transaction_hash AS legit_tx_hash,
    l.log_index        AS legit_log_index,
    l.block_timestamp  AS legit_time,
    l.block_number     AS legit_block_number,
    s.victim_addr      AS victim_address,
    s.phishing_addr    AS phishing_address,
    CASE
      WHEN l.from_address = s.victim_addr THEN l.to_address
      ELSE l.from_address
    END AS intended_address,
    s.suspect_type
  FROM canonical_suspect s
  JOIN canonical_legit l
    ON s.victim_addr IN (l.from_address, l.to_address)
   AND l.block_timestamp < s.block_timestamp
  WHERE
    SUBSTR(s.phishing_addr, 3, 3) =
      SUBSTR(
        CASE
          WHEN l.from_address = s.victim_addr THEN l.to_address
          ELSE l.from_address
        END,
        3, 3
      )
    AND SUBSTR(s.phishing_addr, -4) =
      SUBSTR(
        CASE
          WHEN l.from_address = s.victim_addr THEN l.to_address
          ELSE l.from_address
        END,
        -4
      )
),

-- 7) Match fake-token suspect transfers (C) to prior legit canonical transfers
matched_fake AS (
  SELECT
    s.transaction_hash AS spoof_tx_hash,
    s.log_index        AS spoof_log_index,
    s.block_timestamp  AS spoof_time,
    s.block_number     AS spoof_block_number,
    l.transaction_hash AS legit_tx_hash,
    l.log_index        AS legit_log_index,
    l.block_timestamp  AS legit_time,
    l.block_number     AS legit_block_number,
    s.victim_addr      AS victim_address,
    s.phishing_addr    AS phishing_address,
    CASE
      WHEN l.from_address = s.victim_addr THEN l.to_address
      ELSE l.from_address
    END AS intended_address,
    s.suspect_type
  FROM fake_suspect s
  JOIN canonical_legit l
    ON s.victim_addr = l.from_address
   AND l.block_timestamp < s.block_timestamp
  WHERE
    SUBSTR(s.phishing_addr, 3, 3) =
      SUBSTR(
        CASE
          WHEN l.from_address = s.victim_addr THEN l.to_address
          ELSE l.from_address
        END,
        3, 3
      )
    AND SUBSTR(s.phishing_addr, -4) =
      SUBSTR(
        CASE
          WHEN l.from_address = s.victim_addr THEN l.to_address
          ELSE l.from_address
        END,
        -4
      )
),

matched_pairs AS (
  SELECT * FROM matched_official
  UNION ALL
  SELECT * FROM matched_fake
),

confirmed_spoofs AS (
  SELECT DISTINCT spoof_tx_hash, spoof_log_index
  FROM matched_pairs
),
confirmed_legit AS (
  SELECT DISTINCT legit_tx_hash, legit_log_index
  FROM matched_pairs
),

-- 8) Payoff: victim later sends real canonical stablecoin to phishing address,
--    but now ONLY within 1000 blocks after the spoof
payoffs AS (
  SELECT DISTINCT
    t.transaction_hash AS payoff_tx_hash,
    t.log_index        AS payoff_log_index
  FROM matched_pairs p
  JOIN canonical_legit t
    ON t.from_address   = p.victim_address
   AND t.to_address     = p.phishing_address
   AND t.block_number   > p.spoof_block_number
   AND t.block_number  <= p.spoof_block_number + 1000  -- <-- 1000-block window
),

-- 9) Final labeling
labeled AS (
  SELECT
    t.block_timestamp,
    t.block_number,
    t.transaction_hash,
    t.log_index,
    t.token_symbol,
    t.from_address,
    t.to_address,
    t.value_tokens,
    t.is_canonical,
    CASE
      WHEN cl.legit_tx_hash  IS NOT NULL THEN "victim_legit_tx"
      WHEN cs.spoof_tx_hash  IS NOT NULL THEN "spoof_tx"
      WHEN po.payoff_tx_hash IS NOT NULL THEN "victim_mistake_tx"
      ELSE "benign"
    END AS tag
  FROM classified_transfers t
  LEFT JOIN confirmed_legit  cl
    ON cl.legit_tx_hash   = t.transaction_hash
   AND cl.legit_log_index = t.log_index
  LEFT JOIN confirmed_spoofs cs
    ON cs.spoof_tx_hash   = t.transaction_hash
   AND cs.spoof_log_index = t.log_index
  LEFT JOIN payoffs po
    ON po.payoff_tx_hash  = t.transaction_hash
   AND po.payoff_log_index = t.log_index
)

SELECT *
FROM labeled
ORDER BY block_timestamp;
