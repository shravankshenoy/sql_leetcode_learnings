
# SQL Learnings from Leetcode SQL50

## Day 1
* Relearnt CTE syntax in SQL

* **USING keyword**: If we want to join 2 tables on one or more columns that have the exact same name in both the tables, then we can use the USING keyword instead of ON keyword. Remember that the join column should be within parenthesis when we use USING


```
-- https://leetcode.com/problems/replace-employee-id-with-the-unique-identifier
with emp as (
    select * from Employees
),
emp_uni as (
    select * from EmployeeUNI
)

select 
emp.name,
emp_uni.unique_id
from emp left emp_uni using (id)

```
* One major advantage of the USING keyword is we do not need to qualify the join column with the table alias (infact we should not qualify with alias even if the same column is used elsewhere in the SQL statement, refer 3). While using ON keyword, if we do not qualify the join column with the alias, it gives the error Column name 'id' is ambiguous i.e. it does not know which of the 2 tables to pick the column from


* When we want to filter a column in one table based on common column in another table but not having all values, we can either do a join on that column and then filter nulls (LEFT JOIN + IS NULL) or we can directly use NOT IN, as shown below 

```
-- https://leetcode.com/problems/customer-who-visited-but-did-not-make-any-transactions
-- Goal : To find the IDs of the customers who visited the mall but did not make any transactions and the number of times they made these types of visits (Visits table has customer_id and visit_id, Transactions table has visit_id and transaction_id)
-- Approach 1

with 
Visits as (select * from Visits),
Transactions as (select * from Transactions)

select 
customer_id
,count(*) as count_no_trans
from Visits 
left join Transactions using (visit_id)
where Transactions.transaction_id is null
group by 1

-- Approach 2
SELECT customer_id, COUNT(visit_id) as count_no_trans 
FROM Visits
WHERE visit_id NOT IN (
    SELECT visit_id FROM Transactions
    )
GROUP BY customer_id

```

* We cannot combine USING and ON keyword in the same statement (refer 2). 


### References
1. https://stackoverflow.com/questions/11366006/mysql-join-on-vs-using
2. https://stackoverflow.com/questions/52206667/can-the-sql-join-using-clause-be-used-with-one-or-more-and-on-clauses-fo
3. https://www.geeksforgeeks.org/sql-using-clause/

## Day 2

* Cross join generates all possible combinations of values/rows. Cross join is useful when we want to create a grid like structure with all possible combinations. In the example below, we want to create a grid with all possible combinations of Students and Subjects, and then use that to see how many times a student has taken an exam. Very useful to find cases when student has not taken an exam even once, which would not be captured without a cross join

```

-- Without CTE
select 
    StudentsAllSubjects.student_id, 
    StudentsAllSubjects.student_name, 
    StudentsAllSubjects.subject_name, 
    count(Examinations.student_id) as attended_exams
from
(select * from Students cross join Subjects) as StudentsAllSubjects
left join Examinations 
on 
StudentsAllSubjects.student_id = Examinations.student_id 
and StudentsAllSubjects.subject_name = Examinations.subject_name

group by 1,2,3
order by 1,3

-- With CTE

with StudentsAllSubjects as (
    select * from Students cross join Subjects
)

select 
    a.student_id, 
    a.student_name, 
    a.subject_name, 
    coalesce(count(e.student_id), 0) as attended_exams
from
    StudentsAllSubjects as a

left join Examinations e
on 
a.student_id = e.student_id 
and a.subject_name = e.subject_name

group by 1,2,3
order by 1,3

```

* Using CTE can make code more readable as shown above. Also remember that while using CTE, we use WITH keyword only once in the beginning, not before each table

```
-- wrong syntax
with Visits as (select * from Visits),
with Transactions as (select * from Transactions)

select * from Visits left join Transactions using (id)

```

* Coalesce returns first non null argument in list of arguments
```
SELECT COALESCE(NULL, 1, 2, 'W3Schools.com'); -- Output:1

```

* COALESCE vs IFNULL : ifnull takes 2 arguments, whereas coalesce can take 2 or more than 2 arguments. Ifnull returns the first argument if it is not null, and the second argument if the first argument is null. coalesce return the first non null argument in the list of arguments, and if all the arguments are null, it returns null

### References
1. https://stackoverflow.com/questions/219716/what-are-the-uses-for-cross-join
2. https://stackoverflow.com/questions/8974328/mysql-multiple-joins-in-one-query

## Day 3

* Self join : You use a self join when a table references data in itself (for example managerId references to employeeId in the same table). In other words, foreign key refers to a primary key in the same table


```
-- https://leetcode.com/problems/managers-with-at-least-5-direct-reports/?envType=study-plan-v2&envId=top-sql-50
with Managers as (
    select managerId from Employee
    group by managerId
    having count(*) >= 5
)
select e.name 
from Employee e 
inner join Managers m 
on e.id = m.managerId

```

* Conditional aggregation in SQL is very useful when we want percentage of each category in column (say colA - action) while grouping by another column (say colB - user_id) and not the category column itself

```

-- https://leetcode.com/problems/confirmation-rate/description/?envType=study-plan-v2&envId=top-sql-50

select user_id, 
    round(coalesce(sum(case when action = "confirmed" then 1 else 0 end), 0) * 1.0 / count(*), 2)as confirmation_rate
from
Signups left join Confirmations 
using (user_id)
group by user_id

```
### References
1. https://stackoverflow.com/questions/770579/how-to-calculate-percentage-with-a-sql-statement
2. https://stackoverflow.com/questions/51489103/how-do-i-get-percentage-amount-of-categorical-variables-per-day-using-sql
3. https://stackoverflow.com/questions/3362038/what-is-self-join-and-when-would-you-use-it
4. https://stackoverflow.com/questions/18680680/can-a-foreign-key-refer-to-a-primary-key-in-the-same-table

## Day 4

* Use the mod function or % to find modulus of number by another number
* Use round function to round of result to a given number of decimal places

```
-- https://leetcode.com/problems/not-boring-movies/?envType=study-plan-v2&envId=top-sql-50

select * 
from Cinema
where id % 2 = 1 and lower(description) <> "boring"
order by rating desc


-- https://leetcode.com/problems/project-employees-i/?envType=study-plan-v2&envId=top-sql-50
select 
    project_id, 
    round(avg(experience_years), 2) as average_years
from 
    Project 
left join 
    Employee 
using (employee_id)
group by 1

```
