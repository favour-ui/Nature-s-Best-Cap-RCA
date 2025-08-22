# Nature's Best Drink leakyCap RCA

## Context and problem
In the same operational week at the Onitsha plant, Nature Best Juice Co. experienced an unusual spike in Leaky Cap defects across multiple lines, concentrated on July 2–3, 2025. The surge exceeded normal levels, disrupted production, and signaled potential issues in the capping process, materials, or controls that need urgent investigation.

## Problem Statement
During the same operational week at the Onitsha plant, Nature Best Juice Co. experienced an unusual and sharp spike in Leaky Cap defects across multiple production lines. The issue was concentrated on Wednesday, July 2, 2025, and Thursday, July 3, 2025, significantly exceeding defect levels recorded on other days within the week. This sudden rise in defective caps disrupted normal production performance and highlighted a potential underlying issue within the capping process, materials, or operational controls that requires immediate investigation and resolution.

## who are the stakeholders
 - **Procurement Manager** → suspects a bad batch from a supplier 
 - **Quality Control Lead** → needs clarity on whether this was a system-wide issue 
 - **Line Managers** → want to isolate the problem quickly to resume normal production

## BUSINESS IMPACT
From the 2 days of leaky cap issues (July 2–3):
 - 3,355 leaky caps recorded, with thousands more in the following week
 - Daily losses jumped to 34.3% compared to just 2.1% the previous week
 - Financial losses totaled about ₦45,000 in 2 days, versus only ₦6,000 the week before

## Data Exploration and Schema Design
We worked on a structured production dataset built around a star schema with a central fact table called "FactProductionEvent"
- **DimDate** -- This is the calendar table
- **DimSupplier** -- This table contains information about the companies that supply us with empty bottles
- **DimCapBatch** -- This table provides information about each delivery (batch) of bottle caps we 
receive. 

## ER DIAGRAM

The ER diagram is use to visualize table relationships and track foreign keys used during analysis.

<img width="502" height="404" alt="ER diagram" src="https://github.com/user-attachments/assets/94f76626-16ae-4a94-88d7-0950338b4ef4" />

## Data cleaning phase process

**PHASE 1**

**STANDARDIZING NUMERICS COLUMNS**
  
The cleaning are done on the following column where we noticed unusual Numerics values,symbols and text.
Also note that this cleaning are carry out before we carry out our analysis and RCA
 - **Timestamp** -- are critical for trend analysis, cleaning them first ensured that sorting, filtering and comparisons worked properly. We fixed malformed and inconsistent formats, recast from text to datetime.
 - **JuiceTemperatureC_In** -- this column are neccessary for the determinations of the various temperatgure of the juice as at when produced. it contains some unusual rows like 'WAY TOO HOT','SENSOR_BROKEN','COLD!','HOT!'
 - **ActualCapTorque_Nm** -- The actual "tightness" measured for the cap on this bottle, in Newton
meters.
 - **AmbientTemperatureC_Line** -- The air temperature around the production line when this 
bottle was processed, in degrees Celsius.  

**PHASE 2**

   **STANDARDIZING TEXT COLUMNS**

- **Defect Result(String):** this column tells us the kind of problem (if any) was found with this bottle: "None" (no defect), 
"Underfilled" (not enough juice), "Leaky_Cap" (the cap leaks), or "Both" (both underfilled and leaky). so all other similar rows are group and categorize into this category
- **LeakTestResult(String):** The outcome of the test to see if the bottle leaks: "Pass" (it doesn't leak) or 
"Fail" (it leaks).

 **PHASE 3**

   **REMOVING DUPLICATES**
We tested key columns to identify duplicate values in the dataset. After analysis, we removed all duplicate rows. In total, over 32,000 duplicate records were deleted, ensuring the dataset is accurate, consistent, and ready for reliable analysis.
  **Steps:**
1. Detect duplicates with COUNT(*)
2. Create an index to speed up the update
3. Use a CTE with ROW_NUMBER() to retain one copy of each duplicate set
4. Delete all extra duplicate rows

 - **Code Used for Testing and Deleting all Duplicate** 

```sql
SELECT ProductionEventSK,BottleID_Natural,Timestamp, COUNT(*)
FROM FactProductionEvent
GROUP BY ProductionEventSK,BottleID_Natural,`Timestamp`
HAVING COUNT(*) > 1;

SELECT ProductionEventSK,BottleID_Natural,Timestamp 
FROM factproductionevent
WHERE ProductionEventSK IN (275543,256831,160069)
ORDER BY ProductionEventSK;

-- NOW CREATE INDEX TO QUICKEN THE UPDATE 
CREATE INDEX IDX_DUPLICATE 
ON factproductionevent (ProductionEventSK(100),BottleID_Natural(100),`Timestamp`);

SHOW INDEX FROM factproductionevent;

-- DELETING ALL DUPLICATES
WITH CTE AS (
    SELECT ProductionEventSK,BottleID_Natural,Timestamp,
           ROW_NUMBER() OVER(PARTITION BY ProductionEventSK,BottleID_Natural,Timestamp
                             ORDER BY ProductionEventSK) AS rn
    FROM factproductionevent
)
DELETE FROM factproductionevent
WHERE ProductionEventSK IN (SELECT ProductionEventSK FROM CTE WHERE rn > 1);

-- Check if duplicates still exist
SELECT ProductionEventSK,BottleID_Natural,Timestamp,COUNT(*)
FROM factproductionevent
GROUP BY ProductionEventSK,BottleID_Natural,Timestamp
HAVING COUNT(*) > 1;
```

 **PHASE 3**
 
**Handling NULL Values**
During the data cleaning process, several key columns contained missing values (NULL) such as:
 - JuiceTemperatureC_In
 - AmbientHumidityPercent_Line
 - ActualCapTorque_Nm
 - CapHopperLevel_Percent

### Solution Applied

 **1. Detection of NULLs**
 
Queried each column to identify how many rows contained missing values.

 **2. Calculation of Replacement Values**
 
Computed the column average (AVG()) using only non-NULL records.

 **3. Imputation**
 
Replaced all NULL values with their respective column averages, ensuring consistency in analysis.

 **4. Validation**
 
Rechecked the columns to confirm no remaining NULL values.

### Why This Approach?

- Using averages avoids dropping valuable records.

- Maintains dataset size for downstream analysis.

- Provides a balanced, non-biased replacement that reflects typical operating values.


## Hypothesis Exploration

 **Before we confirmed the root cause, we explore the following hypthesis:**

 - Date when the issues started: although the date has been given to us but we did that to confirm it and also to know the analysis days before now
 - Checking defect with the production line
 - Checking defect with shift time
 - Checking defect with the Captype and Capmaterial

## ANALYSIS



