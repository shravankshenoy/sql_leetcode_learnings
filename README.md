
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

## Day 5

* Note that select count(user_id) from Users although an expression returns a constant value and does not depend on the group by fields

```
-- Approach 1
select 
    contest_id, 
    round((count(*) * 100) / (select count(*) from Users), 2) as percentage
from
Register 
left join Users
using (user_id)
group by contest_id
order by percentage desc, contest_id

-- Approach 2 (better approach)
-- left join is not required in this scenario
-- also we need to do distinct count
select 
    contest_id, 
    round((count(distinct user_id) * 100) / (select count(user_id) from Users), 2) as percentage
from
    Register 
group by 
    contest_id
order by 
    percentage desc, 
    contest_id asc
```

### References
1. https://stackoverflow.com/questions/2051162/sql-multiple-column-ordering


## Day 6

* To format date to a specific format, use date_format function


```
-- https://leetcode.com/problems/monthly-transactions-i/?envType=study-plan-v2&envId=top-sql-50
-- Approach 1
select
  left(trans_date, 7) as month,
  country,
  count(id) as trans_count,
  sum(case when state="approved" then 1 else 0 end) as approved_count,
  sum(amount) as trans_total_amount,
  sum(case when state="approved" then amount else 0 end) as approved_total_amount
from Transactions
group by month, country


-- Approach 2
select
  date_format(trans_date, '%Y-%m') as month,
  country,
  count(id) as trans_count,
  sum(case when state="approved" then 1 else 0 end) as approved_count,
  sum(amount) as trans_total_amount,
  sum(case when state="approved" then amount else 0 end) as approved_total_amount
from Transactions
group by month, country

```

## Day 7

* Cannot use next_login in the where clause of corresponding to the select, hence had to put it as a subquery

* Need to alias the sUbquery to select from it, else an error

```
# 12/15 cases passed
select round(count(*) / (select count(distinct player_id) from Activity),2) as fraction 
from 
    (select * from 
        (select 
            player_id, 
            event_date, 
            lead(event_date) over (order by player_id, event_date) as next_login
         from Activity) a1
    where event_date = (select min(event_date) from Activity a2 where a1.player_id=a2.player_id) and datediff(event_date, next_login) = -1
    ) x

# 14/15 cases passed
select round(count(distinct player_id)/(select count(distinct player_id) from Activity),2) as fraction 
from 
    (select * from 
        (select player_id, 
        event_date, 
        lead(event_date) over (order by player_id, event_date) as next_login
        from Activity) a1
    where event_date = (select min(event_date) from Activity a2 where a1.player_id=a2.player_id) and datediff(event_date, next_login) = -1
    )x

```

## Day 8

*  To compare two dates we can use arithmetic operators like >, >=, <, <=, == and <>. For these operators to work, the two dates should be in the ANSI-standard YYYY-MM-DD format. Other keywords for comparison are BETWEEN, ISNULL, DATEDIFF


* Be careful about the edge cases i.e. >= and >, in this case since 27th July 2019 is included, it should be > with -30 days and >= with -29 days

```
-- https://leetcode.com/problems/user-activity-for-the-past-30-days-i/?envType=study-plan-v2&envId=top-sql-50

-- Approach 1 (using date_add)

select 
    activity_date as day,
    count(distinct user_id) as active_users
from Activity
where activity_date <= '2019-07-27' and activity_date > date_add('2019-07-27', INTERVAL -30 DAY)
group by activity_date

-- Approach 2 (using datediff)
-- TO DO : Add sql code using datediff approach

```

* Dateadd syntax is different in mysql compared to other databases


### References
1. https://www.dbvis.com/thetable/how-to-compare-sql-dates/

## Day 9

*  We can use WHERE...IN with multiple columns simulataneously i.e. 

```

SELECT * FROM tableA
WHERE
   ([C1], [C2],..., [Cn]) IN (SELECT [D1], [D2],..., [Dn] FROM tableB)

```
In some databases like SQL Server this is not supported. In that case, you can use EXISTS, as mentioned in reference 1

* Correlated subqueries (which is used in approach 1) are very slow. 

