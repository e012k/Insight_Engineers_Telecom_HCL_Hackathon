# Insight_Engineers_Telecom_HCL_Hackathon

# 📡 Telecom CDR Data Processing using Informatica IICS

### 🏆 Insight Engineers – HCL Hackathon Submission

---

## 🚀 Project Overview

The *Telecom CDR Data Processing System* is a robust end-to-end ETL pipeline built using *Informatica Intelligent Cloud Services (IICS)*.

It processes raw *Call Detail Records (CDR)* and transforms them into *validated, enriched, and analytics-ready datasets*.

---

## 🎯 Objectives

* Clean and standardize raw telecom data
* Remove duplicates and invalid records
* Enrich with Customer & Tower master data
* Generate KPIs (Revenue, International Calls %)
* Enable business analytics

---

## 📂 Data Sources

### 📄 CDR.csv

CallID, Caller, Receiver, Duration, CallType, TowerID, Timestamp

### 👤 Customer Master

PhoneNumber, CustomerName, PlanType

### 🗼 Tower Master

TowerID, Region, City

---

# 🔄 ETL ARCHITECTURE (DETAILED)

---

## 🔹 Mapping 1: Data Cleaning & Transformation

text
Source → Aggregator → Expression → Target


---

### 🔧 Step 1: Source

* Reads raw CDR file
* Data may contain:

  * NULL values
  * Duplicate CallIDs
  * Invalid duration

---

### 🔧 Step 2: Aggregator (Deduplication)

#### 📌 Purpose:

Remove duplicate records

#### ⚙️ Configuration:

* *Group By:* CallID

#### 📊 Output Fields:

* CallID
* Caller
* Receiver
* Duration
* CallType
* TowerID
* Timestamp

👉 Only *one record per CallID* will be retained

---

### 🔧 Step 3: Expression Transformation

#### 📌 1. Handle NULL Duration

sql
IIF(ISNULL(Duration), 0, Duration)


---

#### 📌 2. Standardize Timestamp

sql
TO_DATE(Timestamp, 'YYYY-MM-DD HH24:MI:SS')


---

#### 📌 3. Revenue Calculation

sql
Revenue = Duration * 0.02


---

#### 📌 4. International Call Flag

sql
IIF(CallType = 'INT', 1, 0)


---

#### 📌 5. Data Cleaning Conditions

* Trim spaces:

sql
LTRIM(Caller)
LTRIM(Receiver)


---

### 🔧 Step 4: Target

Stores cleaned data:

* Cleaned Duration
* Revenue
* International Flag

---

# 🔹 Mapping 2: Data Enrichment & Validation

text
Source → Lookup(Customer) → Lookup(Tower) → Router → Target


---

### 🔧 Step 1: Source

* Reads cleaned data from Mapping 1

---

### 🔧 Step 2: Lookup – Customer

#### 📌 Condition:

sql
Caller = PhoneNumber


#### 📊 Returns:

* CustomerName
* PlanType

#### ⚠️ Lookup Type:

* Connected Lookup
* Return default NULL if not found

---

### 🔧 Step 3: Lookup – Tower

#### 📌 Condition:

sql
TowerID = TowerID


#### 📊 Returns:

* Region
* City

---

### 🔧 Step 4: Router (VALIDATION LOGIC)

---

## ✅ Valid Group Condition

sql
NOT ISNULL(Caller)
AND NOT ISNULL(Receiver)
AND LENGTH(Caller) = 10
AND LENGTH(Receiver) = 10
AND Duration >= 0


---

## ❌ Invalid Group Condition

sql
TRUE


👉 All records not satisfying valid condition go to reject

---

### 🔧 Step 5: Targets

#### ✔ Valid Target:

* Enriched dataset
* Used for analytics

#### ❌ Reject Target:

* Stores invalid records
* Used for data quality monitoring

---

# 📊 BUSINESS USE CASES (DETAILED)

---

## ✅ 🔷 USE CASE 1: Daily Call Volume Summary

### 🎯 Objective

Calculate daily call KPIs

---

### 🔧 Steps

#### 1. Aggregator → AGG_DAILY_SUMMARY

*Group By:*

sql
TRUNC(Timestamp)


*Metrics:*

sql
TotalCalls = COUNT(CallID)
TotalDuration = SUM(Duration)
AvgDuration = AVG(Duration)


---

#### 2. Second Aggregator → AGG_CALLS_PER_CUSTOMER

