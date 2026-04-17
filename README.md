# Insight_Engineers_Telecom_HCL_Hackathon

# 📡 Telecom CDR Data Processing using Informatica IICS

## 🚀 Project Overview
This project demonstrates an end-to-end ETL pipeline using **Informatica Intelligent Cloud Services (IICS)** to process Telecom Call Detail Records (CDR).

The pipeline reads raw CSV data, performs data cleaning, validation, enrichment, and loads the final processed data into a target system.

---

## 📂 Data Sources

### 1. CDR Source File (CDR.csv)
Contains call details:
- CallID
- Caller
- Receiver
- Duration
- CallType
- TowerID
- Timestamp

### 2. Customer Master
- Phone Number
- Customer Name
- Plan Type

### 3. Tower Master
- TowerID
- Region
- City

---

## 🔄 Mapping Design

### 🔹 Mapping 1: Data Cleaning & Transformation


Source → Aggregator → Expression → Target

Steps:


Source: Reads raw CDR CSV


Aggregator:


Removes duplicate records


Group By: CallID




Expression:


Handle null duration → default 0


Convert timestamp format


Calculate Revenue:
Revenue = Duration * 0.02


Identify international calls:
IF CallType = 'INT' THEN 1 ELSE 0




Target: Stores cleaned data



🔹 Mapping 2: Data Enrichment
Source → Lookup (Customer) → Lookup (Tower) → Router → Target
Steps:


Source: Reads cleaned data


Lookup 1 (Customer):


Join on Caller = PhoneNumber




Lookup 2 (Tower):


Join on TowerID




Router:


Valid records:


Duration ≥ 0


Caller NOT NULL


Receiver NOT NULL




Invalid records → rejected




Target:


Final enriched dataset





✅ Data Validation Rules


Remove duplicate records


Replace NULL Duration with 0


Validate phone numbers (length = 10)


Ensure Duration ≥ 0


Remove invalid records using Router



📊 Calculated Fields
FieldLogicRevenueDuration * 0.02International FlagCallType = 'INT' → 1 else 0

🧠 Transformations Used


Source Transformation


Aggregator (Deduplication)


Expression (Data Cleaning & Calculation)


Lookup (Data Enrichment)


Router (Validation)


Target (Load Data)





🛠 Tools Used


Informatica IICS


SQL Database


CSV Files



📌 Author
Insight Engineer Team
