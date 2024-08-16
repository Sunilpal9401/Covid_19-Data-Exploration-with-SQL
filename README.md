
## Covid_19-Data-Exploration-with-SQL

Covid 19 pandemic have had several short-term as well as long term impacts on human health, society, economy and environment. A pandemic which broke out in 2019 created significant knock-on effects on the daily life of citizens, as well as about the global economy.

Presently the impacts of COVID-19 in daily life are extensive and have far reaching consequences.

In this project, I analyzed and explored covid data with SQL using Micrsoft SQL Server Management Studio.

This covid-19 data was sourced from Our World in Data starting from 1st January, 2020 to 5th November, 2022.
## Summary of Findings

According to the insights derived from the covid data :

1.The total covid-19 cases confirmed was 630,711,800 cases

2.The total deaths recorded was 6,561,569

3.The global percentage death was 1.04 percent

4.United satates, India, France, and Brazil had the total number of confirmed cases.

5.United states, Brazil, India, Russia and Mexico recorded the highest deaths.

6.Europe had the highest total cases per continrent, followed by Asia, North merica, and South America.

7.Europe had the total number of deaths recorded as per continent, followed by North America, Asia, and South America.
## Screenshots of Dataset of Covid_19

![App Screenshot](https://github.com/Sunilpal9401/Covid_19-Data-Exploration-with-SQL/blob/main/Data%20Snapshot/1.jpg?raw=true)



Covid 19 Data Exploration 

Date Range: FROM 1st January, 2020 t0 5th November, 2022.

Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types


ANALYSIS OF COVID DATA FROM 1st January, 2020 t0 5th November, 2022.
```sql
SELECT *
FROM CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 3, 4;
```
```sql
SELECT *
FROM CovidVaccinations
ORDER BY 3, 4;
```

SELECT DATA WE ARE GOING TO BE STARTING WITH
```sql
SELECT Location, Date, total_cases, new_cases, total_deaths, population
FROM CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;
```


LOOKING AT THE TOTAL CASES VS TOTAL DEATHS
Shows Likelihood of dying if you contact covid in your country
```sql
SELECT Location, Date, total_cases, total_deaths, (total_deaths/total_cases) * 100 AS DeathPercentage
FROM CovidDeaths
WHERE Location LIKE '%states'
AND continent IS NOT NULL
ORDER BY 1, 2;
```
```sql
SELECT Location, Date, total_cases, total_deaths, (total_deaths/total_cases) * 100 AS DeathPercentage
FROM CovidDeaths
WHERE Location LIKE '%Nigeria%'
AND continent IS NOT NULL
ORDER BY 1, 2;
```

LOOKING AT THE TOTAL CASES VS POPULATION
Shows Percentage of Population infected with covid
```sql
SELECT Location, Date, population,  total_cases, (total_cases/population) * 100  AS PercentPopulationInfected
FROM CovidDeaths
WHERE continent IS NOT NULL
-- WHERE Location LIKE '%states'
ORDER  BY 1, 2;
```
```sql
SELECT Location, Date, population, total_cases, (total_cases/population) * 100  AS PercentPopulationInfected
FROM CovidDeaths
WHERE continent IS NOT NULL
AND Location LIKE '%Nigeria%'
ORDER  BY 1, 2;
```

LOOKING AT COUNTRIES WITH HIGHEST INFECTION RATE COMPARED TO POPULATION
```sql
SELECT Location, population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/population)) * 100 AS PercentPopulationInfected 
FROM CovidDeaths
WHERE continent IS NOT NULL
--WHERE Location LIKE '%states'
GROUP BY Location, population
ORDER BY  PercentPopulationInfected DESC;
```

SHOWING COUNTRIES WITH HIGHEST DEATH COUNT PER POPULATION
```sql
SELECT Location, MAX(cast(total_deaths AS int)) AS TotalDeathCount
FROM CovidDeaths
--WHERE Location LIKE '%states'
WHERE continent IS NOT NULL
GROUP BY Location
ORDER BY TotalDeathCount DESC;
```

LET'S BREAK IT DOWN BY CONTINENT

NB:
This method kind of gives you the accurate result if you want to break it down to continent
We use where continent is null because if you check in the excel sheet, the locations has the names of the continent in them, 
instead of the names of the countries.
```sql
SELECT Location, MAX(cast(total_deaths AS int)) AS TotalDeathCount
FROM CovidDeaths
WHERE continent IS NULL
--WHERE Location LIKE '%states'
GROUP BY Location
ORDER BY TotalDeathCount DESC;
``

But we will use this for now for the sake of visualization purposes
```sql
SELECT continent, MAX(cast(total_deaths AS int)) AS TotalDeathCount
FROM CovidDeaths
WHERE continent IS NOT NULL
--WHERE Location LIKE '%states'
GROUP BY continent
ORDER BY TotalDeathCount DESC;
```

GLOBAL NUMBERS
```sql
SELECT Date, SUM(new_cases) AS total_cases, SUM(cast(new_deaths AS int)) AS total_deaths, SUM(cast(new_deaths AS int))/SUM(new_cases) * 100 AS DeathPercentage
FROM CovidDeaths
--WHERE Location LIKE '%states'
WHERE continent IS NOT NULL
GROUP BY Date
ORDER BY 1, 2;
```
```sql
SELECT SUM(new_cases) AS total_cases, SUM(cast(new_deaths AS int)) AS total_deaths, SUM(cast(new_deaths AS int))/SUM(new_cases) * 100 AS DeathPercentage
FROM CovidDeaths
--WHERE Location LIKE '%states'
WHERE continent IS NOT NULL
ORDER BY 1, 2;
```

ASSESSING VACCINATION DATA
```sql
SELECT *
FROM CovidVaccinations;
```

JOINING THE TWO TABLES TOGETHER FOR VIEWING
```sql
SELECT * 
FROM CovidDeaths death
JOIN CovidVaccinations vac
	ON death.location = vac.location
	AND death.date = vac.date;
```

LOOKING AT THE TOTAL POPULATION VS VACCINATIONS
Shows Percentage of Population that has recieved at least one Covid Vaccine
```sql
SELECT death.continent, death.location, death.date, death.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS bigint)) OVER 
(PARTITION BY death.location ORDER BY death.location, death.date) AS RollingPeopleVaccinated
FROM  CovidDeaths death
JOIN CovidVaccinations vac
	ON death.location = vac.location
	AND death.date = vac.date