*Group By:*

sql
Caller


*Metric:*

sql
CallsPerCustomer = COUNT(CallID)


---

### 🎯 Target Table: KPI_DAILY_SUMMARY

*Columns:*

text
Date
TotalCalls
TotalDuration
AvgDuration


---

## 🌍 Use Case 2: International Call Monitoring

---

### 🎯 Objective

Track international call performance

---

### 🔧 Steps

#### 1. Filter Transformation

sql
CallType = 'INT'


---

#### 2. Aggregator

* Group By: None / Region

#### Metrics:

sql
IntlCallCount = COUNT(CallID)
IntlDuration = SUM(Duration)
IntlRevenue = SUM(Revenue)


---

#### 3. Total Calls Calculation

(Using variable / separate aggregator)

sql
TotalCalls = COUNT(CallID)


---

#### 4. Expression

sql
PercentIntl = (IntlCallCount / TotalCalls) * 100


---

## 👤 Use Case 3: Customer Analytics

---

### 🎯 Objective

Analyze customer behavior

---

### 🔧 Steps

#### 1. Group By Customer

sql
Group By: CustomerName


#### Metrics:

sql
TotalCalls = COUNT(CallID)
TotalDuration = SUM(Duration)
TotalRevenue = SUM(Revenue)


---

#### 📊 Output:

* Calls per customer
* Revenue per customer
* Plan-wise usage

---

## 🗼 Use Case 4: Tower / Region Analysis

---

### 🎯 Objective

Analyze telecom traffic geographically

---

### 🔧 Steps

#### 1. Group By Region / City

sql
Group By: Region, City


---

#### 2. Aggregations

sql
TotalCalls = COUNT(CallID)
TotalRevenue = SUM(Revenue)
TotalDuration = SUM(Duration)


---

#### 📊 Output:

* Calls per city
* Revenue per region
* High traffic towers

---

# 🔁 Taskflow Design (Decision Task with Status Field Logic)

Control execution of Data Tasks using *Decision Task with status value (1 = SUCCESS, 0 = FAILED)*.

---

## 🧩 Taskflow Flow

text
Start
  ↓
Data Task 1 (Mapping Task1)
  ↓
Decision Task
   ├── Path 1 (Value = 1) → Data Task 2 (Mapping Task 2) → Success Email → End
   └── Default Path → Failure Email → End


---

## 🔧 Decision Task Configuration (Important)

### 📌 Field Selection

* *Field:* DataTask1 → Task Status

👉 This field returns numeric value:

* 1 → Success
* 0 → Failed

---

## 🔀 Path Configuration

---

### ✅ Path 1 (SUCCESS PATH)

#### 📌 Condition:

text
DataTask1.TaskStatus == 1


#### 🚀 Flow:

text
DT1 (Mapping Task1)→ Decision → DT2 (Mapping Task2)→ Success Email → End


---

### ❌ Default Path (FAILURE PATH)

#### 📌 Condition:

(No condition required — default path)

#### 🚀 Flow:

text
DT1(Mapping Task1) → Decision → Failure Email → End


---

## 📧 Email Trigger Logic

### ✅ Success Email

Triggered after:

text
DataTask1.TaskStatus == 1 AND DT2 completed


---

### ❌ Failure Email

Triggered when:

text
DataTask1.TaskStatus != 1


---

## 🚀 Final Flow

text
START
  ↓
DT1(Mapping Task1)
  ↓
Decision (TaskStatus == 1 ?)
   ├── YES → DT2(Mapping Task 1) → Success Email → END
   └── NO  → Failure Email → END


---

# 🔧 IMPLEMENTATION IN IICS

* Create Taskflow
* Add Mapping Tasks
* Add Decision Task
* Add Email Task
* Configure SMTP
* Use parameters & variables

---

# 📦 DEPLOYMENT STEPS

1. Upload files
2. Create connections
3. Build mappings
4. Create tasks
5. Configure taskflow
6. Run & monitor

---

# 🎉 CONCLUSION

This project delivers a *complete telecom ETL pipeline* with:

✔ Data Cleaning
✔ Enrichment
✔ Validation
✔ Business KPIs
✔ Automation via Email Alerts

---

## 🚀 Future Scope

* Real-time streaming
* Fraud detection
* AI-based analytics

---

## 👩‍💻 Author

*Insight Engineers Team*
HCL Hackathon Submission

---