```

-- https://leetcode.com/problems/product-sales-analysis-iii/?envType=study-plan-v2&envId=top-sql-50
-- Approach 1 (was timing out for some test cases)
select product_id, year as first_year, quantity, price
from Sales s1
where year = (
    select min(year) from Sales s2 where s1.product_id = s2.product_id
)

-- Apporach 2
select product_id, year as first_year, quantity, price
from Sales 
where (product_id, year) in (
    select product_id, min(year) from Sales group by product_id
)


```

### Doubts
1. How do you define a correlated subquery? What are some situations in which only correlated subquery works?


### References
1. https://dba.stackexchange.com/questions/247382/where-in-based-on-multiple-columns


## Day 10

* WHERE vs GROUP BY : Use Having clause when you are to filter out on the table which is already grouped by. Use Where clause when you are to filter out on the table before group by.

```

-- https://leetcode.com/problems/number-of-unique-subjects-taught-by-each-teacher/?envType=study-plan-v2&envId=top-sql-50
select 
    teacher_id, 
    count(distinct subject_id) as cnt
from Teacher
group by teacher_id

-- https://leetcode.com/problems/classes-more-than-5-students/?envType=study-plan-v2&envId=top-sql-50

select class 
from Courses
group by class
having count(student) >= 5


-- https://leetcode.com/problems/find-followers-count/?envType=study-plan-v2&envId=top-sql-50


select user_id, count(follower_id) as followers_count
from Followers
group by user_id
order by user_id asc


-- https://leetcode.com/problems/biggest-single-number/submissions/1482926065/?envType=study-plan-v2&envId=top-sql-50

--- Approach 1 (using CTE)
with SingleNumbers as (
    select num
    from MyNumbers
    group by num
    having count(*) = 1
)
select max(num) as num from SingleNumbers


--- Approach 2 (using subqueries)
TO DO : Add subqueries approach

-- https://leetcode.com/problems/customers-who-bought-all-products/submissions/1482940486/?envType=study-plan-v2&envId=top-sql-50
-- can we solve this problem without using count
select customer_id 
from Customer
group by customer_id
having count(distinct product_key) = (select count(product_key) from Product)

```

## Day 11

```
-- https://leetcode.com/problems/the-number-of-employees-which-report-to-each-employee/?envType=study-plan-v2&envId=top-sql-50
--- Approach 1 (using cte)
with Managers as 
(
select 
    reports_to as employee_id, 
    count(distinct employee_id) as reports_count, 
    round(avg(age), 0) as average_age
from Employees
where reports_to is not null
group by reports_to
)
select m.employee_id, e.name, m.reports_count, m.average_age
from Managers m
left join Employees e
on m.employee_id = e.employee_id

--- Approach 2 (using self join)
--- TO DO : Add self join code

-- https://leetcode.com/problems/primary-department-for-each-employee/?envType=study-plan-v2&envId=top-sql-50

--- Approach 1 (using union)

select employee_id, min(department_id) as department_id
from Employee 
group by employee_id
having count(*) = 1

union

select employee_id, department_id
from Employee
where primary_flag='Y'

--- Apporach 2 (without using union, just where clause)
--- TO DO : Add union code


```

## Day 12

```
-- https://leetcode.com/problems/triangle-judgement/?envType=study-plan-v2&envId=top-sql-50
select 
    x,
    y,
    z,
    case when x + y > z and y + z > x and x+z > y then "Yes" else "No" end as triangle
from Triangle

-- https://leetcode.com/problems/consecutive-numbers/description/?envType=study-plan-v2&envId=top-sql-50
select distinct num as ConsecutiveNums 
from 
(
    select 
        id, 
        num, 
        lag(num) over (order by id) as num1, lag(num, 2) over (order by id) as num2
    from 
        Logs
) LogsAdjacent
where num = num1 and num1 = num2

-- Write the same code using CTE, look for other logics


```

* When we create a subquery we need to alias it, else we get an error

