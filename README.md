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


