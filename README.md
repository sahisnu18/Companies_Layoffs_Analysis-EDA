# Companies Layoffs Analysis - Exploratory Data Analysis (EDA)

## Table of Contents

- [Project Overview](#project-overview)
- [Data Source](#data-source)
- [Exploratory Data Analysis](#exploratory-data-analysis)

### Project Overview

This data analysis aims to identify trends on companies layoffs around the world during covid pandemic from March 2020 to March 2023.

### Data Source

The dataset used for this project is 'layoffs.csv', you can download it [here.](layoffs.csv)

### Exploratory Data Analysis
EDA involved exploring the layoffs data to answer questions, such as:

- Which companies have laid off employees? Order from most to least.
- Which company laid off all of its employees?
- Which industry was impacted the most?
- What are the total employees laid off through the year?
- What is the trend of employees laid off each month?
- What are the top 5 companies laid off each year?

The layoffs data gathered from 2020-03-11 to 2023-03-06

<img width="113" alt="{64466A57-8B1B-4008-92EA-FF48474F0C62}" src="https://github.com/user-attachments/assets/743c517e-df56-4266-95c4-94b987c9ad23">

```sql
select min(`date`), max(`date`)
from layoffs_staging2;
```

#### First, we want to look at companies that laid off employees, ordered from most to least.

<img width="94" alt="{3DABB79E-21DB-4ABE-AD45-2467B6CF1219}" src="https://github.com/user-attachments/assets/cce5fe98-dee6-4ce0-84b7-a56e92d5ad74">

```sql
select company, sum(total_laid_off) total
from layoffs_staging2
group by company
order by total desc;
```
#### Which company laid off all of its employees?
> Signify by percentage laid off of 100%, or as in the data shown as 1.

<img width="590" alt="{1B235845-CB3C-4EE3-BCAC-19CA41D18156}" src="https://github.com/user-attachments/assets/bc9e32db-1657-4a97-939f-ea183948b0eb">

```sql
select * from layoffs_staging2
where percentage_laid_off = 1
order by total_laid_off desc
limit 10;
```

#### Which industry was impacted the most?

<img width="196" alt="{2568885E-1490-4EC3-8680-E7B4DB52444B}" src="https://github.com/user-attachments/assets/3fe020ac-fd3f-4524-bef4-64beb8d6bd52">

> From the table, the industry impacted the most is consumer with 45182 employees laid off.
```sql
select industry, sum(total_laid_off), count(distinct company) total_companies
from layoffs_staging2
group by industry
order by sum(total_laid_off) desc;
```

#### What are the total employees laid off through the year?

<img width="140" alt="{CABA25DE-D94C-4A9C-B2EC-D7E5A1B470B1}" src="https://github.com/user-attachments/assets/720fba38-b31b-422b-96fc-5c5f91e5b863">

```sql
select year(`date`), sum(total_laid_off)
from layoffs_staging2
group by year(`date`)
order by year(`date`) desc;
```

#### What is the trend of employees laid off each month?

<img width="145" alt="{EB53CD8E-FF2B-49AE-9AFC-7F101137D1E7}" src="https://github.com/user-attachments/assets/7861cd3e-4dcf-444f-a83b-4fbbec7bff8c">

```sql
-- cte rolling total of employee laid off each month
with Rolling_total as
(
select substring(`date`, 1,7) as `month`, sum(total_laid_off) total_emp
from layoffs_staging2
where substring(`date`, 1,7) is not null
group by `month`
order by month asc
)
select `month`, total_emp, sum(total_emp) over(order by `month`) as rolling_total
from Rolling_total;
```
This query calculates a rolling total of employees laid off each month using a Common Table Expression (CTE) and a window function. We use CTE as it can be thought of as a way to prepare or pre-filter data for further processing in subsequent queries.

1. First Filtering (in the CTE):
- The CTE (Rolling_total) filters and aggregates the data from the original table (layoffs_staging2).
- Specifically, it:
  - Extracts the year and month (SUBSTRING(date, 1, 7)).
  - Excludes rows with NULL months (WHERE SUBSTRING(date, 1, 7) IS NOT NULL).
  - Groups the data by month and calculates total layoffs for each (SUM(total_laid_off)).

2. Second Filtering (in the main query):
- After defining the pre-processed data (Rolling_total), the main query applies additional calculations or filters.
- In this case, the main query:
  - Retrieves month and total_emp from the CTE.
  - Calculates the rolling total (SUM(total_emp) OVER (ORDER BY month)).

#### What are the top 5 companies laid off each year?

<img width="190" alt="{FE0D2CA3-6770-4409-86A0-8FC4FA9460E7}" src="https://github.com/user-attachments/assets/f8559637-6715-4d01-a175-ac98c9296b5b">

```sql
with Company_year (company, years, total_laid_off) as
(
select company, year(`date`), sum(total_laid_off)
from layoffs_staging2
group by company, year(`date`)
), company_year_rank as
(
select *, dense_rank() over(partition by years order by total_laid_off desc) as ranking
from Company_year
where years is not null
)
select * from company_year_rank
where ranking <= 5;
```
How It Works Overall
1. The first CTE (Company_year) aggregates the total layoffs for each company in each year.
2. The second CTE (company_year_rank) calculates the rank of each company by their layoffs, grouped by year.
3. The final query filters the data to show only the top 5 companies with the most layoffs for each year.

Query Explanation
1. First CTE: Company_year
```sql
WITH Company_year (company, years, total_laid_off) AS (
    SELECT 
        company, 
        YEAR(`date`) AS years, 
        SUM(total_laid_off) AS total_laid_off
    FROM layoffs_staging2
    GROUP BY company, YEAR(`date`)
)

```
Purpose: Summarizes layoffs by company and year.
Breakdown:
- YEAR('date') AS years: Extracts the year from the 'date' column.
- SUM(total_laid_off) AS total_laid_off: Sums the total layoffs for each company in each year.
- GROUP BY company, YEAR('date'): Groups the data by company and year to ensure the aggregation SUM is performed for each company-year combination.

2. Second CTE: company_year_rank
```sql
, company_year_rank AS (
    SELECT 
        *, 
        DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS ranking
    FROM Company_year
    WHERE years IS NOT NULL
)

```
Purpose: Ranks companies by layoffs within each year.
Breakdown:
- DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC):
  - Assigns a rank to each company based on their total layoffs within the same year (PARTITION BY years).
  - Companies with the same total_laid_off receive the same rank (dense ranking skips numbers, unlike RANK).
- WHERE years IS NOT NULL: Ensures that only rows with valid years are included.

3. Final Query:
```sql
  SELECT * 
  FROM company_year_rank
  WHERE ranking <= 5;
```
Purpose: Filters the results to show only the top 5 companies for each year (based on ranking).
Breakdown:
- WHERE ranking <= 5: Includes only rows where the rank is 5 or less, effectively limiting results to the top 5 companies with the most layoffs per year.
