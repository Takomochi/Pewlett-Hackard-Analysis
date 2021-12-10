# Pewlett Hackard Analysis


## Overview of the project
Pewlett Hackard is a large company with thousands of employees.
They are preparing for people who are retiring the company soon.
We were given following tasks for that.

1. Determine the number of retiring employees per title.
2. Identify employees who are eligible to participate in a mentorship program.

Using SQL, we will retrieve specific information and save the tables in new csv files. The Queries for this project is located [here](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Queries/Employee_Database_challenge.sql).

## Resources
- Data Source: [departments.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/departments.csv), [dept_emp.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/dept_emp.csv), [dept_manager.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/dept_manager.csv),     [employees.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/employees.csv), 
[salaries.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/salaries.csv), 
[titles.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/titles.csv)

- Software: PostgreSQL 14, Visual Studio Code

## Technical Steps
- <b>Create retirement_titles table</b> ([retirement_titles.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/retirement_titles.csv))<br> 
  By joining Employees table and Titles table, retrieved emp_no, first_name, last_name, title, from_date, and to_date columns. Also, filtered the data by specifying employee's birth date. Below is the code. <br>
  ```
  SELECT e.emp_no,
	e.first_name,
	e.last_name,
	tl.title,
	tl.from_date,
	tl.to_date
    INTO retirement_titles
    FROM employees as e
    INNER JOIN titles as tl
        ON (e.emp_no = tl.emp_no)
    WHERE (e.birth_date BETWEEN '1952-01-01' AND '1955-12-31')
    ORDER BY e.emp_no;
  ```
  
  In the outcome, we can see that there are exactly the same names and emp_no in the table.

  <img width="490" alt="retirement_titles" src="https://user-images.githubusercontent.com/85041697/144729534-3fcb230c-ee45-454e-b35f-9bd3e4afa905.png">

- <b> Create unique_titles table </b>([unique_titles.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/unique_titles.csv)) <br>
  There are duplicate entries for some employees because they have switched titles over the years. So we removed duplicates by using ```DISTINCT ON``` statement. Below is the code.
    ```
    SELECT DISTINCT ON (emp_no) emp_no,
    first_name,
    last_name,
    title
    INTO unique_titles
    FROM retirement_titles
    ORDER BY emp_no, to_date DESC;
    ```
  We do not see duplicates in the outcome.

    <img width="400" alt="unique_titles" src="https://user-images.githubusercontent.com/85041697/144729543-8fb699a8-446c-4463-9a36-12f2b76b043b.png">
  

-  <b> Create retiring_titles table</b> ([retiring_titles.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/retiring_titles.csv))<br>
  Using unique_titles table, we created a new table which shows the number of employees by their most recent job title who are about to retire. Here is the code.
    ```
    SELECT COUNT(title), title
    INTO retiring_titles
    FROM unique_titles
    GROUP BY title
    ORDER BY COUNT(title) DESC;
    ```

    This is the outcome that shows employees who were born between 1952-01-01' and '1955-12-31'.<br>

   <img height=200 alt="retiring_titles" src="https://user-images.githubusercontent.com/85041697/144729553-59a530e7-dcef-41a7-bfc4-f4b87cc49d0e.png">


- <b>Create  mentoship_edigibility table</b> ([mentoship_edigibility.csv](https://github.com/Takomochi/Pewlett-Hackard-Analysis/blob/main/Data/mentorship_eligibilty.csv)) <br>
     By joining Employees table, Department Employee table, and Titles table, retrieved emp_no, first_name, last_name, birth_date, from_date, to_date, and title columns. Filtered the data by all the current employees and employees whose birth dates are between January 1, 1965 and December 31, 1965. Here is the code.

    ```
    SELECT DISTINCT ON(e.emp_no) 
	e.emp_no,
	e.first_name,
	e.last_name,
	e.birth_date,
	de.from_date,
	de.to_date,
	tl.title
    INTO mentorship_eligibilty
    FROM employees as e
    INNER JOIN dept_emp as de
        ON (e.emp_no = de.emp_no)
    INNER JOIN titles as tl
        ON (e.emp_no = tl.emp_no)
    WHERE (de.to_date = '9999-01-01')
    AND (e.birth_date BETWEEN '1965-01-01' AND '1965-12-31')
    ORDER BY e.emp_no;	
    ```
    This is the table created by the code.

    <img width="500" alt="mentorship" src="https://user-images.githubusercontent.com/85041697/144729557-614b49f6-9634-456d-8096-43e282131472.png">

## Results
- Total 90398 employees are at retiring age according to unique_titles table.

- Senior Engineer position has the highest number of 29414 employees who are at retiring age. Senior Staff position also has a high number of 28254 employees.

- There will be a lot of "Engineer" positions that need to be filled with the new generation because about 48% of retiring age employees are in Engineer positions.

- There are 1549 employees who are eligible to participate in a mentorship program.

## Summary
- How many roles will need to be filled as the "silver tsunami" begins to make an impact?

  The retirement_titles table is not specific because it still contains the information of people who already left the company. We filtered the table and created a new table so that we can find the number of employees who are retiring soon.

  ```
  # Filter the silver employees who are currently working at the company.
  SELECT  DISTINCT ON (emp_no) * 
  INTO retirement_titles_unique
  FROM retirement_titles
  WHERE to_date = '9999-01-01';

  # Calculate the number of employees for each title.
  SELECT COUNT(title),title
  FROM retirement_titles_unique
  GROUP BY title
  ORDER BY COUNT(title) DESC;
  ```
  This is the outcome. The number of employees who are currently working at the company and retiring soon for each position.<br>

  <img width="200" alt="summary_2" src="https://user-images.githubusercontent.com/85041697/144779917-d720ac9c-64ca-438f-aaf6-000a9dd0036a.png">

- Are there enough qualified, retirement-ready employees in the departments to mentor the next generation of Pewlett Hackard employees?

   We made the table which shows the number of mentor for each title.
  ```
  SELECT COUNT(title), title
  FROM mentorship_eligibilty
  GROUP BY title
  ORDER BY COUNT(title) DESC;
  ```
  <img width="200" alt="summary_1" src="https://user-images.githubusercontent.com/85041697/144777703-347b5ad2-f083-47f3-830f-e5683cc8bae3.png">


  As we can see, the company has a significant number of employees who are retiring soon. There are a small number of mentors. For example, although 25916 senior engineers are retiring soon, only 169 employees are available to mentor the next generation. Also, there is no manager to mentor the next generation. The company should prepare for tackling the problem as soon as possible.