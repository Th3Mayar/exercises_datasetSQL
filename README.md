## Documentation for Dataset Operations with SQLite and Pandas

This documentation describes the process of working with datasets using **SQLite** and **Pandas** in Python, including loading data, performing SQL queries, and displaying results. The example focuses on creating an in-memory SQLite database and working with data related to cities, employees, countries, and departments.

### Libraries Used
- `sqlite3`: Python's built-in library for interacting with SQLite databases.
- `pandas`: A powerful data analysis library used for reading datasets and handling data frames.

### Connecting to an In-Memory SQLite Database
```python
import sqlite3
import pandas as pd

conn = sqlite3.connect(":memory:")
```
This establishes a connection to an in-memory SQLite database, which is temporary and exists only during the runtime of the script.

### Loading Datasets into SQLite
1. **Cities Dataset**
   - URL: `https://raw.githubusercontent.com/datasets/world-cities/master/data/world-cities.csv`
   - This dataset contains city names and their corresponding countries.
   - It is loaded into a SQLite table named `cities`.

2. **Employees Dataset**
   - URL: `https://raw.githubusercontent.com/justmarkham/DAT8/master/data/u.user`
   - This dataset includes user data with attributes like user ID, age, and occupation.
   - It is loaded into a SQLite table named `employees`.

3. **Countries Dataset**
   - Contains information on countries, their regions, and populations.
   - It is loaded into a table named `countries`.

4. **Departments Dataset**
   - Contains the relationship between occupations and department names, along with the budget for each department.
   - It is loaded into a table named `departments`.

### Function for Executing SQL Queries
A helper function `run_query` is defined to execute SQL queries and return the results as Pandas DataFrames.
```python
def run_query(query):
    return pd.read_sql_query(query, conn)
```

### SQL Queries and Exercises
This section describes various SQL queries executed using the datasets, demonstrating different operations such as aggregations, joins, CTEs (Common Table Expressions), and window functions.

#### (A) Basic Queries
1. **Count Cities by Country**  
   Query to count the number of cities per country where the count is greater than 2.
   ```sql
   SELECT country, COUNT(name) AS city_count
   FROM cities
   GROUP BY country
   HAVING city_count > 2;
   ```

2. **List of Employees and Departments**  
   Query to list employees with their corresponding departments using a LEFT JOIN.
   ```sql
   SELECT e.user_id, e.occupation, d.department_name
   FROM employees e
   LEFT JOIN departments d ON e.occupation = d.occupation;
   ```

3. **Average Age by Occupation**  
   Query to calculate the average age for each occupation.
   ```sql
   SELECT occupation, AVG(age) AS average_age
   FROM employees
   GROUP BY occupation;
   ```

4. **Cities and Regions**  
   Query to list cities along with their regions by joining `cities` and `countries`.
   ```sql
   SELECT c.name, co.region
   FROM cities c
   JOIN countries co ON c.country = co.country;
   ```

#### (B) Advanced Queries
1. **Number of Cities per Region with CTE**  
   Using a CTE to count the number of cities per region.
   ```sql
   WITH RegionCities AS (
       SELECT co.region, COUNT(c.name) AS city_count
       FROM countries co
       JOIN cities c ON co.country = c.country
       GROUP BY co.region
   )
   SELECT region, city_count
   FROM RegionCities
   WHERE city_count > 3;
   ```

2. **Employee Age Ranking by Occupation with Window Functions**  
   Ranking employees by age within each occupation using the `RANK()` window function.
   ```sql
   SELECT user_id, occupation, age,
          RANK() OVER (PARTITION BY occupation ORDER BY age DESC) AS age_rank
   FROM employees
   WHERE age IS NOT NULL
   ORDER BY occupation, age_rank;
   ```

3. **Comparison of Cities and Population by Region**  
   Query to list regions with more than 3 cities and a total population greater than 50 million.
   ```sql
   SELECT co.region, COUNT(c.name) AS city_count, SUM(co.population_millions) AS total_population
   FROM countries co
   JOIN cities c ON co.country = c.country
   GROUP BY co.region
   HAVING city_count > 3 AND total_population > 50;
   ```

4. **Employees Older than the Overall Average in Their Department**  
   Query to find employees older than the average age within their occupation.
   ```sql
   SELECT user_id, occupation, age
   FROM employees e1
   WHERE age > (SELECT AVG(age) FROM employees e2 WHERE e1.occupation = e2.occupation);
   ```

#### (C) Complex Queries
1. **Countries with the Highest Number of Cities per Region**  
   Using a CTE and `RANK()` to find the country with the most cities in each region.
   ```sql
   WITH CountryCityCount AS (
       SELECT co.country, co.region, COUNT(c.name) AS city_count
       FROM countries co
       JOIN cities c ON co.country = c.country
       GROUP BY co.country, co.region
   ),
   RankedCountries AS (
       SELECT country, region, city_count,
              RANK() OVER (PARTITION BY region ORDER BY city_count DESC) AS city_rank
       FROM CountryCityCount
   )
   SELECT country, region, city_count
   FROM RankedCountries
   WHERE city_rank = 1;
   ```

2. **Comparison of Budgets and Number of Employees by Department**  
   Query to identify departments where the budget per employee is less than 0.1 million.
   ```sql
   SELECT d.department_name,
          COUNT(e.user_id) AS num_employees,
          d.budget_millions,
          (d.budget_millions / COUNT(e.user_id)) AS budget_per_employee
   FROM departments d
   LEFT JOIN employees e ON d.occupation = e.occupation
   GROUP BY d.department_name
   HAVING budget_per_employee < 0.1;
   ```

### Conclusion
This example demonstrates how to work with different datasets using SQLite and Pandas for data analysis and querying. It covers a range of SQL operations, from basic joins and aggregations to advanced window functions and CTEs. This approach is especially useful when working with temporary datasets in an in-memory database for quick analysis and testing.
