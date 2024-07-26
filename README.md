# Projects---SQL-Data-Cleaning
-- Data Cleaning

USE world_layoffs1;
SELECT *
FROM layoffs;


-- 1.Remove Duplicates
-- 2. Standardize the data 
-- 3. Null values or Blank Values
-- 4. Remove any irrelevant columns


-- copying raw data into another table
CREATE TABLE layoffs_staging
Like layoffs;

SELECT *
FROM layoffs_staging;

INSERT layoffs_staging
SELECT *
FROM layoffs;

SELECT *
FROM layoffs_staging;

-- using partition by to find the duplicates
SELECT *,
ROW_NUMBER() OVER(
PARTITION BY company, industry, total_laid_off, percentage_laid_off,`date`) as ROW_NUM
FROM layoffs_staging;

-- REMOVING DUPLICATES - PARTITION BY WITHIN A CTE 
WITH duplicate_CTE as
(
SELECT *,
ROW_NUMBER() OVER (
PARTITION BY company, industry, total_laid_off, percentage_laid_off,`date`, stage, country, funds_raised_millions) as ROW_NUM
FROM layoffs_staging
)
SELECT *
FROM duplicate_CTE
WHERE row_num>1;

-- checking the duplicates and changing the partition by appropriately
select *
from layoffs_staging
WHERE company = 'Casper';

-- deleting the duplicates
-- CREATE ANOTHER TABLE 'Layoffs_staging2'
-- Right lick on the Layoffs_staging - copy to clipboard - create a table
-- paste the code and change the name to 'layoffs_staging2' 
-- add another column row_nubers INT
CREATE TABLE `layoffs_staging2` (
  `company` text,
  `location` text,
  `industry` text,
  `total_laid_off` int DEFAULT NULL,
  `percentage_laid_off` text,
  `date` text,
  `stage` text,
  `country` text,
  `funds_raised_millions` int DEFAULT NULL,
  `row_num` INT
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4 COLLATE=utf8mb4_0900_ai_ci;

SELECT*
FROM layoffs_staging2;

-- We have an empty table and need to insert the information of partition by here i.e., all the information along with row_numbers
INSERT INTO layoffs_staging2
SELECT *,
ROW_NUMBER() OVER (
PARTITION BY company, industry, total_laid_off, percentage_laid_off,`date`, stage, country, funds_raised_millions) as ROW_NUM
FROM layoffs_staging;
-- Identifying duplicates as we couldn't delete it within the CTE 
SELECT*
FROM layoffs_staging2
WHERE row_num >1;
-- Deleting
DELETE
FROM layoffs_staging2
WHERE row_num >1;

SELECT *
FROM layoffs_staging2;

-- Standardizing data - finding issues within the data and fixing it
-- Removing spaces at the beginning and end
SELECT Company, TRIM(Company)
FROM layoffs_staging2; 

-- Updating
UPDATE layoffs_staging2
SET company = TRIM(COMPANY);

-- identifying unique industries and arranging it alphabetically
SELECT DISTINCT Industry
FROM layoffs_staging2
ORDER BY 1;
-- changing names of the industry , crypto and crypto currency are the same - updating 
SELECT *
FROM layoffs_staging2
WHERE INDUSTRY LIKE 'crypto%';



UPDATE layoffs_staging2
SET industry = 'Crypto'
WHERE industry like 'crypto%';
-- checking all the columns
SELECT DISTINCT location
FROM layoffs_staging2
ORDER BY 1;

SELECT DISTINCT country
FROM layoffs_staging2
ORDER BY 1;
-- here the united states is repeated twice with a '.' at the end - updating
UPDATE layoffs_staging2
SET country = 'United States'
WHERE country like 'United States.';

-- or set country = can use TRIM(Trailing,'.',from country)

-- date is in text format
-- to change it to date format
SELECT `date`
FROM layoffs_staging2;

-- updating the date into date format
UPDATE layoffs_staging2
SET `date`=str_to_date(`date`,'%m/%d/%Y');

-- Though the date looks like it is in the date format, it shows as text
-- converting

ALTER TABLE layoffs_staging2
MODIFY COLUMN `date` DATE;

-- Identifying NULL/BLANK values
SELECT *
FROM layoffs_staging2
WHERE total_laid_off IS NULL
AND percentage_laid_off IS NULL;

SELECT *
FROM layoffs_staging2
WHERE industry IS NULL 
OR industry = '';
-- replacing values of industry for the companies for which industry names are missing
SELECT t1.industry, t2.industry
FROM layoffs_staging2 t1
join layoffs_staging2 t2
 on t1.company=t2.company
 and t1.location=t2.location
WHERE (t1.industry IS NULL OR t1.industry = '')
AND t2.industry IS NOT NULL;


-- updating the industry names

UPDATE layoffs_staging2
SET industry = null
WHERE industry = '';

UPDATE layoffs_staging2 t1
join layoffs_staging2 t2
 on t1.company=t2.company
SET t1.industry = t2.industry
WHERE t1.industry IS NULL 
AND t2.industry IS NOT NULL;

SELECT*
from layoffs_staging2
WHERE company LIKE 'Bally%';

SELECT *
FROM layoffs_staging2;

ALTER TABLE layoffs_staging2
DROP COLUMN row_num;


