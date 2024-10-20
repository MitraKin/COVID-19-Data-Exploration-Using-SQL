<h1>COVID-19 Data Exploration using SQL</h1>

<p>This project explores COVID-19 data, specifically focusing on deaths, infections, and vaccinations across different countries and continents. Using various SQL techniques such as joins, common table expressions (CTEs), temporary tables, window functions, aggregate functions, data type conversions, and views, this analysis provides insights into the global COVID-19 situation. The dataset includes information on total cases, deaths, population, and vaccinations, sourced from the <strong>CovidDeaths</strong> and <strong>CovidVaccinations</strong> tables within the <code>PortfolioProject</code> database.</p>

<h2>Skills Demonstrated:</h2>
<ul>
  <li>Joins</li>
  <li>Common Table Expressions (CTEs)</li>
  <li>Temporary Tables</li>
  <li>Window Functions</li>
  <li>Aggregate Functions</li>
  <li>Creating Views</li>
  <li>Data Type Conversions</li>
</ul>

<hr>

<h2>Project Outline</h2>

<h3>1. Exploring the Dataset</h3>
<p>We begin by previewing the dataset and identifying the main attributes such as the location, date, total cases, new cases, total deaths, and population.</p>

<pre><code>
SELECT * 
FROM PortfolioProject..CovidDeaths 
WHERE continent IS NOT NULL 
ORDER BY 3, 4;
</code></pre>

<p>To focus the analysis, the following key columns were selected for subsequent queries:</p>
<ul>
  <li>Location</li>
  <li>Date</li>
  <li>Total Cases</li>
  <li>New Cases</li>
  <li>Total Deaths</li>
  <li>Population</li>
</ul>

<pre><code>
SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL 
ORDER BY 1, 2;
</code></pre>

<h3>2. Total Cases vs Total Deaths</h3>
<p>This query calculates the <strong>death percentage</strong> (likelihood of death if infected by COVID-19) for locations containing "states" in their names.</p>

<pre><code>
SELECT Location, date, total_cases, total_deaths, 
       (total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE location LIKE '%states%'
AND continent IS NOT NULL 
ORDER BY 1, 2;
</code></pre>

<h3>3. Total Cases vs Population</h3>
<p>We calculated the <strong>percentage of the population infected</strong> by dividing the total cases by the population size.</p>

<pre><code>
SELECT Location, date, Population, total_cases,  
       (total_cases/population)*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
ORDER BY 1, 2;
</code></pre>

<h3>4. Countries with the Highest Infection Rate (Per Population)</h3>
<p>This query identifies countries with the <strong>highest infection rate</strong> in relation to their population by calculating the highest infection count per country.</p>

<pre><code>
SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount,  
       MAX((total_cases/population))*100 AS PercentPopulationInfected
FROM PortfolioProject..CovidDeaths
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC;
</code></pre>

<h3>5. Countries with the Highest Death Count</h3>
<p>This query lists countries with the <strong>highest number of deaths</strong> due to COVID-19.</p>

<pre><code>
SELECT Location, MAX(CAST(Total_deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL 
GROUP BY Location
ORDER BY TotalDeathCount DESC;
</code></pre>

<h3>6. Breakdown by Continent</h3>
<p>We further break down the data by <strong>continent</strong>, showing which continents have the highest total deaths.</p>

<pre><code>
SELECT continent, MAX(CAST(Total_deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL 
GROUP BY continent
ORDER BY TotalDeathCount DESC;
</code></pre>

<h3>7. Global Numbers</h3>
<p>This query aggregates global totals for <strong>new cases, deaths, and death percentage</strong>.</p>

<pre><code>
SELECT SUM(new_cases) AS total_cases, 
       SUM(CAST(new_deaths AS INT)) AS total_deaths, 
       SUM(CAST(new_deaths AS INT))/SUM(New_Cases)*100 AS DeathPercentage
FROM PortfolioProject..CovidDeaths
WHERE continent IS NOT NULL 
ORDER BY 1, 2;
</code></pre>

<h3>8. Total Population vs Vaccinations</h3>
<p>We explore the <strong>percentage of the population vaccinated</strong> by calculating the rolling sum of vaccinations and dividing by the population size.</p>

<pre><code>
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
       SUM(CONVERT(INT, vac.new_vaccinations)) OVER (Partition BY dea.Location 
                                                     ORDER BY dea.location, dea.Date) 
       AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
ON dea.location = vac.location
AND dea.date = vac.date
WHERE dea.continent IS NOT NULL 
ORDER BY 2, 3;
</code></pre>

<h3>9. Using CTE (Common Table Expressions) to Simplify Queries</h3>
<p>We create a CTE to simplify the previous vaccination calculation and show the percentage of vaccinated individuals.</p>

<pre><code>
WITH PopvsVac AS (
    SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
           SUM(CONVERT(INT, vac.new_vaccinations)) OVER (Partition BY dea.Location 
                                                         ORDER BY dea.location, dea.Date) 
           AS RollingPeopleVaccinated
    FROM PortfolioProject..CovidDeaths dea
    JOIN PortfolioProject..CovidVaccinations vac
    ON dea.location = vac.location
    AND dea.date = vac.date
    WHERE dea.continent IS NOT NULL 
)
SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentVaccinated
FROM PopvsVac;
</code></pre>

<h3>10. Using Temporary Tables</h3>
<p>This section uses a temporary table to store the percentage of the population vaccinated for later use.</p>

<pre><code>
DROP TABLE IF EXISTS #PercentPopulationVaccinated;
CREATE TABLE #PercentPopulationVaccinated (
    Continent NVARCHAR(255),
    Location NVARCHAR(255),
    Date DATETIME,
    Population NUMERIC,
    New_vaccinations NUMERIC,
    RollingPeopleVaccinated NUMERIC
);

INSERT INTO #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
       SUM(CONVERT(INT, vac.new_vaccinations)) OVER (Partition BY dea.Location 
                                                     ORDER BY dea.location, dea.Date) 
       AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
ON dea.location = vac.location
AND dea.date = vac.date;

SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentVaccinated
FROM #PercentPopulationVaccinated;
</code></pre>

<h3>11. Creating Views for Visualization</h3>
<p>To enable easier access to the calculated data for future visualizations, we created a view that stores the percentage of the population vaccinated.</p>

<pre><code>
CREATE VIEW PercentPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
       SUM(CONVERT(INT, vac.new_vaccinations)) OVER (Partition BY dea.Location 
                                                     ORDER BY dea.location, dea.Date) 
       AS RollingPeopleVaccinated
FROM PortfolioProject..CovidDeaths dea
JOIN PortfolioProject..CovidVaccinations vac
ON dea.location = vac.location
AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
</code></pre>

<hr>

<h2>Conclusion</h2>
<p>This project provided key insights into the global spread of COVID-19, including:</p>
<ul>
  <li>The percentage of populations infected.</li>
  <li>The mortality rate among those infected.</li>
  <li>Global trends in vaccination efforts and coverage.</li>
</ul>

<p>By using SQL techniques such as joins, CTEs, temp tables, and window functions, we were able to effectively analyze the large COVID-19 dataset and uncover critical patterns that are useful for understanding the pandemic's impact on various regions.</p>
