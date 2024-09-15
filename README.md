
-- Exploratory Data Analysis 
-- in exploratory data analysis we sometimes explore data and sometimes even clean it at the same time but here we are only going to explore data and try bring positive insight from it 

select *
from layoffs_staging2; 

-- total_laid_off means no of employess laid off the job at one time 
-- percentage laid off gives us their percentage compared to total employees in the company so 1 means 100% employees where laid off 
 
select max(total_laid_off),max(percentage_laid_off)
from layoffs_staging2;

select *
from layoffs_staging2
where percentage_laid_off = 1
order by total_laid_off desc ; 

-- this gives us the sum of people laid off in total by a company it could be at one time or could be in multiple times
select company,sum(total_laid_off)
from layoffs_staging2
group by company
order by 2 desc;    -- 2 means 2nd column

-- we look for min and max date means the oldest date and the latest date between which these laid offs were made
select min(`date`),max(`date`)
from layoffs_staging2; 

-- now we look for the industry which laid off the max people and no of people laid off by each industry  ( consumer industry laid off the max)
select industry,sum(total_laid_off)
from layoffs_staging2
group by industry
order by 2 desc;    

-- now we look for the country which laid off the max people and no of people laid off by each country in the data set ( US laid off the max) 
select country,sum(total_laid_off)
from layoffs_staging2
group by country
order by 2 desc;   

-- this gives us the no of people being laid off per year in total (in 2023 most people were laid off)
select year(`date`),sum(total_laid_off)    -- year function gives us the date year 
from layoffs_staging2
group by year(`date`)
order by 1 desc;

-- stage tells us the business position of the company like post-ipo are companies like google , amazon etc ( Post ipo companies made the most laid offs)
select stage,sum(total_laid_off)
from layoffs_staging2
group by stage
order by 2 desc; 

-- this gives us the timeline of no of people laid off per month from starting date tot he ending  ending date 
select substring(`date`,1,7) as	 `Year_month`,sum(total_laid_off)
from layoffs_staging2
where substring(`date`,1,7) is not null
group by substring(`date`,1,7)
order by `Year_month`;

-- made a CTE to have a record of the rolling total along with the no of people being laid off every month 
-- for rolling total we have again applied sum on the query sum which was the sum of monthly laid off people 
with Rolling_total as 
(
select substring(`date`,1,7) as	 `Year_month`,sum(total_laid_off) as monthly_laid_off
from layoffs_staging2
where substring(`date`,1,7) is not null
group by substring(`date`,1,7)
order by `Year_month`
)
select `Year_month`, monthly_laid_off, sum( monthly_laid_off) over(order by `Year_month`) as rolling_total
from Rolling_total ;

-- now we look at the no of laid offs made by each company per year 

select company
from layoffs_staging2;

-- this first combines all the laid offs made in 2020 and then out of those laid offs it arranges them in desc order with company having highest laid at the top
  
select company,year(`date`),sum(total_laid_off)
from layoffs_staging2
where year(`date`) is not null
group by company,year(`date`)
order by 2, 3 desc;

-- this gives the max laid offs made in a single year by a company 

with Company_Year(Company,years,total_laid_off) as
(
select company,year(`date`),sum(total_laid_off)
from layoffs_staging2
group by company,year(`date`)
order by  3 desc
) 
,Company_Year_rank as
(
select *,dense_rank() over(partition by years order by total_laid_off desc) as Ranking 
-- this ranks the company making the highest laid off as no 1 and patition by the year so all the laid offs of 2020 appear at the top 
from Company_Year
where years is not null
# order by ranking         -- it is giving us top companies in terms of laid off per year , first we have all the companies ranked 1 then companies ranked 3 every year and so on 

)
-- here we have combined two CTES and in 2nd cte we hit the first cte
select *
from Company_Year_rank
where ranking <=5           -- it is giving us the top 5 companies by laid off per year 
;










