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

```
select min(`date`), max(`date`)
from layoffs_staging2;
```

#### First, we want to look at companies that laid off employees, ordered from most to least.

<img width="94" alt="{3DABB79E-21DB-4ABE-AD45-2467B6CF1219}" src="https://github.com/user-attachments/assets/cce5fe98-dee6-4ce0-84b7-a56e92d5ad74">

```
select company, sum(total_laid_off) total
from layoffs_staging2
group by company
order by total desc;
```
#### Which company laid off all of its employees?
> Signify by percentage laid off of 100%, or as in the data shown as 1.

<img width="590" alt="{1B235845-CB3C-4EE3-BCAC-19CA41D18156}" src="https://github.com/user-attachments/assets/bc9e32db-1657-4a97-939f-ea183948b0eb">

```
select * from layoffs_staging2
where percentage_laid_off = 1
order by total_laid_off desc
limit 10;
```

#### Which industry was impacted the most?

<img width="196" alt="{2568885E-1490-4EC3-8680-E7B4DB52444B}" src="https://github.com/user-attachments/assets/3fe020ac-fd3f-4524-bef4-64beb8d6bd52">

> From the table, the industry impacted the most is consumer with 45182 employees laid off.
```
select industry, sum(total_laid_off), count(distinct company) total_companies
from layoffs_staging2
group by industry
order by sum(total_laid_off) desc;
```

#### What are the total employees laid off through the year?

<img width="140" alt="{CABA25DE-D94C-4A9C-B2EC-D7E5A1B470B1}" src="https://github.com/user-attachments/assets/720fba38-b31b-422b-96fc-5c5f91e5b863">

```
select year(`date`), sum(total_laid_off)
from layoffs_staging2
group by year(`date`)
order by year(`date`) desc;
```

#### What is the trend of employees laid off each month?

<img width="145" alt="{EB53CD8E-FF2B-49AE-9AFC-7F101137D1E7}" src="https://github.com/user-attachments/assets/7861cd3e-4dcf-444f-a83b-4fbbec7bff8c">

```
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

#### What are the top 5 companies laid off each year?

<img width="190" alt="{FE0D2CA3-6770-4409-86A0-8FC4FA9460E7}" src="https://github.com/user-attachments/assets/f8559637-6715-4d01-a175-ac98c9296b5b">

```
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
