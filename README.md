# COVID-19 Data Exploration Using SQL Server

![SQL Server](https://img.shields.io/badge/SQL%20Server-Data%20Exploration-red)
![SQL](https://img.shields.io/badge/SQL-Analysis-blue)
![Status](https://img.shields.io/badge/Project-Completed-brightgreen)

## 📌 Project Overview

This project explores global COVID-19 data using **Microsoft SQL Server**.

The goal was to analyze COVID-19 cases, deaths, population infection rates, and vaccination progress across different countries and continents.

I used SQL to transform raw data into meaningful metrics and explore relationships between **COVID-19 cases, deaths, population, and vaccinations**.

The project also demonstrates several intermediate and advanced SQL techniques, including **JOINs, CTEs, temporary tables, window functions, aggregate functions, data-type conversion, and views**.

---

# 🎯 Project Objective

The main objectives of this project were to:

* Analyze COVID-19 cases and deaths over time
* Calculate death percentages for selected countries
* Compare COVID-19 cases with population
* Identify countries with high infection rates relative to population
* Analyze total death counts by country and continent
* Calculate global COVID-19 statistics
* Analyze vaccination progress
* Calculate rolling vaccination totals using window functions
* Create a SQL view for future visualization

---

# 🗂️ Dataset

The project uses two datasets:

### COVID Deaths

`CovidDeaths`

Contains information related to:

* COVID-19 cases
* COVID-19 deaths
* Population
* Location
* Continent
* Date

### COVID Vaccinations

`CovidVaccinations`

Contains information related to:

* Vaccination counts
* Location
* Date
* New vaccinations

The two datasets were joined using:

```sql
ON dea.location = vac.location
AND dea.date = vac.date
```

This allowed COVID-19 death/case information to be analyzed together with vaccination data.

---

# 🛠️ Tools & Technologies

### Tools

* **Microsoft SQL Server**
* **SQL Server Management Studio (SSMS)**

### SQL Skills Demonstrated

* SELECT statements
* Filtering with `WHERE`
* Sorting with `ORDER BY`
* Aggregate functions
* `GROUP BY`
* `JOIN`
* Common Table Expressions (CTEs)
* Temporary tables
* Window functions
* `PARTITION BY`
* `OVER()`
* `SUM()`
* Data-type conversion
* Calculated columns
* Creating SQL Views
* NULL handling

---

# 🔍 Analysis Performed

## 1. Initial Data Exploration

The project began by inspecting the COVID-19 dataset and selecting the fields required for analysis.

```sql
SELECT Location, date, total_cases, new_cases,
       total_deaths, population
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1,2;
```

This provided the foundation for the subsequent analysis.

---

# 2. Total Cases vs. Total Deaths

I calculated the percentage of reported cases that resulted in death.

```sql
SELECT
    Location,
    date,
    total_cases,
    total_deaths,
    (total_deaths / total_cases) * 100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE location LIKE '%states%'
AND continent IS NOT NULL
ORDER BY 1,2;
```

### Metric

**Death Percentage**

```text
Total Deaths / Total Cases × 100
```

This metric provides a way to examine the reported relationship between cases and deaths over time.

> Note: This should not be interpreted as an individual's actual probability of dying after infection because reported cases and deaths are affected by testing, reporting practices, timing, and other factors.

---

# 3. Total Cases vs. Population

I compared reported COVID-19 cases with population size to calculate the percentage of the population represented by reported cases.

```sql
SELECT
    Location,
    date,
    Population,
    total_cases,
    (total_cases / population) * 100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
ORDER BY 1,2;
```

This helped provide population context when comparing locations with different population sizes.

---

# 4. Countries With Highest Infection Rates

I grouped the data by country and calculated the maximum reported case count and corresponding infection percentage.

```sql
SELECT
    Location,
    Population,
    MAX(total_cases) AS HighestInfectionCount,
    MAX((total_cases / population)) * 100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC;
```

This allows countries to be compared using a population-adjusted metric rather than raw case counts alone.

---

# 5. Countries With Highest Death Counts

I analyzed the maximum reported death count for each country.

```sql
SELECT
    Location,
    MAX(CAST(Total_deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY Location
ORDER BY TotalDeathCount DESC;
```

The `CAST()` function was used to convert the death field into an integer for aggregation.

---

# 6. Analysis by Continent

The data was also grouped by continent to compare reported death counts.

```sql
SELECT
    continent,
    MAX(CAST(Total_deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC;
```

This demonstrates how the same metric can be analyzed at different geographic levels.

---

# 7. Global COVID-19 Numbers

I calculated aggregate global case and death figures.

```sql
SELECT
    SUM(new_cases) AS total_cases,
    SUM(CAST(new_deaths AS INT)) AS total_deaths,
    SUM(CAST(new_deaths AS INT))
        / SUM(New_Cases) * 100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL;
```

This provided an overall summary of reported cases, deaths, and the calculated death-to-case percentage in the dataset.

---

# 💉 8. Population vs. Vaccinations

The second major part of the project focused on vaccination data.

I joined the COVID deaths and vaccination datasets and used a window function to calculate cumulative vaccinations by location.

```sql
SELECT
    dea.continent,
    dea.location,
    dea.date,
    dea.population,
    vac.new_vaccinations,

    SUM(CONVERT(INT, vac.new_vaccinations))
        OVER (
            PARTITION BY dea.Location
            ORDER BY dea.location, dea.Date
        ) AS RollingPeopleVaccinated

FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date

WHERE dea.continent IS NOT NULL
ORDER BY 2,3;
```

### Key SQL Technique

The window function:

```sql
SUM(...) OVER (
    PARTITION BY Location
    ORDER BY Date
)
```

creates a **rolling cumulative vaccination total for each location**.

---

# 9. Using a CTE

A Common Table Expression was used to make the rolling vaccination calculation easier to work with.

```sql
WITH PopvsVac AS
(
    SELECT
        dea.continent,
        dea.location,
        dea.date,
        dea.population,
        vac.new_vaccinations,

        SUM(CONVERT(INT, vac.new_vaccinations))
            OVER (
                PARTITION BY dea.Location
                ORDER BY dea.location, dea.Date
            ) AS RollingPeopleVaccinated

    FROM PortfolioProject..CovidDeaths dea
    JOIN PortfolioProject..CovidVaccinations vac
        ON dea.location = vac.location
        AND dea.date = vac.date

    WHERE dea.continent IS NOT NULL
)

SELECT *,
       (RollingPeopleVaccinated / Population) * 100
FROM PopvsVac;
```

The CTE allowed the rolling vaccination value to be referenced in a subsequent calculation.

---

# 10. Using a Temporary Table

I also recreated the vaccination analysis using a temporary table.

```sql
DROP TABLE IF EXISTS #PercentPopulationVaccinated;

CREATE TABLE #PercentPopulationVaccinated
(
    Continent NVARCHAR(255),
    Location NVARCHAR(255),
    Date DATETIME,
    Population NUMERIC,
    New_vaccinations NUMERIC,
    RollingPeopleVaccinated NUMERIC
);
```

The results were then inserted into the temporary table for further calculations.

This demonstrated another approach to storing intermediate results during SQL analysis.

---

# 11. Creating a SQL View

Finally, I created a SQL View containing the vaccination analysis.

```sql
CREATE VIEW PercentPopulationVaccinated AS

SELECT
    dea.continent,
    dea.location,
    dea.date,
    dea.population,
    vac.new_vaccinations,

    SUM(CONVERT(INT, vac.new_vaccinations))
        OVER (
            PARTITION BY dea.Location
            ORDER BY dea.location, dea.Date
        ) AS RollingPeopleVaccinated

FROM PortfolioProject..CovidDeaths dea

JOIN PortfolioProject..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date

WHERE dea.continent IS NOT NULL;
```

The view can then be used as a reusable data source for future visualization work.

---




---










---

# 💡 Key Insights / What I Learned

## 1. SQL can answer complex analytical questions

This project helped me move beyond basic data retrieval and use SQL to calculate meaningful analytical metrics.

## 2. Window functions are extremely useful

Using `SUM() OVER()` and `PARTITION BY` helped me calculate cumulative vaccination totals without collapsing the underlying records.

This was one of the most important SQL techniques I practiced in this project.

## 3. JOINs allow different datasets to work together

By joining the COVID deaths dataset with the vaccination dataset using location and date, I was able to combine information from two different sources and perform a more complete analysis.

## 4. CTEs make complex queries easier to structure

The CTE approach allowed me to separate the calculation of rolling vaccinations from the calculation of vaccination percentages.

This made the query easier to understand and extend.

## 5. Temporary tables provide another way to manage intermediate results

I practiced creating and populating temporary tables when calculations needed to be reused within the analysis.

## 6. SQL Views can support future reporting

Creating a view allowed me to save an analytical query as a reusable data source that could later be connected to a visualization tool.

## 7. Raw numbers need context

Comparing countries using only total cases or total deaths can be misleading because countries have very different population sizes.

Using population-based metrics provides additional context for comparisons.

---

# 📚 What This Project Demonstrates

This project demonstrates my ability to:

* Explore a large dataset
* Identify relevant analytical questions
* Combine multiple datasets
* Calculate derived metrics
* Use aggregate functions
* Use window functions
* Work with CTEs
* Create temporary tables
* Create reusable SQL views
* Convert data types
* Prepare analytical data for visualization

---

# 🚀 Future Improvements

I plan to extend this project by:

* Building an interactive **Power BI dashboard**
* Adding more advanced SQL analysis
* Creating additional time-series metrics
* Improving query performance
* Adding data-quality checks
* Creating a more interactive country-level analysis
* Documenting the data pipeline from raw data to dashboard

---

# 👨‍💻 Author

**[Your Name]**

Aspiring Data Analyst | SQL | Power BI | Excel | Python

This project is part of my data analytics portfolio and demonstrates my ability to explore, transform, and analyze real-world datasets using SQL Server.
