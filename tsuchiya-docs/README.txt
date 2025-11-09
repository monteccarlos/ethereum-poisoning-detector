This README.txt file was generated on 20250601 by Taro Tsuchiya

#
# General instructions for completing README: 
# For sections that are non-applicable, mark as N/A (do not delete any sections). 
# Please leave all commented sections in README (do not delete any text). 
#

-------------------
GENERAL INFORMATION
-------------------

1. Title of Dataset: Blockchain Address Poisoning (Companion Dataset)

#
# Authors: Include contact information for at least the 
# first author and corresponding author (if not the same), 
# specifically email address, phone number (optional, but preferred), and institution. 
# Contact information for all authors is preferred.
#

2. Author Information
<create a new entry for each additional author>

First Author Contact Information
    Name: Taro Tsuchiya
    Institution: Carnegie Mellon University
    Address: 5000 Forbes Avenue, Pittsburgh, Pennsylvania, USA, 15213
    Email: ttsuchiy@andrew.cmu.edu

Author Contact Information 
    Name: Jin-Dong Dong
    Institution: Carnegie Mellon University
    Address: 5000 Forbes Avenue, Pittsburgh, Pennsylvania, USA, 15213
    Email: jd0@cmu.edu

Author Contact Information 
    Name: Kyle Soska
    Institution: Independent
    Email: ksoska@alumni.cmu.edu

Author Contact Information 
    Name: Nicolas Christin
    Institution: Carnegie Mellon University
    Address: 5000 Forbes Avenue, Pittsburgh, Pennsylvania, USA, 15213
    Email: nicolasc@andrew.cmu.edu

---------------------
DATA & FILE OVERVIEW
---------------------

#
# Directory of Files in Dataset: List and define the different 
# files included in the dataset. This serves as its table of 
# contents. 
#

Directory of Files:
   A. Filename: address_poisoning_ethereum.sql.gz        
      Short description: All the transfers related to address poisoning including over 17 million poisoning transfers on Ethereum. The compressed file is ~2.2GB, while the original one is ~10GB.       

   B. Filename: payoff_transfers_ethereum.csv        
      Short description: all (confirmed) payoff transfers on Ethereum.        

   C. Filename: payoff_transfers_bsc.csv        
      Short description: all (confirmed) payoff transfers on BSC (Binance Smart Chain).        
      
   D. Filename: guidance.ipynb        
      Short description: a jupyter python notebook that produces the basic statistics such as the number of poisoning transfers. The notebook also includes the detailed guide on how to manually verify the payoff transfers on Etherscan. 
   
   E. Filename: requirements.txt        
      Short description: a list of python libraries required to run guidance.ipynb. 


Additional Notes on File Relationships, Context, or Content 
(for example, if a user wants to reuse and/or cite your data, 
what information would you want them to know?):              


How to restore and access the database?
1. We highly recommend using PostgreSQL to access our dataset, so install the latest version of PostgreSQL. 

2. Configure the database. Use the following command to decompress the file and restore the table in your PostgreSQL database.
```
gunzip -c address_poisoning_ethereum.sql.gz | psql -h localhost -U USERNAME -d DATABASE 
```
Please specify your PostgreSQL credentials (USERNAME, DATABASE) above.

3. (To run `guidance.ipynb`) Install the python packages (`pip install -r requirements.txt`). We recommend using the python virtual environment. 

4. Execute the jupyter notebook (`guidance.ipynb`) to learn how to compute the basic statistics (e.g., the number of poisoning transfers). 


Ethics consideration
Our dataset includes full transaction hashes and wallet addresses. 
We decided to present them as is, as obfuscating part of the transaction hash would impede reproducibility or external validation of our findings, for little to no privacy benefit: users of Ethereum (or BSC) are relying on a public blockchain, in which all records are publicly available. 
Furthermore, wallet addresses are not, on their own, considered personally identificable information, as 1) a wallet address does not, in itself, contain information directly mapping to a real-world identity, and 2) there is no one-to-one mapping to specific individuals, as a given person may hold several wallets, and a single wallet may be shared by multiple people (e.g., belonging to the same organization).

#
# File Naming Convention: Define your File Naming Convention 
# (FNC), the framework used for naming your files systematically 
# to describe what they contain, which could be combined with the
# Directory of Files. 
#

File Naming Convention: N/A