* There are different types of window functions listed below
    * Aggregate window functions: Perform operations on sets of rows within a window, such as using SUM(), MAX(), and COUNT().
    * Ranking window functions : Rank the rows within a given window, such as using RANK(), DENSE_RANK(), or ROW_NUMBER()
    * Value window functions : Combine multiple operations into one, such as using LAG(), LEAD(), or FIRST_VALUE().

### Doubts
1. What are the different types of window functions? 


## Day 13
* Have filtered multiple columns simulataneously using WHERE....IN. Had to consider multiple scenarios
    * No change_date before 2019-08-16
    * Having one or more change_date only before 2019-08-16
    * Having change_date both before and after 2019-08-16
    * Having a change_date on 2019-08-16
    Since we are not concerned with change_date post 2019-08-16, 2,3,4 can be handled simulataneously

```
-- https://leetcode.com/problems/product-price-at-a-given-date/?envType=study-plan-v2&envId=top-sql-50
with PricePrior as (
    -- for products which have atleast once change_date before or on 2019-08-16
    select product_id, new_price, change_date
    from Products
    where change_date <= "2019-08-16"
    
    union all
    -- for products where there is no change_date before 2019-08-16
    select product_id, 10 as new_price, "1900-01-01" as change_date
    from Products
)

select distinct
    product_id, new_price as price
from PricePrior
where (product_id, change_date) in 
    (select product_id, max(change_date) from PricePrior group by product_id)

```

* UNION ALL will perform better than UNION when you're not concerned about eliminating duplicate records because you're avoiding an expensive distinct sort operation

### Doubts
1. Suppose we have to find the value at the end of each month of the year, and we have new records in the table only when there is a change, how do we write a sql for that?

### References
1. https://stackoverflow.com/questions/3627946/performance-of-union-versus-union-all-in-sql-server


## Day 14

* Used the same idea that I had learnt from (Day 7 problem)[https://leetcode.com/problems/game-play-analysis-iv/solutions/3673167/best-optimum-solution-with-explanation/?envType=study-plan-v2&envId=top-sql-50]. In that problem, to join a row with the next day row, we joined with the on condition having datediff = 1. In this problem, for every turn, I wanted to join with all the previous turns, and then get sum of all those rows to get the cumulative sum hence the join condition

* When there is a missing catgory, then we have to handle each category separately with union

```
-- https://leetcode.com/problems/last-person-to-fit-in-the-bus/?envType=study-plan-v2&envId=top-sql-50
with CumulativeQueue as (
    select q1.person_id, q1.person_name, q1.weight, q1.turn, sum(q2.weight) as net_weight,
    case when sum(q2.weight) <=1000 then 0 else 1 end as exceeds 
    from Queue q1
    left join Queue q2
    on q1.turn >= q2.turn
    group by 1,2,3,4
    order by q1.turn asc
)
select person_name
from CumulativeQueue
where turn = (select max(turn) from CumulativeQueue where exceeds = 0)

--- Optimize above code

--- Solve the problem using window function

--- Solve the problem using SQL variables


-- https://leetcode.com/problems/count-salary-categories/submissions/1486960437/?envType=study-plan-v2&envId=top-sql-50
--- Wrong approach : Does not work because if there is a category with no entries it does not appear in final result
with AccountsWithCategory as
(
    select
        account_id,
        case when income < 20000 then "Low Salary"
            when income >= 20000 and income <= 50000 then "Average Salary"
            when income > 50000 then "High Salary" 
        end as category
    from
    Accounts
   
)

select category, count(account_id) as accounts_count from AccountsWithCategory group by category

--- Approach 1
select "Low Salary" as category, count(account_id) as accounts_count from Accounts where income < 20000

union

select "Average Salary" as category, count(account_id) as accounts_count from Accounts where income >= 20000 and income <= 50000

union
select "High Salary" as category, count(account_id) as accounts_count from Accounts where income > 50000

```

### Doubts
1. How to compute cumulative sum in Mysql? How about other databases?
2. What are variables in mysql and how do we use them?

### References
1. https://stackoverflow.com/a/61904024
2. https://stackoverflow.com/questions/2563918/create-a-cumulative-sum-column-in-mysql
