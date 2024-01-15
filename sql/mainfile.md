# COVID-19 Data Analysis Project

## Overview

This project focuses on analyzing COVID-19 data using SQL queries and presenting the insights through data visualization. The dataset used contains information related to the spread and impact of COVID-19 globally.

## Table of Contents


- [SQL Queries](#sql-queries)
- [Data Visualization](#data-visualization)

## Features

- SQL queries for COVID-19 data cleaning and analysis
- Data visualization using Tableau for vaccination trends 
- Exploration of key metrics like confirmed cases, deaths, and vaccination
- Comparative analysis across countries and regions

## Prerequisites

Before you begin, ensure you have the following requirements:

- SQL database (for executing SQL queries)
- Tableau Desktop (for viewing Tableau visualizations)

## Usage

To utilize this project, follow these steps:

1. Run the SQL queries in the [sql](/sql) directory against your COVID-19 database to clean and analyze the data.

2. Explore the Tableau visualizations for vaccination trends located in the [tableau_visualizations](/tableau_visualizations) directory.


## SQL Queries

Explore COVID-19 data through the following SQL queries:

- [Query 1: Cleaning the raw data and preparing it for analysis.]
- [Query 2: Total confirmed cases, deaths, and vaccination globally.]
- [Query 3: Comparative analysis of vaccine in specific regions or countries.]
- [Query 4: Identify hotspots and trends over time.]

## SQL Data Cleaning Queries

### 1. Adjusting Column Data Types
- This queries modifies the data types of certain columns in the 'death1' table to accommodate larger numbers, ensuring data integrity and preventing potential overflow issues.
  
```sql
-- Change data type of 'new_cases' column to bigint
ALTER TABLE death1
ALTER COLUMN new_cases bigint;

-- Change data type of 'new_deaths' column to bigint
ALTER TABLE death1
ALTER COLUMN new_deaths bigint;

-- Change data type of 'total_deaths' column to bigint in the 'p1' schema
ALTER TABLE p1..death1
ALTER COLUMN total_deaths BIGINT;
```

### 2. Removing Zero Values
- This queries addresses data quality by updating rows where 'total_cases' or 'total_deaths' have a value of 0, setting them to NULL to avoid misleading or inaccurate information
```
-- Set 'total_cases' to NULL where its value is 0
UPDATE death1
SET total_cases = NULL
WHERE total_cases = 0;

-- Set 'total_deaths' to NULL where its value is 0
UPDATE death1
SET total_deaths = NULL
WHERE total_deaths = 0;
```
### 3. Finding affected and death rate by country

- This query retrieves data for India from the 'p1..death1' table, calculating the percentage of the population affected and the death rate for each entry.
  
- 'affected' column represents the percentage of the population affected by COVID-19, while the 'deathrate' column shows the percentage of deaths among the total cases.
  
- The results are ordered by the year of the date in ascending order.
```
/* Finding affected and death rate by country */
SELECT
    location,
    date,
    population,
    total_cases,
    (total_cases / population) * 100 AS affected,
    total_deaths,
    (total_deaths / total_cases) * 100 AS deathrate
FROM p1..death1
WHERE location = 'India'
ORDER BY YEAR(date) ASC;

```
### 4. Finding max value by country

- This query retrieves aggregated data for each country from the 'p1..death1' table, showing the maximum date, maximum total cases, highest percentage of affected population, and maximum total deaths.
- The results are filtered to exclude entries where the 'continent' is NULL and are ordered by the highest percentage of affected population in descending order.

```
/* Finding max value by country */
SELECT
    location,
    MAX(date) AS max_date,
    population,
    MAX(total_cases) AS maxcases,
    MAX((total_cases / population)) * 100 AS highestaffected,
    MAX(total_deaths) AS maxdeath
FROM p1..death1
WHERE continent IS NOT NULL
GROUP BY location, population
ORDER BY highestaffected DESC;

```
### 5. Selecting max affected by country within a date range

- This query uses parameters to define a date range and retrieves data for each country from the 'p1..death1' table within that range.
- It shows the maximum total cases and the highest percentage of affected population for each country within the specified date range.

```
DECLARE @start_date DATE = '2020-04-01';
DECLARE @end_date DATE = '2021-08-31';
SELECT
    location,
    date,
    population,
    MAX(total_cases) AS t1,
    MAX((total_cases / population)) * 100 AS highestaffected
FROM p1..death1
WHERE date BETWEEN @start_date AND @end_date
GROUP BY location, date, population
ORDER BY highestaffected DESC;

```
### 6.  Death by Continent

- This query aggregates data by continent from the 'p1..death1' table, showing the maximum total deaths for each continent.
- The results are ordered by the maximum total deaths in descending order.

```
-- Death by Continent
SELECT
    continent,
    MAX(total_deaths) AS DeathbyContinent
FROM p1..death1
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY DeathbyContinent DESC;

```
### 7.Calculating Total Death Rate by Location and Date

- This query calculates the total death rate for each location and date by summing up the total deaths and total cases.
- The result shows the percentage of deaths among the total cases for each location and date, ordered by location.

```
SELECT
    location,
    date,
    SUM(total_deaths) / SUM(total_cases) * 100 AS totaldeathrate
FROM p1..death1
WHERE continent IS NOT NULL
GROUP BY location, date
ORDER BY location;
```

### 8.Counting Total Number of Deaths and Calculating Death Percentage
- This script creates a temporary table to store the total number of cases and deaths.
- It then calculates the death percentage (DP) for the aggregated data and displays the results

```
CREATE TABLE #tempDeathStats
(
    total_cases NUMERIC,
    total_deaths NUMERIC
);

INSERT INTO #tempDeathStats
SELECT
    SUM(new_cases) AS total_cases,
    SUM(new_deaths) AS total_deaths
FROM
    p1..death1
WHERE
    continent IS NOT NULL;

-- Select data from the temporary table and calculate the death percentage (DP)
SELECT
    total_cases,
    total_deaths,
    CASE WHEN total_cases <> 0 THEN total_deaths / total_cases * 100 ELSE NULL END AS DP
FROM
    #tempDeathStats;

-- Drop the temporary table
DROP TABLE #tempDeathStats;
```

### 9.Joining Two Tables to Get Specific Data

- This query joins the 'p1..death1' and 'p1..vac1' tables based on the location and date columns to retrieve specific data from both tables.
```
SELECT *
FROM p1..death1
JOIN p1..vac1
    ON death1.location = vac1.location
    AND death1.date = vac1.date;


```

### 10.Finding Vaccination Rate at the End of Data

- This query calculates the vaccination rate at the end of the data by summing up new vaccinations over time for each location.
- The results include information about continent, location, date, population, new vaccinations, and the cumulative count of vaccinations.

```
SELECT
    death1.continent,
    death1.location,
    death1.date,
    death1.population,
    vac1.new_vaccinations,
    SUM(CAST(vac1.new_vaccinations AS FLOAT)) OVER (PARTITION BY death1.location ORDER BY death1.location, death1.date) AS newcountedvacc
FROM p1..death1
JOIN p1..vac1
    ON death1.location = vac1.location
    AND death1.date = vac1.date
WHERE death1.continent IS NOT NULL
ORDER BY location, date;
```
### 11.Creating a Temporary Table and Calculating Vaccination Percentage

- This script creates a temporary table '#vaccinatedpopulations' and inserts data into it by joining 'p1..death1' and 'p1..vac1'.
- It calculates the cumulative count of vaccinations and the vaccination percentage for each location, providing valuable insights into vaccination coverage.

```
-- Create Temp Table and Calculate Vaccination Percentage
CREATE TABLE #vaccinatedpopulations
(
    continent NVARCHAR(255),
    location NVARCHAR(255),
    Date DATETIME,
    population NUMERIC,
    new_vaccinations NUMERIC,
    newcountedvacc NUMERIC
);

-- Insert data into the temporary table
INSERT INTO #vaccinatedpopulations
SELECT
    death1.continent,
    death1.location,
    death1.date,
    death1.population,
    vac1.new_vaccinations,
    SUM(vac1.new_vaccinations) OVER (PARTITION BY death1.location ORDER BY death1.location, death1.date) AS newcountedvacc
FROM
    p1..death1
JOIN
    p1..vac1
    ON death1.location = vac1.location
    AND death1.date = vac1.date
WHERE
    death1.continent IS NOT NULL
ORDER BY
    death1.location, death1.date;

-- Select data and calculate the percentage
SELECT
    *,
    (newcountedvacc / population) * 100 AS vaccination_percentage
FROM
    #vaccinatedpopulations;

```


## Data Visualization
Tableau Dashboard:<https://public.tableau.com/app/profile/vraj.patel1693/viz/CovidVacination_17045511504310/Dashboard2>
<https://public.tableau.com/app/profile/vraj.patel1693/viz/DEATHDATA/Dashboard1>

Visualize the analyzed data with the following:

- [Visualization 1: Interactive map of  worldwide.]
- [Visualization 2: Time series plots showing the progression of vaccination.]
- [Visualization 3: Tableau visualizations for vaccination trends.]
  # COVID-19 Data Visualization

## Part 1: Vaccination Data Visualization

### Continent-wise Vaccination Coverage

For this visualization, my aim to represent the vaccination coverage on a global scale, categorized by continents.


#### Tableau Visualization

- Utilizing Tableau, I have created an interactive visualization that displays the vaccination status for each continent.
- The visualization includes key metrics such as total vaccinations, vaccinations per capita, and the percentage of the population partially and fully vaccinated.

### Global Vaccination Progress on World Map

To visually represent the global vaccination progress, I have created a dynamic world map visualization.
![vac-p-100](https://github.com/vraj9860/c0vidDATA/assets/141504835/b50067de-f8c9-4085-8ed4-a5f69b1cb94d)

#### Tableau Visualization

- The Tableau world map visualization illustrates the vaccination progress by country.
- Colors represent different levels of vaccination coverage, allowing for a quick assessment of which regions have higher or lower vaccination rates.
  ![gdp-100 (1)](https://github.com/vraj9860/c0vidDATA/assets/141504835/519eb343-140d-4943-8a63-09dce1adaaef)

### Partially and Fully Vaccinated Population

- To gain insights into the percentage of partially and fully vaccinated populations, we've created a visualization that focuses on this specific aspect.
- Also visualization illustrates the proportion of individuals who have received at least one dose (partially vaccinated) and those who have completed the vaccination series (fully vaccinated) globally.
  ![par-ful-lo](https://github.com/vraj9860/c0vidDATA/assets/141504835/cc238b92-01b8-4f8f-9f7b-29820feedbd6)


## Part 2: Death Ratio and Affected Population Visualization

### Death Ratio and Affected Population by Country

- Visualization focuses on portraying the death ratio and affected population for each country using a map-based representation.
  ![Sheet 3](https://github.com/vraj9860/c0vidDATA/assets/141504835/4a03b194-0d3d-4fa2-b542-c648d3a8f8d7)

#### Tableau Visualization

- The Tableau visualization offers a country-level view of the death ratio (percentage of deaths among total cases) and the percentage of the population affected by COVID-19.

![Sheet 4](https://github.com/vraj9860/c0vidDATA/assets/141504835/ab3b321d-1683-4528-8658-96b1c35ca595)



