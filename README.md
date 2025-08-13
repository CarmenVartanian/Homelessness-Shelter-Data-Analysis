# Homelessness-Shelter-Data-Analysis
Homelessness &amp; Shelter Data from 2023 to 2025
Data form https://www.kaggle.com/datasets/shamimhasan8/homelessness-and-shelter-data
-- importing table
drop table if exists homelessness_shelter_data;
Create table homelessness_shelter_data
(
		id_num INT primary key,
		dates varchar(10),
		shelter_name varchar (20),
		city varchar(20),
		states varchar(5),
		total_capacity INT,
		occupied_beds INT,
		available_beds INT,
		occupancy_rate REAL,
		average_age INT,
		male_percentage INT,
		female_percentage INT,
		season varchar(10),
		notes varchar (100)
);
select * from homelessness_shelter_data;
-- cleaning data
-- 1. Remove Duplicates
-- 2. Standardize the Data
-- 3. Null Values or Blank Values



-- 1. Remove Duplicates

with duplicates AS (
	select *, 
		Row_number() over (partition by dates, shelter_name, city, states, total_capacity, 
						  occupied_beds, occupancy_rate, average_age, male_percentage, female_percentage, 
						   season, notes order by id_num ) as rn
	from homelessness_shelter_data
)
Select * from duplicates where rn > 1;

-- There isn't any duplicates.
-- 2. Standardize the data
Update homelessness_shelter_data
SET shelter_name = trim(shelter_name);
Update homelessness_shelter_data
Set city = trim(city);
Update homelessness_shelter_data
Set states = trim (states);
Update homelessness_shelter_data
set season = trim (season);
Update homelessness_shelter_data
set notes = trim (notes);

select * from homelessness_shelter_data
order by id_num;

select distinct (city)
from homelessness_shelter_data;

select distinct (shelter_name)
from homelessness_shelter_data;
select dates from homelessness_shelter_data;

-- Changing dates coulumn format 
update homelessness_shelter_data
Set dates = to_date(dates, 'MM-DD-YY');

select column_name, data_type from information_schema.columns
where table_name ='homelessness_shelter_data';

-- Changing dates column type to date

Alter Table homelessness_shelter_data
Alter column dates TYPE DATE
Using dates :: DATE;

--3. Null Values or Blank Values

Select * from homelessness_shelter_data
where shelter_name is Null or  shelter_name = ' ';

Select * from homelessness_shelter_data
where city is null or city = ' ';

Select * from homelessness_shelter_data
where average_age is null ;

-- There isn't any null data exept in notes column, which is okay to saty as is.
-- Exploratory Analysis

Select shelter_name, occupancy_rate, states, city
from homelessness_shelter_data
group by shelter_name, occupancy_rate, states, city
Having occupancy_rate = (select MAX(occupancy_rate) from homelessness_shelter_data);

select * from homelessness_shelter_data;

select MAX(average_age)
from homelessness_shelter_data;

select MIN(average_age)
from homelessness_shelter_data;

select states, SUM(occupied_beds)
from homelessness_shelter_data
group by states
order by 2 DESC
-- CA has the highest occupied beds

select city, SUM(occupied_beds)
from homelessness_shelter_data
group by city, states
Having states ILIKE 'CA'
order by 2 DESC
-- Los Angeles in CA has the highest occupied beds.

select shelter_name, SUM(occupied_beds)
from homelessness_shelter_data
group by shelter_name
order by 2 DESC
-- New Beginnings has the highest occupied beds

select MIN(dates), MAX(dates)
from homelessness_shelter_data;
-- min date is 2023-07-30 and max date is 2025-07-29

select date_part('year', dates) as years, sum(occupied_beds)
from homelessness_shelter_data
group by years
order by years DESC;

-- 2024 has the highest occupied beds. 

Select substring(TO_CHAR(dates, 'YYYY-MM-DD') from 6 For 2) AS months, sum(occupied_beds)
from homelessness_shelter_data
Group by months
Order by 1
;

With Rolling_Total AS 
(
Select substring(TO_CHAR(dates, 'YYYY-MM-DD') from 1 For 7) AS months, sum(occupied_beds) as total_occupied
from homelessness_shelter_data
Group by months
Order by 1 ASC
)
Select months, total_occupied,
SUM(total_occupied) OVER(order by months) as rolling_total
From Rolling_Total;

-- Rolling total of occupited beds

select shelter_name, Extract(YEAR FROM dates) AS years, sum(occupied_beds)
From homelessness_shelter_data
Group by shelter_name, Years
Order by 3 DESC;

with shelter_year(shelter_name, years, occupied_beds) AS
(
	select shelter_name, Extract(YEAR FROM dates) as years, sum(occupied_beds)
From homelessness_shelter_data
Group by shelter_name, years
Order by 3 DESC
)
Select *, 
dense_Rank() over(partition by years order by occupied_beds DESC) as Ranking
From shelter_year
Order by Ranking ASC;

-- in 2023, Shelter Plus has the most occupied beds.
-- in 2024, Harbor Home has the most occupied beds.
-- in 2025, New Beginnings has the most occupied beds.