#
# Data Description: A data description, dictionary, or codebook
# defines the variables and abbreviations used in a dataset. This
# information can be included in the README file, in a separate 
# file, or as part of the data file. If it is in a separate file
# or in the data file, explain where this information is located
# and ensure that it is accessible without specialized software.
# (We recommend using plain text files or tabular plain text CSV
# files exported from spreadsheet software.) 
#


-----------------------------------------
DATA DESCRIPTION FOR: address_poisoning_ethereum.sql.gz
-----------------------------------------
<create sections for each dataset included>


1. Number of variables: 18


2. Number of cases/rows: 34,905,969


3. Missing data codes:
        Code/symbol: 'NaN', Definition: 'Not a Number'

4. Variable List

#
# Example. Name: Wall Type 
#     Description: The type of materials present in the wall type for housing surveys collected in the project.
#         1 = Brick
#         2 = Concrete blocks
#	  3 = Clay
#         4 = Steel panels


    A. Name: block_number (integer)
       Description: The block number of the transfer. 

    B. Name: tx_hash (character(66))
       Description: The transaction hash of the transfer.

    C. Name: addr (character(42))
       Description: The address of the token transferred.

    D. Name: topics_from_addr (character(42))
       Description: The sender.

    E. Name: topics_to_addr (character(42))
       Description: The receiver.

    F. Name: is_sender_victim (boolean)
       Description: True if the sender is a victim.

    G. Name: value (real)
       Description: The value transferred.

    H. Name: value_usd (real)
       Description: The value converted to USD at the time of the transfer.

    I. Name: intended_transfer (boolean)
       Description: True if the transfer is an intended transfer.
       
    J. Name: tiny_transfer (boolean)
       Description: True if the transfer is a tiny transfer.
       
    K. Name: zero_value_transfer (boolean)
       Description: True if the transfer is a zero-value transfer.
       
    L. Name: counterfeit_token_transfer (boolean)
       Description: True if the transfer is a counterfeit token transfer.

    M. Name: payoff_transfer (boolean)
       Description: True if the transfer is a payoff transfer (confirmed).

    N. Name: payoff_transfer_unconfirmed (boolean)
       Description: True if the transfer is potentially a payoff transfer (more investigation required).

    O. Name: is_not_categorized (boolean)
       Description: True if the transfer is not labeled for any categories.

    P. Name: intended_addr (character(42))
       Description: The intended address of the transfer.

    Q. Name: num_first_matched_digits (integer)
       Description: The number of first matched digits compared.
       
    R. Name: num_last_matched_digits (integer)
       Description: The number of last matched digits compared.


-----------------------------------------
DATA DESCRIPTION FOR: payoff_transfers_ethereum.csv (payoff_transfers_bsc.csv)
-----------------------------------------
<create sections for each dataset included>


1. Number of variables: 11


2. Number of cases/rows: 1,738 (4,895)


3. Missing data codes: The dataset does not include any missing data.


4. Variable List: The variables are the same as address_poisoning_ethereum.sql.gz. However, this data does not contain the category of the transfers (e.g., intended_transfer) because all of them are payoff transfers. 


--------------------------
METHODOLOGICAL INFORMATION
--------------------------

#
# Software: If specialized software(s) generated your data or
# are necessary to interpret it, please provide for each (if
# applicable): software name, version, system requirements,
# and developer. 
#If you developed the software, please provide (if applicable): 
#A copy of the software’s binary executable compatible with the system requirements described above. 
#A source snapshot or distribution if the source code is not stored in a publicly available online repository.
#All software source components, including pointers to source(s) for third-party components (if any)

1. Software-specific information:
<create a new entry for each qualifying software program>

Name: PostgreSQL
Version: 16.8
System Requirements: Linux
Open Source? (Y/N): Y

(if available and applicable)
Executable URL:
Source Repository URL:
Developer:
Product URL:
Software source components:


Additional Notes(such as, will this software not run on 
certain operating systems?):


#
# Equipment: If specialized equipment generated your data,
# please provide for each (if applicable): equipment name,
# manufacturer, model, and calibration information. Be sure
# to include specialized file format information in the data
# dictionary.
#

2. Equipment-specific information:
<create a new entry for each qualifying piece of equipment>

Manufacturer: Dell Inc.
Model: PowerEdge R6615

(if applicable)
Embedded Software / Firmware Name: 1.1.3
Embedded Software / Firmware Version: Fri 2023-01-20
Additional Notes:

#
# Dates of Data Collection: List the dates and/or times of
# data collection.
#

3. Date of data collection (single date, range, approximate date): 20240701 - 20240831 (the dataset contains information on 20220701 - 20240630)

