# COVID-19 Data Exploration with Tableau

This project explores and visualizes COVID-19 data using SQL queries and Tableau. The queries extract key metrics related to cases, deaths, and vaccinations, which are then used to create interactive dashboards in Tableau.

## Project Objectives

* To provide insights into the global spread and impact of COVID-19.
* To demonstrate the use of SQL for data extraction and transformation.
* To showcase the power of Tableau for data visualization and analysis.

## Installation Instructions

### Database Setup

1.  **Microsoft SQL Server:**
    * Ensure you have Microsoft SQL Server installed on your machine. If not, download and install it from the official Microsoft website.
2.  **Database Creation:**
    * Open SQL Server Management Studio (SSMS).
    * Connect to your SQL Server instance.
    * Create a new database (e.g., `PortfolioProject1`). You can do this by right-clicking on "Databases" and selecting "New Database...".
3.  **Data Import:**
    * Download the `CovidDeaths.csv` and `CovidVaccinations.csv` files (or similar data source).
    * Import these CSV files into your `PortfolioProject1` database as tables named `CovidDeaths$` and `CovidVaccinations$`. You can use the SQL Server Import and Export Wizard or SQL Server Management Studio's import functionality.

### Tableau Setup

1.  **Tableau Installation:**
    * Install Tableau Desktop or Tableau Public from the official Tableau website.
2.  **Database Connection:**
    * Open Tableau.
    * Connect to your Microsoft SQL Server database using the connection options provided by Tableau.
    * Select the `PortfolioProject1` database.

## Usage Instructions

### SQL Queries

1.  **Execution:**
    * Open SQL Server Management Studio (SSMS).
    * Connect to your SQL Server instance and select the `PortfolioProject1` database.
    * Execute the provided SQL queries to verify the data and understand the metrics.
2.  **Purpose:**
    * These queries are designed to extract and transform the data required for visualization in Tableau. They provide the underlying data for the Tableau dashboards.

### Tableau Visualization

1.  **Connection:**
    * Open Tableau and connect to your SQL Server database.
2.  **Visualization Creation:**
    * Create visualizations (charts, maps, etc.) using the data extracted by the SQL queries.
    * Use the provided queries to build the worksheets within tableau.
3.  **Dashboard Development:**
    * Build interactive dashboards to present the insights in a clear and engaging manner.
    * Utilize filters and parameters to allow for user exploration.

## Query Summary

Here's a breakdown of the SQL queries included in this project:

* **Total Cases and Deaths:**
    * Calculates the total number of COVID-19 cases and deaths worldwide.
    * Computes the death percentage (deaths/cases * 100).
    * ```sql
        Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage From PortfolioProject1..CovidDeaths$ where continent is not null order by 1,2
        ```
* **Total Deaths by Country:**
    * Calculates the total death count for each country.
    * Excludes "World," "European Union," and "International" locations.
    * ```sql
        Select location, SUM(cast(new_deaths as int)) as TotalDeathCount From PortfolioProject1..CovidDeaths$ Where continent is null and location not in ('World', 'European Union', 'International') Group by location order by TotalDeathCount desc
        ```
* **Highest Infection Count:**
    * Determines the highest infection count for each country.
    * Calculates the percentage of the population infected.
    * ```sql
        Select Location, Population, MAX(total_cases) as HighestInfectionCount, Max((total_cases/population))*100 as PercentPopulationInfected From PortfolioProject1..CovidDeaths$ Group by Location, Population order by PercentPopulationInfected desc
        ```
* **Daily Infection Rate:**
    * Shows the daily infection rate and highest infection count by location.
    * ```sql
        Select Location, Population,date, MAX(total_cases) as HighestInfectionCount, Max((total_cases/population))*100 as PercentPopulationInfected From PortfolioProject1..CovidDeaths$ Group by Location, Population, date order by PercentPopulationInfected desc
        ```
* **Original Queries:**
    * The second set of queries provided in the sql file are the original queries used in the project,  They are provided as reference.

## License

This project is intended for educational and personal use. You are free to use and modify the code and visualizations for your own purposes. However, please acknowledge the source and provide attribution if you share or distribute the project.
