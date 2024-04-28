# Companies Layoff Analysis

## Table of Contents
- [Project  Overview](#project-overview)
- [Data Source](#data-source)
- [Tools](#tools)
- [Data Cleaning/Preparation](#data-cleaningpreparation)
- [Exploratory Data Analysis](#exploratory-data-analysis)
- [Data Analysis](#data-analysis)
- [Findings](#findings)
- [Recommendations](#recommendations)
- [Limitations](#limitations)

### Project Overview

This data analysis project aims to delve into workforce reduction patterns spanning the years 2020 to 2023 across various industries and countries.We will be analyzing the top 20 companies with the highest layoff, industries and the country. By scrutinizing data from this period,
 we endeavor to grasp the nuances and fluctuations in layoffs, shedding light on the global economic ramifications during this period.

 ### Data Source
 
 Layoff Data: The primary dataset utilized for this analysis is the "layoffs.csv" file, encompassing various details about companies, industries, years, countries, and total layoffs.

 ### Tools

 - SQL Server- Data Cleaning[Download Here]
- SQL Server- Exploratory Data Analysis
- Tableau- Report Generation and Visualization [View here](https://public.tableau.com/app/profile/abimbola.ajayi8433/vizzes)

  
### Data Cleaning/Preparation

During the initial data preparation phase, the following tasks were executed:
1. Data importation into MySQL Workbench
2. Removal of duplicates
3. Standardization of data to identify and rectify issues
4. Conversion of date format from string to date
5. Handling of missing values
6. Dropping of unused columns and rows
7. Data cleaning and formatting

```sql

-- Removing Duplicates- the data set doesn;t have a unique value to identify the duplicate values

With duplicate_cte AS 
(
SELECT *,
ROW_NUMBER()OVER(PARTITION BY company,location,industry,total_laid_off,percentage_laid_off,`date`,stage,country) AS row_num
FROM layoffs
)
SELECT *
FROM duplicate_cte
WHERE row_num > 1;
```
```sql

-- Creating a new table in order to delete the duplicates and adding row_num created into the new table
CREATE TABLE `layoff2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` int
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

INSERT INTO layoff2
SELECT *,
ROW_NUMBER()OVER(PARTITION BY company,location,industry,total_laid_off,percentage_laid_off,`date`,stage,country,funds_raised_millions) AS row_num
FROM layoffs;

SELECT *
FROM layoff2
WHERE row_num > 1;

-- Removing the duplicates

DELETE FROM layoff2
WHERE row_num > 1;
```

```sql

-- Standardize the data- finding issues in the data and fixing it 
-- Fixing the extra space in the company's column

SELECT company,TRIM(company)
from layoff2;

SELECT DISTINCT(company)
from layoff2;
UPDATE layoff2
SET company = TRIM(company);
```
```sql

-- Standardizing the industry column

SELECT industry
FROM layoff2
WHERE industry LIKE 'Crypto%';

UPDATE layoff2
SET industry = 'Crypto'
WHERE industry LIKE 'Crypto%';
```
```sql
-- Changing the date format from string to date 

SELECT `date`,str_to_date(`date`,'%m/%d/%Y')
FROM layoff2;

UPDATE layoff2
SET `date`= str_to_date(`date`,'%m/%d/%Y');

SELECT DISTINCT(country), TRIM(TRAILING'.' FROM country)
FROM layoff2
ORDER BY 1;

UPDATE layoff2
SET country = TRIM(TRAILING'.' FROM country);
```
```sql

-- NUll values & Blank values: Populating the blank values

SELECT *
FROM layoffs
WHERE industry IS NULL 
OR industry = '';

SELECT company, industry
FROM layoffs
WHERE company = 'Airbnb';

SELECT L2.industry,L2.company
FROM layoff2 L1
JOIN layoff2 L2 ON 
L1.company = L2.company
WHERE (L1.industry IS NULL OR L1.industry ='')
AND L2.industry IS NOT NULL;

UPDATE layoff2 L1
JOIN layoff2 L2
ON L1.company = L2.company
SET L1.industry = L2.industry
WHERE L1.industry = ''
AND L2.industry <> '';
```

### Exploratory Data Analysis 

EDA involve exploring the data to answer the key questions, such as:

- What are the top 20 companies experiencing the highest number of layoffs?
- What industries that experienced the highest and lowest rates of layoffs?
- Which countries has the highest rates of layoffs?
- What was the year with the highest number of layoffs recorded?
- Which companies had the highest number of layoffs in each year?

### Data Analysis

```Sql

 SELECT company, SUM(total_laid_off) AS Sum_total
 FROM layoff2
 GROUP BY company
 ORDER BY sum_total DESC
 LIMIT 20;
```
```sql
SELECT industry, SUM(total_laid_off) AS Sum_total
 FROM layoff2
 GROUP BY industry
 ORDER BY sum_total DESC;
```
```sql
SELECT YEAR(`date`), SUM(total_laid_off) AS Sum_total
 FROM layoff2
 GROUP BY YEAR(`date`)
 ORDER BY 1 DESC;
```
```sql
WITH company_Year (Company, years,total_laid_off) AS
 (
 SELECT Company, YEAR(`date`), SUM(total_laid_off) AS Sum_total
 FROM layoff2
 GROUP BY company, YEAR(`date`)
 )
 SELECT *, DENSE_RANK() OVER(PARTITION BY years ORDER BY total_laid_off DESC) AS `Rank`
 FROM Company_Year
 WHERE years IS NOT NULL 
 AND total_laid_off IS NOT NULL;
```
```sql
With Company_Year (company, year, total_laid_off) AS 
 (
 SELECT company, YEAR(`date`),SUM(total_laid_off)
 FROM layoff2
 GROUP BY company,YEAR(`date`)
 ORDER BY SUM(total_laid_off) DESC
 )
 SELECT *
 FROM company_year;
```

### Findings

- The analysis revealed a notable surge in workforce reduction events starting from the onset of the COVID-19 pandemic in 2020, with the trend persisting through subsequent years.
Year-over-year layoff increases as the year progresses from 2020-2023, comparisons highlighted fluctuations in layoff rates, suggesting varying degrees of economic recovery and instability across different 
periods.
- Certain industries exhibited a disproportionately higher frequency and magnitude of layoffs compared to others. Industries such as consumer, hospitality, travel,retail and real estate were particularly hard-hit by the pandemic-induced economic downturn, experiencing significant workforce reductions.
- Geographic analysis unveiled regional disparities in layoff trends, reflecting the uneven impact of the pandemic on different countries and regions. Countries like united states, India, Netherland witnessed higher layoff rate.

### Recommendations

Based on the analysis, we recommend the following actions

The analysis identified specific companies that stood out for their substantial layoffs during the study period. By examining the characteristics and circumstances surrounding these companies, we gained insights into the factors driving workforce reductions, such as financial instability, market challenges, and restructuring efforts.
Diversification of Revenue Streams:

1. Investment in Digital Transformation
2. Employee Reskilling and Upskilling
3. Flexible Work Arrangements
4. Strategic Talent Retention Strategies
5. Financial Planning and Risk Management
6. Foster collaboration and strategic partnerships with other businesses, industry associations, and government entities to leverage collective resources, share best practices, and explore opportunities for mutual growth and resilience.

### Limitations
In order to ensure the accuracy and consistency of my analysis, I needed to exclude certain data points where the total number of layoffs and the corresponding percentage were null. Including this incomplete data could have introduced inconsistencies into the analysis, potentially skewing the results.