WHERE death.continent IS NOT NULL
ORDER BY 2,3;
```

Getting the percentage of RollingPeopleVaccinated for each location

Using CTE to perform Calculation on Partition By in previous query
```sql
WITH PopvsVac (Continent, Location, Date, Population, New_vaccination, RollingPeopleVaccinated)
AS
(
SELECT death.continent, death.location, death.date, death.population, vac.new_vaccinations,
SUM(CAST(vac.new_vaccinations AS bigint)) OVER (PARTITION BY death.location ORDER BY death.location, death.date) AS RollingPeopleVaccinated
FROM  CovidDeaths death
JOIN CovidVaccinations vac
	ON death.location = vac.location
	AND death.date = vac.date
WHERE death.continent IS NOT NULL
--ORDER BY 2,3
)
SELECT *, (RollingPeopleVaccinated/Population) * 100 AS RollingPeopleVaccinatedPercentage
FROM PopvsVac;
```


Using Temp Table to perform Calculation on Partition By in previous query
```sql
DROP TABLE IF EXISTS  #PercentPopulationVaccinated
CREATE TABLE #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_Vaccinations numeric,
RollingPeopleVaccinated numeric
)
INSERT INTO #PercentPopulationVaccinated
SELECT death.continent, death.location, death.date, death.population, vac.new_vaccinations, 
SUM(CAST(vac.new_vaccinations AS bigint)) OVER (PARTITION BY death.location ORDER BY death.location, death.date) AS RollingPeopleVaccinated
FROM  CovidDeaths death
JOIN CovidVaccinations vac
	ON death.location = vac.location
	AND death.date = vac.date
--WHERE death.continent IS NOT NULL
--ORDER BY 2,3

SELECT *, (RollingPeopleVaccinated/Population) * 100 AS PercentRollingPeopleVaccinated
FROM #PercentPopulationVaccinated;


```

CREATING VIEW TO STORE DATA FOR DATA VISUALIZATION
```sql
CREATE VIEW PercentPopulationVaccinated AS
SELECT death.continent, death.location, death.date, death.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS bigint)) OVER 
(PARTITION BY death.location ORDER BY death.location, death.date) AS RollingPeopleVaccinated
FROM  CovidDeaths death
JOIN CovidVaccinations vac
	ON death.location = vac.location
	AND death.date = vac.date
WHERE death.continent IS NOT NULL
--ORDER BY 2,3

SELECT *
FROM PercentPopulationVaccinated;

```












