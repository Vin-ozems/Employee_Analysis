# Employee_Analysis

## Project Overview
This project is a SQL exercise set built on the classic open-source "employees" sample database, demonstrating a range of SQL techniques — from joins and subqueries to stored procedures, triggers, and custom functions — applied to real HR-style data (employees, departments, titles, salaries).

## Dataset Description
The standard **employees** sample database, consisting of related tables:
- `employees`: employee number, name, gender, birth date, hire date
- `departments`: department number and name
- `dept_emp`: mapping of employees to departments over time (with from/to dates)
- `titles`: job titles held by employees
- `salaries`: salary history per employee (with from/to dates)

## What This Project Demonstrates

**Querying & Joins**
- Average salary by department and gender, using multi-table joins across salaries, employees, dept_emp, and departments
- Lowest and highest department numbers in use
- Employees hired in a specific year
- Filtering employees by job title (e.g. all "Engineer" and "Senior Engineer" title holders)
- Counting high-value, long-duration salary contracts (≥ $100,000, lasting more than a year)

**Subqueries & Conditional Logic**
- Retrieving each employee's lowest department number via a correlated subquery
- Assigning manager IDs conditionally based on employee number ranges using CASE logic

**Stored Procedures**
- `la_dept(p_emp)`: given an employee number, returns their most recent department (number and name)

**Triggers**
- A `BEFORE INSERT` trigger on the `employees` table that automatically corrects any hire date set in the future, replacing it with the current date — preventing invalid future-dated hire records from being inserted

**Custom Functions**
- `f_highest_salary(p_emp_no)`: returns an employee's highest recorded salary
- `f_lowest_salary(p_emp_no)`: returns an employee's lowest recorded salary
- `f_salary(p_emp_no, p_min_or_max)`: a more flexible version that returns the min, max, or (if neither 'min' nor 'max' is passed) the salary range (max − min) for a given employee, based on a second input parameter

## Recommendation / Tool
The custom functions and stored procedure in this script are reusable building blocks that could plug into a larger HR reporting system:
- `f_salary()` in particular is a flexible, parameterized way to pull salary insights per employee without writing a new query each time
- `la_dept()` gives instant lookup of an employee's current department, useful for HR lookups or org-chart tools
- The hire-date trigger is a practical example of enforcing data integrity automatically at the database level, preventing bad data (future hire dates) from ever being stored

## Tools & Technology
- MySQL
- Stored procedures, triggers, and deterministic functions
- Correlated subqueries and CASE-based conditional logic

## Notes
A few issues in the script would need fixing before it runs cleanly:
- **`la_dept` procedure (line 92)**: the join condition `e.emp_no = d.dept_no` compares an employee number to a department number, which looks like a typo — it likely should be `de.dept_no = d.dept_no` given the surrounding joins.
- **`la_dept` procedure (line 95)**: references `p_emp_no`, but the procedure parameter is actually named `p_emp` — this mismatch would cause an error and should be corrected to match.
- **Line 99 (`call e.la_dept(10010);`)**: procedures aren't called with a table alias prefix like `e.` — this should simply be `CALL la_dept(10010);`.
- **Line 205–207**: functions are called as `employees.f_salary(...)`, using the database name as a prefix — this only works if explicitly qualifying the schema; if running inside the `employees` database already, `f_salary(...)` alone would suffice.
