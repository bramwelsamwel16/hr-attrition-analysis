# SQL Analysis — HR Attrition

Queries run against the star-schema HR Attrition database (`Fact_Attrition` joined to its dimension tables). Each query below answers one business question, with real output and the action it points to.

---

## 1. What is the company's overall attrition rate?

```sql
SELECT
    COUNT(*) AS total_employees,
    SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) AS attrition_count,
    ROUND(100.0 * SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition;
```

**Output**

| total_employees | attrition_count | attrition_rate_pct |
|---|---|---|
| 1470 | 237 | 16.1 |

**Key insight & recommendation:** 1 in every 6 employees leaves. That's a baseline to track over time — any retention initiative below should be measured against whether it moves this number down.

---

## 2. Which department has the highest attrition?

```sql
SELECT
    d.Department,
    COUNT(*) AS headcount,
    SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) AS left_count,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition f
JOIN Dim_Department d ON f.DepartmentID = d.DepartmentID
GROUP BY d.Department
ORDER BY attrition_rate_pct DESC;
```

**Output**

| Department | headcount | left_count | attrition_rate_pct |
|---|---|---|---|
| Sales | 446 | 92 | 20.6 |
| Human Resources | 63 | 12 | 19.0 |
| Research & Development | 961 | 133 | 13.8 |

**Key insight & recommendation:** Sales has both the highest rate *and* a large headcount, so it drives the biggest share of total turnover. Prioritize Sales for a retention review before HR, even though HR's rate is close — the dollar and headcount impact is larger in Sales.

---

## 3. Which job roles are most at risk?

```sql
SELECT
    r.JobRole,
    COUNT(*) AS headcount,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition f
JOIN Dim_JobRole r ON f.JobRoleID = r.JobRoleID
GROUP BY r.JobRole
ORDER BY attrition_rate_pct DESC;
```

**Output**

| JobRole | headcount | attrition_rate_pct |
|---|---|---|
| Sales Representative | 83 | 39.8 |
| Laboratory Technician | 259 | 23.9 |
| Human Resources | 52 | 23.1 |
| Sales Executive | 326 | 17.5 |
| Research Scientist | 292 | 16.1 |
| Manufacturing Director | 145 | 6.9 |
| Healthcare Representative | 131 | 6.9 |
| Manager | 102 | 4.9 |
| Research Director | 80 | 2.5 |

**Key insight & recommendation:** Sales Representative attrition (39.8%) is nearly double the next-highest role. This is the single highest-leverage group for a targeted retention plan — losing 4 in 10 reps every year is expensive to keep re-hiring and retraining.

---

## 4. Does overtime actually predict attrition?

```sql
SELECT
    OverTime,
    COUNT(*) AS headcount,
    ROUND(100.0 * SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition
GROUP BY OverTime
ORDER BY attrition_rate_pct DESC;
```

**Output**

| OverTime | headcount | attrition_rate_pct |
|---|---|---|
| Yes | 416 | 30.5 |
| No | 1054 | 10.4 |

**Key insight & recommendation:** Overtime nearly triples attrition risk. Audit which teams generate the most overtime and whether it's from understaffing — fixing headcount there is likely cheaper than the cost of repeated turnover.

---

## 5. How does attrition change by job level?

```sql
SELECT
    l.JobLevelLabel,
    COUNT(*) AS headcount,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition f
JOIN Dim_JobLevel l ON f.JobLevelID = l.JobLevelID
GROUP BY l.JobLevelLabel
ORDER BY attrition_rate_pct DESC;
```

**Output**

| JobLevelLabel | headcount | attrition_rate_pct |
|---|---|---|
| Entry Level | 543 | 26.3 |
| Mid-Level | 218 | 14.7 |
| Junior | 534 | 9.7 |
| Executive | 69 | 7.2 |
| Senior | 106 | 4.7 |

**Key insight & recommendation:** Attrition falls steadily as job level rises. Entry Level is the leak point — invest in onboarding, mentorship, and a visible promotion path in the first 1–2 years, where the data shows people are most likely to leave.

---

## 6. Does business travel frequency affect attrition?

```sql
SELECT
    t.BusinessTravel,
    COUNT(*) AS headcount,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition f
JOIN Dim_BusinessTravel t ON f.BusinessTravelID = t.BusinessTravelID
GROUP BY t.BusinessTravel
ORDER BY attrition_rate_pct DESC;
```

