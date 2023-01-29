# PortfolioCovidProject
--Select Data that we are going to use 
select Location, date,total_cases, new_cases, total_deaths,population from `Covid2020_2023.Final_Portfolio_Data` order by 1,2;

--Looking at total cases vs total deaths
select Location, date,total_cases, total_deaths,round(((total_deaths/total_cases)*100),2) AS Death_Percentage from `Covid2020_2023.Final_Portfolio_Data` order by 1,2;

--Looking Into India's Database
--Death Pecentage (Likelihood of Death if contracted with Covid in India)
select Location, date,total_cases, total_deaths,round(((total_deaths/total_cases)*100),2) AS Death_Percentage from `Covid2020_2023.Final_Portfolio_Data` where location like '%India%' order by 1,2;

--Looking into total cases w.r.t. Population in India
select Location, date,total_cases, population,round(((total_cases/population)*100),4) AS Case_Percentage from `Covid2020_2023.Final_Portfolio_Data` where location like '%India%' order by 1,2;

--Looking into total cases w.r.t. Population around the world
select Location, date,total_cases, population,round(((total_cases/population)*100),4) AS Case_Percentage from `Covid2020_2023.Final_Portfolio_Data` 
--where location like '%India%' 
order by 1,2;

--Looking at countries with Highest Infection rate compared to Population 
select Distinct Location, date,total_cases, population,round(((total_cases/population)*100),4) AS Case_Percentage from `Covid2020_2023.Final_Portfolio_Data` 
--where location like '%India%' 
order by Case_Percentage DESC,1,2;

--Looking into the highest with Highest Infection rate compared to Population at any time of the given period 
select Distinct Location, population,max(total_cases),Max(round(((total_cases/population)*100),4)) AS Case_Percentage from `Covid2020_2023.Final_Portfolio_Data` group by location,population order by Case_Percentage DESC,1,2;

--Showing Countries with highest Death Count per population
select Distinct Location, population,Max(CAST(total_deaths AS INT64)) as Max_Deaths,Max(round(((total_deaths/population)*100),2)) AS Death_Percentage from `Covid2020_2023.Final_Portfolio_Data` where continent is not null group by location,population order by Death_Percentage DESC,1,2;


--Showing Continent with highest Death Count per population
select continent ,Max(CAST(total_deaths AS INT64)) as Max_Deaths_Continent,Max(round(((total_deaths/population)*100),2)) AS Death_Percentage 
from `Covid2020_2023.Final_Portfolio_Data` 
where continent is not null 
group by continent 
order by Death_Percentage DESC,1,2;

--Actual Projections
--Global Numbers

select date,SUM(total_cases),Max(Round(((total_deaths/population)*100),2)) AS Death_Percentage 
from `Covid2020_2023.Final_Portfolio_Data` 
where continent is not null and total_cases is not null 
group by date order by date;

-- Total Population vs Vaccinations
select d.continent,d.location,d.date,d.population,v.new_vaccinations,
Sum(v.new_vaccinations) over (partition by d.location order by d.location,d.date) as progressive_vaccination 
FROM `Covid2020_2023.Final_Portfolio_Data` d
JOIN `Covid2020_2023.Final Covid Vaccination`v
on d.location=v.location and d.date=v.date
where d.continent is not null and d.total_cases is not null and v.new_vaccinations is not null
group by d.continent,d.location,d.date,d.population,v.new_vaccinations
order by 1,2
;


With PopVSVac AS 
(SELECT
d.continent,d.location,d.date,d.population,v.new_vaccinations,
SUM(v.new_vaccinations) over (partition by d.location order by d.location,d.date) AS Progressive_Vaccination 
FROM `Covid2020_2023.Final_Portfolio_Data` d
JOIN `Covid2020_2023.Final Covid Vaccination`v
on d.location=v.location and d.date=v.date
where d.continent is not null and d.total_cases is not null and v.new_vaccinations is not null
--group by d.continent,d.location,d.date,d.population,v.new_vaccinations
order by 2,3
)
SELECT *, Round(((Progressive_Vaccination/population)*100),2) As popvsvacci from PopVSVac;
