# **Summary**
This project involves exploring global Covid-19 data from Our World in Data using MySQL. The primary focus is to aggregate, transform, and integrate data to support comprehensive analysis and visualization, helping to identify trends, disparities, and insights for informed decision-making.

## **Project Overview**
- Data Source: [Our World in Data](https://ourworldindata.org/covid-deaths)
- Tools Used: MySQL, Excel
- Data Size: 414,812 rows


## **Project Objectives**
- Aggregate and transform Covid-19 data to facilitate analysis.
- Develop data models to extract actionable insights.
- Conduct comparative analysis across regions to identify trends and disparities.
- Create views and manage data for efficient retrieval and visualization.


## **Key Analysis and Queries**

1. Database Setup:
```sql
DROP DATABASE IF EXISTS PortfolioProject;
CREATE DATABASE PortfolioProject;
USE PortfolioProject;
```

2. Initial Data Exploration:
```sql
SELECT * FROM coviddeaths WHERE continent IS NOT NULL ORDER BY 3, 4;
SELECT * FROM covidvaccinations ORDER BY 3, 4;
```

3. Total Cases and Deaths Analysis:

```sql
SELECT Location, date, total_cases, total_deaths, (total_deaths/total_cases * 100) AS DeathPercentage
FROM coviddeaths
WHERE location LIKE '%states%' AND continent IS NOT NULL
ORDER BY Location, date;
```

4. Infection Rate Analysis:

```sql
SELECT Location, population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/population * 100)) AS Percentage
FROM coviddeaths
WHERE continent IS NOT NULL
GROUP BY Location, population
ORDER BY Percentage DESC;
```

5. Death Count Analysis:


```sql
SELECT Location, MAX(CAST(Total_deaths AS float)) AS TotalDeathCount
FROM coviddeaths
WHERE continent IS NOT NULL
GROUP BY Location
ORDER BY TotalDeathCount DESC;
```

6. Continental Death Count Analysis:

```sql
SELECT continent, MAX(CAST(Total_deaths AS float)) AS TotalDeathCount
FROM coviddeaths
WHERE continent IS NOT NULL
GROUP BY Location
ORDER BY continent DESC;
```

7. Global Summary:
```sql
SELECT SUM(new_cases) AS total_cases, SUM(CAST(new_deaths AS float)) AS total_deaths,
SUM(CAST(new_deaths AS float))/SUM(New_Cases)*100 AS DeathPercentage
FROM coviddeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;
```

8. Vaccination Analysis:
```sql
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CAST(vac.new_vaccinations AS float)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM coviddeaths dea
JOIN covidvaccinations vac
ON dea.location = vac.location AND dea.date = vac.date;
```

8. Using CTE for Partition Calculations:
```sql
WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated) AS (
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
    SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
    FROM coviddeaths dea
    JOIN covidvaccinations vac
    ON dea.location = vac.location AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL
)
SELECT *, (RollingPeopleVaccinated/Population)*100
FROM PopvsVac;
```

9. Using Temp Table for Partition Calculations:
```sql
DROP TABLE IF EXISTS PercentPopulationVaccinated;
CREATE TEMPORARY TABLE PercentPopulationVaccinated (
    Continent NVARCHAR(255),
    Location NVARCHAR(255),
    Date DATETIME,
    Population INT,
    New_vaccinations INT,
    RollingPeopleVaccinated INT
);

INSERT INTO PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM coviddeaths dea
JOIN covidvaccinations vac
ON dea.location = vac.location AND dea.date = vac.date;

SELECT *, (RollingPeopleVaccinated/Population)*100
FROM PercentPopulationVaccinated;
```
10. Creating View for Visualization:
```sql
CREATE VIEW PercentPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations,
SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM coviddeaths dea
JOIN covidvaccinations vac
ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
```

## **Data Sample**
The dataset includes the following columns:
- iso_code
- continent
- location
- date
- total_cases
- new_cases
- new_cases_smoothed
- total_deaths
- new_deaths
- new_deaths_smoothed
- total_cases_per_million
- new_cases_per_million
- new_cases_smoothed_per_million
- total_deaths_per_million
- new_deaths_per_million
- new_deaths_smoothed_per_million
- reproduction_rate
- icu_patients
- icu_patients_per_million
- hosp_patients
- hosp_patients_per_million
- weekly_icu_admissions
- weekly_icu_admissions_per_million
- weekly_hosp_admissions
- weekly_hosp_admissions_per_million
- total_tests
- new_tests
- total_tests_per_thousand
- new_tests_per_thousand
- new_tests_smoothed
- new_tests_smoothed_per_thousand
- positive_rate
- tests_per_case
- tests_units
- total_vaccinations
- people_vaccinated
- people_fully_vaccinated
- total_boosters
- new_vaccinations
- new_vaccinations_smoothed
- total_vaccinations_per_hundred
- people_vaccinated_per_hundred
- people_fully_vaccinated_per_hundred
- total_boosters_per_hundred
- new_vaccinations_smoothed_per_million
- new_people_vaccinated_smoothed
- new_people_vaccinated_smoothed_per_hundred
- stringency_index
- population_density
- median_age
- aged_65_older
- aged_70_older
- gdp_per_capita
- extreme_poverty
- cardiovasc_death_rate
- diabetes_prevalence
- female_smokers
- male_smokers
- handwashing_facilities
- hospital_beds_per_thousand
- life_expectancy
- human_development_index
- population
- excess_mortality_cumulative_absolute
- excess_mortality_cumulative
- excess_mortality
- excess_mortality_cumulative_per_million
