/*
COVID-19 Data Exploration
Skills used: Joins, CTEs, Temporary Tables, Window Functions, Aggregate Functions, Creating Views, Converting Data Types
*/

-- Selecting data from CovidDeaths table

SELECT *
FROM CovidPortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 3, 4;

-- Selecting initial data to work with

SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM CovidPortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;

-- Analyzing total cases vs total deaths

SELECT Location, date, total_cases, total_deaths, (total_deaths / total_cases) * 100 AS DeathPercentage
FROM CovidPortfolioProject..CovidDeaths
WHERE location LIKE '%states%' AND continent IS NOT NULL
ORDER BY 1, 2;

-- Analyzing total cases vs population

SELECT Location, date, population, total_cases, (total_cases / population) * 100 AS PercentPopulationInfected
FROM CovidPortfolioProject..CovidDeaths
WHERE location LIKE '%states%'
ORDER BY 1, 2;

-- Countries with highest infection rate compared to population

SELECT Location, population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases / population) * 100) AS PercentPopulationInfected
FROM CovidPortfolioProject..CovidDeaths
GROUP BY location, population
ORDER BY PercentPopulationInfected DESC;

-- Countries with the highest death count per population

SELECT Location, MAX(CAST(total_deaths AS INT)) AS TotalDeathCount
FROM CovidPortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location
ORDER BY TotalDeathCount DESC;

-- Analyzing continents with the highest death count per population

SELECT continent, MAX(CAST(total_deaths AS INT)) AS TotalDeathCount
FROM CovidPortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
GROUP BY continent
ORDER BY TotalDeathCount DESC;

-- Global numbers

SELECT SUM(new_cases) AS total_cases, SUM(CAST(new_deaths AS INT)) AS total_deaths, SUM(CAST(new_deaths AS INT)) / SUM(New_Cases) * 100 AS DeathPercentage
FROM CovidPortfolioProject..CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1, 2;

-- Analyzing total population vs vaccinations using CTE

WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, PeopleVaccinated) AS
(
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(CAST(vac.new_vaccinations AS INT))
OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS PeopleVaccinated
FROM CovidPortfolioProject..CovidDeaths dea
JOIN CovidPortfolioProject..CovidVaccinations vac
ON dea.location = vac.location
AND dea.date = vac.date
WHERE dea.continent IS NOT NULL
)
SELECT *, (PeopleVaccinated / Population) * 100
FROM PopvsVac;

-- Analyzing total population vs vaccinations using temporary table

DROP TABLE IF EXISTS #PercentPeopleVaccinated;

CREATE TABLE #PercentPeopleVaccinated
(
Continent NVARCHAR(255),
Location NVARCHAR(255),
Date DATETIME,
Population NUMERIC,
New_Vaccinations NUMERIC,
PeopleVaccinated NUMERIC
);

INSERT INTO #PercentPeopleVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(CAST
