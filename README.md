# ethereum-poisoning-detector
**Cross-chain Ethereum, Optimism, and Arbitrum address-poisoning detector using full-block analysis and publicly available blockchain datasets on BigQuery.**

This project extracts and analyzes blockchain transaction data to detect **address poisoning attacks** — attacks where malicious actors send tiny “dust” transactions from **look-alike addresses** in order to trick users into copying the wrong address in their wallet history.  
The repository includes detection heuristics, analysis notebooks, and scripts for extracting and inspecting full-block data across Ethereum and major L2s.

Final Paper: https://drive.google.com/file/d/1Zs2CU_jU7XPnMN6hu0lBzPQWClx-wN2P/view?usp=sharing