**Output**

| BusinessTravel | headcount | attrition_rate_pct |
|---|---|---|
| Travel_Frequently | 277 | 24.9 |
| Travel_Rarely | 1043 | 15.0 |
| Non-Travel | 150 | 8.0 |

**Key insight & recommendation:** Frequent travel roughly triples attrition risk compared to no travel. Consider travel rotation, caps, or extra compensation for roles requiring frequent travel.

---

## 7. Do employees who leave earn less and stay for less time?

```sql
SELECT
    Attrition,
    ROUND(AVG(MonthlyIncome), 0) AS avg_monthly_income,
    ROUND(AVG(YearsAtCompany), 1) AS avg_years_at_company,
    ROUND(AVG(TotalWorkingYears), 1) AS avg_total_working_years
FROM Fact_Attrition
GROUP BY Attrition;
```

**Output**

| Attrition | avg_monthly_income | avg_years_at_company | avg_total_working_years |
|---|---|---|---|
| No | 6833.0 | 7.4 | 11.9 |
| Yes | 4787.0 | 5.1 | 8.2 |

**Key insight & recommendation:** Leavers earn ~30% less and have ~2.3 fewer years of tenure and ~3.7 fewer years of total experience. This points to pay compression for newer/junior staff — review whether early-career pay is competitive enough to retain people past year 2–3.

---

## 8. Is work-life balance linked to attrition?

```sql
SELECT
    w.WorkLifeBalanceLabel,
    COUNT(*) AS headcount,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition f
JOIN Dim_WorkLifeBalance w ON f.WorkLifeBalanceID = w.WorkLifeBalanceID
GROUP BY w.WorkLifeBalanceLabel
ORDER BY attrition_rate_pct DESC;
```

**Output**

| WorkLifeBalanceLabel | headcount | attrition_rate_pct |
|---|---|---|
| Bad | 80 | 31.3 |
| Best | 153 | 17.6 |
| Good | 344 | 16.9 |
| Better | 893 | 14.2 |

**Key insight & recommendation:** "Bad" work-life balance nearly doubles attrition risk vs. every other category. Treat a "Bad" self-rating as an early-warning flag for managers to proactively check in, before it turns into a resignation.

---

## 9. Do stock options improve retention?

```sql
SELECT
    StockOptionLevel,
    COUNT(*) AS headcount,
    ROUND(100.0 * SUM(CASE WHEN Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition
GROUP BY StockOptionLevel
ORDER BY StockOptionLevel;
```

**Output**

| StockOptionLevel | headcount | attrition_rate_pct |
|---|---|---|
| 0 | 631 | 24.4 |
| 1 | 596 | 9.4 |
| 2 | 158 | 7.6 |
| 3 | 85 | 17.6 |

**Key insight & recommendation:** Employees with zero stock options leave at 24.4%, more than double levels 1 and 2. Extending even a small equity grant earlier (not just at senior levels) looks like a cost-effective retention lever. Level 3 ticking back up (17.6%) is worth a closer look — that group may include employees leaving for reasons equity doesn't offset (e.g. role or manager issues).

---

## 10. Where is the single highest-risk combination?

```sql
SELECT
    r.JobRole,
    f.OverTime,
    COUNT(*) AS headcount,
    ROUND(100.0 * SUM(CASE WHEN f.Attrition = 'Yes' THEN 1 ELSE 0 END) / COUNT(*), 1) AS attrition_rate_pct
FROM Fact_Attrition f
JOIN Dim_JobRole r ON f.JobRoleID = r.JobRoleID
WHERE r.JobRole = 'Sales Representative'
GROUP BY r.JobRole, f.OverTime;
```

**Output**

| JobRole | OverTime | headcount | attrition_rate_pct |
|---|---|---|---|
| Sales Representative | No | 59 | 28.8 |
| Sales Representative | Yes | 24 | 66.7 |

**Key insight & recommendation:** Sales Representatives who work overtime leave at **66.7%** — two out of every three. This is the single most urgent segment in the entire dataset. Start the retention plan here: reduce forced overtime for this role, or add substantial overtime compensation, before addressing any other segment.
