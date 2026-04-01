# Driftwood-BCPS
# Cyberlocke Sample School Funding Dashboard

### Overview
This is a **sample Power BI dashboard** created for a client (BCPS) to showcase a quarterly view of school funding, expenditures, and student performance. The dashboard uses **synthetic data** representing county schools to demonstrate key reporting metrics and insights.

---

### Purpose
The dashboard is designed to help clients quickly understand:

- Funding vs. expenditure for each program and school  
- Funding spend delta and coverage ratio  
- Areas where funding may be missing or underutilized  
- Truancy trends and students lagging behind  
- Lost dollars by program or category  

It provides both high-level and detailed views of school funding and student performance to support data-driven decisions.

---

### Dashboard Structure

#### 1. Overview Page
Provides a **summary of funding and expenditure** across programs and schools.

Key features:
- Total funding and expenditure  
- Funding vs. spend delta  
- Coverage ratios by program or category  
- Lost dollars demo (highlighting funding gaps)  
- Truancy statistics  

#### 2. Details Page
Provides a **granular breakdown** to help identify specific issues or trends.

Key features:
- Funding and expenditure by program and school  
- Number of students lagging behind  
- Truancy trends  
- Program-wise coverage and gaps  

---

### Tools Used
- Power BI  
- Excel (for sample data generation)  
- DAX for calculated measures  

---

### Key Highlights
- Demonstrates a **quarterly drop view** of school funding and performance  
- Includes measures for coverage ratios, funding delta, and student eligibility  
- Designed for **clarity and stakeholder readability**  

---

### Notes
- Data is **synthetic** for demonstration purposes  
- Measures such as `[Delta Funding]`, `[Coverage Ratio]`, and `[Eligible Students]` showcase typical calculations used in real-world school funding dashboards  
- Dashboard is **interactive** and supports filtering by program, school, and category  

---

### DAX to SQL
**In DAX:** Average Quarter Grade per Student = 
AVERAGEX(
    VALUES(Fact_FundingExpenditure[StudentID]),
    CALCULATE(AVERAGE(Fact_FundingExpenditure[QuarterGrade]))
)
---
**Convert to SQL:** 
WITH StudentAvg AS (
    SELECT
        StudentID,
        AVG(QuarterGrade) AS AvgQuarterGrade
    FROM Fact_FundingExpenditure
    GROUP BY StudentID
)
SELECT
    AVG(AvgQuarterGrade) AS AvgQuarterGradePerStudent
FROM StudentAvg;
---
**In DAX:** 
Avg Attendance % (Last 30d Returnees) = 
VAR ReturnCutoff =
    TODAY() - 30
RETURN
CALCULATE (
    AVERAGEX (
        VALUES (Fact_RefundsVouchers[StudentID] ),
        CALCULATE (
            DIVIDE (
                SUM ( fact_attendance[IsPresent] ),
                COUNTROWS ( fact_attendance ),
                0
            )
        )
    ),
    Fact_RefundsVouchers[ReturnDate] >= ReturnCutoff
)
---
**Convert to SQL:**
WITH Returnees AS (
    -- Students who returned in the last 30 days
    SELECT DISTINCT StudentID
    FROM Fact_RefundsVouchers
    WHERE ReturnDate >= CURRENT_DATE - INTERVAL '30' DAY
),
StudentAttendance AS (
-- Calculate attendance % per student
    SELECT
        a.StudentID,
        CASE 
            WHEN COUNT(*) = 0 THEN 0
            ELSE SUM(a.IsPresent) * 1.0 / COUNT(*) 
        END AS AttendancePct
    FROM Fact_Attendance a
    INNER JOIN Returnees r
        ON a.StudentID = r.StudentID
    GROUP BY a.StudentID
)
-- Average attendance % across all returnees
SELECT AVG(AttendancePct) AS AvgAttendancePctLast30Days
FROM StudentAttendance;
