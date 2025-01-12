[TOC]

### Leetcode-Database题解

#### 175. 组合两个表

##### 1. 题目

**Person 表：**

```html
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId is the primary key column for this table.
```

Address 表：

```html
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId is the primary key column for this table.
```

编写一个 SQL 查询，满足条件：**无论 person 是否有地址信息**，都需要基于上述两表提供 person 的以下信息：

```
FirstName, LastName, City, State
```

https://leetcode.com/problems/combine-two-tables/description/

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS Person;
CREATE TABLE Person ( PersonId INT, FirstName VARCHAR ( 255 ), LastName VARCHAR ( 255 ) );
DROP TABLE
IF
    EXISTS Address;
CREATE TABLE Address ( AddressId INT, PersonId INT, City VARCHAR ( 255 ), State VARCHAR ( 255 ) );
INSERT INTO Person ( PersonId, LastName, FirstName )
VALUES
    ( 1, 'Wang', 'Allen' );
INSERT INTO Address ( AddressId, PersonId, City, State )
VALUES
    ( 1, 2, 'New York City', 'New York' );
```

##### 3. 题解

使用**左外连接**。根据题意，Person 表中的信息一定会出现，所以使用左外连接。

```sql
SELECT
    FirstName,
    LastName,
    City,
    State
FROM
    Person P
    LEFT JOIN Address A
    ON P.PersonId = A.PersonId;
```

#### 176. [第二高的薪水](https://leetcode-cn.com/problems/second-highest-salary/)

##### 1. 题目

编写一个 SQL 查询，获取 Employee 表中**第二高的薪水**（Salary） 。

```html
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

查找工资第二高的员工。

```html
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

没有找到**返回 null** 而不是不返回数据。

##### 2. 数据

```sql
DROP TABLE
IF EXISTS Employee;
CREATE TABLE Employee ( Id INT, Salary INT );
INSERT INTO Employee ( Id, Salary )
VALUES
    ( 1, 100 ),
    ( 2, 200 ),
    ( 3, 300 );
```

##### 3. 题解

将不同的薪资按降序排序，然后使用 LIMIT 子句获得第二高的薪资。

```mysql
SELECT DISTINCT
    Salary AS SecondHighestSalary
FROM
    Employee
ORDER BY Salary DESC
LIMIT 1 OFFSET 1;
```

然而，如果没有这样的第二最高工资，这个解决方案将被判断为 “错误答案”，因为**本表可能只有一项记录**。为了克服这个问题，可以将其作为临时表。

为了在没有查找到数据时返回 null，需要在**查询结果外面再套一层 SELECT**。

```sql
SELECT
    (SELECT DISTINCT Salary FROM Employee ORDER BY Salary DESC LIMIT 1, 1) SecondHighestSalary;
```

解决 “NULL” 问题的另一种方法是使用 “**IFNULL**” 函数，如下所示。

```mysql
SELECT
    IFNULL(
        (SELECT DISTINCT Salary
         FROM Employee
         ORDER BY Salary DESC
         LIMIT 1 OFFSET 1),
        NULL) AS SecondHighestSalary;
```



#### 177. 第N高的薪水

##### 1. 题目

题目与上一类似， 不过是**查找工资第 N 高**的员工。

##### 2. 题解

创建函数法。

```sql
CREATE FUNCTION getNthHighestSalary (N INT) RETURNS INT BEGIN

SET N = N - 1;
RETURN (
    SELECT ( 
        SELECT 
        DISTINCT Salary 
        FROM Employee 
        ORDER BY 
        Salary DESC 
        LIMIT N, 1 ) );
END
```



#### 178. [分数排名](https://leetcode-cn.com/problems/rank-scores/)

##### 1. 题目

得分表：

```html
+----+-------+
| Id | Score |
+----+-------+
| 1  | 3.50  |
| 2  | 3.65  |
| 3  | 4.00  |
| 4  | 3.85  |
| 5  | 4.00  |
| 6  | 3.65  |
+----+-------+
```

**将得分排序，并统计排名。**编写一个 SQL 查询来实现分数排名。

如果两个**分数相同**，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，**名次之间不应该有“间隔”**。

```html
+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```

##### 2. 数据

```sql
DROP TABLE
IF EXISTS Scores;
CREATE TABLE Scores ( Id INT, Score DECIMAL ( 3, 2 ) );
INSERT INTO Scores ( Id, Score )
VALUES
    ( 1, 3.5 ),
    ( 2, 3.65 ),
    ( 3, 4.0 ),
    ( 4, 3.85 ),
    ( 5, 4.0 ),
    ( 6, 3.65 );
```

##### 3. 题解

```sql
SELECT
    S1.score,
    COUNT(DISTINCT S2.score) Rank
FROM
    Scores S1
    INNER JOIN Scores S2
    ON S1.score <= S2.score
GROUP BY
    S1.id, S1.score
ORDER BY
    S1.score DESC;
```

```mysql
SELECT a.Score AS Score,
(SELECT COUNT(DISTINCT b.Score) FROM Scores b WHERE b.Score >= a.Score) AS Rank
FROM Scores a
ORDER BY a.Score DESC;
```



#### 180. [连续出现的数字](https://leetcode-cn.com/problems/consecutive-numbers/)

##### 1. 题目

数字表：

```html
+----+-----+
| Id | Num |
+----+-----+
| 1  |  1  |
| 2  |  1  |
| 3  |  1  |
| 4  |  2  |
| 5  |  1  |
| 6  |  2  |
| 7  |  2  |
+----+-----+
```

查找**连续出现三次**的数字。

```html
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS LOGS;
CREATE TABLE LOGS ( Id INT, Num INT );
INSERT INTO LOGS ( Id, Num )
VALUES
    ( 1, 1 ),
    ( 2, 1 ),
    ( 3, 1 ),
    ( 4, 2 ),
    ( 5, 1 ),
    ( 6, 2 ),
    ( 7, 2 );
```

##### 3. 题解

```sql
SELECT
    DISTINCT L1.num ConsecutiveNums
FROM
    Logs L1,
    Logs L2,
    Logs L3
WHERE L1.id = l2.id - 1
    AND L2.id = L3.id - 1
    AND L1.num = L2.num
    AND l2.num = l3.num;
```



#### 181. 超过经理收入的员工

##### 1. 题目

Employee 表：

```html
+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+
```

查找**薪资大于其经理薪资的员工信息**。

https://leetcode.com/problems/employees-earning-more-than-their-managers/description/

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS Employee;
CREATE TABLE Employee ( Id INT, NAME VARCHAR ( 255 ), Salary INT, ManagerId INT );
INSERT INTO Employee ( Id, NAME, Salary, ManagerId )
VALUES
    ( 1, 'Joe', 70000, 3 ),
    ( 2, 'Henry', 80000, 4 ),
    ( 3, 'Sam', 60000, NULL ),
    ( 4, 'Max', 90000, NULL );
```

##### 3. 题解

**自连接**：

```sql
SELECT
    a.Name AS 'Employee'
FROM
    Employee AS a,
    Employee AS b
WHERE
    a.ManagerId = b.Id AND a.Salary > b.Salary;
```

```mysql
SELECT
     a.NAME AS Employee
FROM Employee AS a JOIN Employee AS b
     ON a.ManagerId = b.Id AND a.Salary > b.Salary;
```



#### 182. 查找重复的电子邮箱

##### 1. 题目

编写一个 SQL 查询，查找 **Person** 表中**所有重复的电子邮箱**。邮件地址表：

```html
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
```

查找重复的邮件地址：

```html
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

https://leetcode.com/problems/duplicate-emails/description/

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS Person;
CREATE TABLE Person ( Id INT, Email VARCHAR ( 255 ) );
INSERT INTO Person ( Id, Email )
VALUES
    ( 1, 'a@b.com' ),
    ( 2, 'c@d.com' ),
    ( 3, 'a@b.com' );
```

##### 3. 题解

```sql
SELECT
    Email
FROM
    Person
GROUP BY
    Email
HAVING
    COUNT(*) >= 2;
```



#### 183. 从不订购的客户

##### 1. 题目

某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，**找出所有从不订购任何东西的客户**。

Customers 表：

```html
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+
```

Orders 表：

```html
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
```

查找没有订单的顾客信息：

```html
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

https://leetcode.com/problems/customers-who-never-order/description/

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS Customers;
CREATE TABLE Customers ( Id INT, NAME VARCHAR ( 255 ) );
DROP TABLE
IF
    EXISTS Orders;
CREATE TABLE Orders ( Id INT, CustomerId INT );
INSERT INTO Customers ( Id, NAME )
VALUES
    ( 1, 'Joe' ),
    ( 2, 'Henry' ),
    ( 3, 'Sam' ),
    ( 4, 'Max' );
INSERT INTO Orders ( Id, CustomerId )
VALUES
    ( 1, 3 ),
    ( 2, 1 );
```

##### 3. 题解

**左外链接**：

```sql
SELECT
    C.Name AS Customers
FROM
    Customers C
    LEFT JOIN Orders O
    ON C.Id = O.CustomerId
WHERE
    O.CustomerId IS NULL;
```

**子查询**：

```sql
SELECT
    Name AS Customers
FROM
    Customers
WHERE
    Id NOT IN (SELECT CustomerId FROM Orders);
```



#### 184. [部门工资最高的员工](https://leetcode-cn.com/problems/department-highest-salary/)

##### 1. 题目

Employee 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。

```html
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Henry | 80000  | 2            |
| 3  | Sam   | 60000  | 2            |
| 4  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
```

Department 表包含公司所有部门的信息。

```html
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```

编写一个 SQL 查询，**找出每个部门工资最高的员工**。例如，根据上述给定的表格，Max 在 IT 部门有最高工资，Henry 在 Sales 部门有最高工资。

```html
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
```

##### 2. 数据

```sql
DROP TABLE IF EXISTS Employee;
CREATE TABLE Employee ( Id INT, NAME VARCHAR ( 255 ), Salary INT, DepartmentId INT );
DROP TABLE IF EXISTS Department;
CREATE TABLE Department ( Id INT, NAME VARCHAR ( 255 ) );
INSERT INTO Employee ( Id, NAME, Salary, DepartmentId )
VALUES
    ( 1, 'Joe', 70000, 1 ),
    ( 2, 'Henry', 80000, 2 ),
    ( 3, 'Sam', 60000, 2 ),
    ( 4, 'Max', 90000, 1 );
INSERT INTO Department ( Id, NAME )
VALUES
    ( 1, 'IT' ),
    ( 2, 'Sales' );
```

##### 3. 题解

创建一个临时表，包含了部门员工的最大薪资。可以对部门进行分组，然后使用 MAX() 汇总函数取得最大薪资。

之后使用连接找到一个部门中薪资等于临时表中最大薪资的员工。

```sql
SELECT
    D.NAME Department,
    E.NAME Employee,
    E.Salary
FROM
    Employee E,
    Department D,
    (SELECT DepartmentId, MAX(Salary) Salary FROM Employee GROUP BY DepartmentId) M
WHERE
    E.DepartmentId = D.Id
    AND E.DepartmentId = M.DepartmentId
    AND E.Salary = M.Salary;
```

```mysql
SELECT
    Department.name AS 'Department',
    Employee.name AS 'Employee',
    Salary
FROM
    Employee
        JOIN
    Department ON Employee.DepartmentId = Department.Id
WHERE
    (Employee.DepartmentId , Salary) IN (
        SELECT
        DepartmentId, MAX(Salary)
        FROM
        Employee
        GROUP BY DepartmentId
    );
```

```mysql
SELECT
	Department.NAME AS Department,
	Employee.NAME AS Employee,
	Salary 
FROM
	Employee,
	Department 
WHERE
	Employee.DepartmentId = Department.Id 
	AND ( Employee.DepartmentId, Salary ) 
    IN (SELECT DepartmentId, max( Salary ) 
        FROM Employee 
        GROUP BY DepartmentId );
```



#### 196. 删除重复的电子邮箱

##### 1. 题目

编写一个 SQL 查询，来删除 **Person** 表中所有重复的电子邮箱，重复的邮箱里**只保留 Id 最小 的那个**。

邮件地址表：

```html
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
```

删除重复的邮件地址：

```html
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
```

https://leetcode.com/problems/delete-duplicate-emails/description/

##### 2. 数据

与 182 相同。

##### 3. 题解

**自连接**：

```sql
DELETE p1
FROM
    Person AS p1,
    Person AS p2
WHERE
    p1.Email = p2.Email
    AND p1.Id > p2.Id; 
```

**子查询**：

```sql
DELETE
FROM
    Person
WHERE
    id NOT IN (SELECT id FROM (SELECT min(id) AS id FROM Person GROUP BY email) AS m);
```

应该注意的是上述解法**额外嵌套了一个 SELECT 语句**，如果不这么做，会出现错误：You can't specify target table 'Person' for update in FROM clause。以下演示了这种错误解法。

```sql
DELETE
FROM
    Person
WHERE
    id NOT IN (SELECT min(id) AS id FROM Person GROUP BY email);
```



#### 197. [上升的温度](https://leetcode-cn.com/problems/rising-temperature/)

##### 1. 题目

给定一个 Weather 表，编写一个 SQL 查询，**来查找与之前（昨天的）日期相比温度更高的所有日期的 Id**。

```mysql
+---------+------------------+------------------+
| Id(INT) | RecordDate(DATE) | Temperature(INT) |
+---------+------------------+------------------+
|       1 |       2015-01-01 |               10 |
|       2 |       2015-01-02 |               25 |
|       3 |       2015-01-03 |               20 |
|       4 |       2015-01-04 |               30 |
+---------+------------------+------------------+
```

例如，根据上述给定的 Weather 表格，返回如下 Id:

```mysql
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```

##### 2. 题解

使用 **JOIN 和 DATEDIFF() 子句**。MySQL 使用 DATEDIFF 来比较两个日期类型的值。因此可以通过将 weather 与自身相结合，并使用 DATEDIFF() 函数。

```mysql
SELECT
    weather.id AS 'Id'
FROM
    weather
        JOIN
    weather w ON DATEDIFF(weather.date, w.date) = 1
        AND weather.Temperature > w.Temperature;
```

也可以用**外连接+子查询**。

```mysql
SELECT 
    w.Id
FROM Weather w
JOIN (
    SELECT 
        RecordDate,Temperature
    FROM 
        Weather
) t1
ON w.RecordDate = DATE_ADD(t1.RecordDate,INTERVAL 1 DAY)
WHERE w.Temperature > t1.Temperature;
```



#### 595. 大的国家

##### 1. 题目

```html
+-----------------+------------+------------+--------------+---------------+
| name            | continent  | area       | population   | gdp           |
+-----------------+------------+------------+--------------+---------------+
| Afghanistan     | Asia       | 652230     | 25500100     | 20343000      |
| Albania         | Europe     | 28748      | 2831741      | 12960000      |
| Algeria         | Africa     | 2381741    | 37100000     | 188681000     |
| Andorra         | Europe     | 468        | 78115        | 3712000       |
| Angola          | Africa     | 1246700    | 20609294     | 100990000     |
+-----------------+------------+------------+--------------+---------------+
```

查找面积超过 3,000,000 或者人口数超过 25,000,000 的国家。

```html
+--------------+-------------+--------------+
| name         | population  | area         |
+--------------+-------------+--------------+
| Afghanistan  | 25500100    | 652230       |
| Algeria      | 37100000    | 2381741      |
+--------------+-------------+--------------+
```

连接：https://leetcode.com/problems/big-countries/description/

#####  2. 数据

```sql
DROP TABLE
IF
    EXISTS World;
CREATE TABLE World ( NAME VARCHAR ( 255 ), continent VARCHAR ( 255 ), area INT, population INT, gdp INT );
INSERT INTO World ( NAME, continent, area, population, gdp )
VALUES
    ( 'Afghanistan', 'Asia', '652230', '25500100', '203430000' ),
    ( 'Albania', 'Europe', '28748', '2831741', '129600000' ),
    ( 'Algeria', 'Africa', '2381741', '37100000', '1886810000' ),
    ( 'Andorra', 'Europe', '468', '78115', '37120000' ),
    ( 'Angola', 'Africa', '1246700', '20609294', '1009900000' );
```

##### 3. 题解

```sql
SELECT name,
    population,
    area
FROM
    World
WHERE
    area > 3000000 OR population > 25000000;
```



#### 596. 超过5名学生的课

##### 1. 题目

```html
+---------+------------+
| student | class      |
+---------+------------+
| A       | Math       |
| B       | English    |
| C       | Math       |
| D       | Biology    |
| E       | Math       |
| F       | Computer   |
| G       | Math       |
| H       | Math       |
| I       | Math       |
+---------+------------+
```

请列出所有超过或等于 5 名学生的课。

```html
+---------+
| class   |
+---------+
| Math    |
+---------+
```

https://leetcode.com/problems/classes-more-than-5-students/description/

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS courses;
CREATE TABLE courses ( student VARCHAR ( 255 ), class VARCHAR ( 255 ) );
INSERT INTO courses ( student, class )
VALUES
    ( 'A', 'Math' ),
    ( 'B', 'English' ),
    ( 'C', 'Math' ),
    ( 'D', 'Biology' ),
    ( 'E', 'Math' ),
    ( 'F', 'Computer' ),
    ( 'G', 'Math' ),
    ( 'H', 'Math' ),
    ( 'I', 'Math' );
```

##### 3. 题解

```sql
SELECT
    class
FROM
    courses
GROUP BY
    class
HAVING
    count( DISTINCT student ) >= 5;
```



#### 620. 有趣的电影

##### 1. 题目


```html
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   1     | War       |   great 3D   |   8.9     |
|   2     | Science   |   fiction    |   8.5     |
|   3     | irish     |   boring     |   6.2     |
|   4     | Ice song  |   Fantacy    |   8.6     |
|   5     | House card|   Interesting|   9.1     |
+---------+-----------+--------------+-----------+
```

查**找 id 为奇数**，并且 **description 不是 boring 的电影**，**按 rating 降序**。

```html
+---------+-----------+--------------+-----------+
|   id    | movie     |  description |  rating   |
+---------+-----------+--------------+-----------+
|   5     | House card|   Interesting|   9.1     |
|   1     | War       |   great 3D   |   8.9     |
+---------+-----------+--------------+-----------+
```

https://leetcode.com/problems/not-boring-movies/description/

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS cinema;
CREATE TABLE cinema ( id INT, movie VARCHAR ( 255 ), description VARCHAR ( 255 ), rating FLOAT ( 2, 1 ) );
INSERT INTO cinema ( id, movie, description, rating )
VALUES
    ( 1, 'War', 'great 3D', 8.9 ),
    ( 2, 'Science', 'fiction', 8.5 ),
    ( 3, 'irish', 'boring', 6.2 ),
    ( 4, 'Ice song', 'Fantacy', 8.6 ),
    ( 5, 'House card', 'Interesting', 9.1 );
```

##### 3. 题解

这里**取模**有两种方法。

```sql
SELECT
    *
FROM
    cinema
WHERE
    id % 2 = 1 AND description != 'boring'
ORDER BY 
	rating DESC;
```

```mysql
SELECT *
FROM cinema
WHERE MOD(id, 2) = 1 AND description != 'boring'
ORDER BY rating DESC
```











#### 626. [换座位](https://leetcode-cn.com/problems/exchange-seats/)

##### 1. 题目

seat 表存储着**座位对应**的学生。

```html
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Abbot   |
|    2    | Doris   |
|    3    | Emerson |
|    4    | Green   |
|    5    | Jeames  |
+---------+---------+
```

要求**交换相邻座位**的两个学生，如果最后一个座位是奇数，那么不交换这个座位上的学生。

```html
+---------+---------+
|    id   | student |
+---------+---------+
|    1    | Doris   |
|    2    | Abbot   |
|    3    | Green   |
|    4    | Emerson |
|    5    | Jeames  |
+---------+---------+
```

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS seat;
CREATE TABLE seat ( id INT, student VARCHAR ( 255 ) );
INSERT INTO seat ( id, student )
VALUES
    ( '1', 'Abbot' ),
    ( '2', 'Doris' ),
    ( '3', 'Emerson' ),
    ( '4', 'Green' ),
    ( '5', 'Jeames' );
```

##### 3. 题解

使用多个 union。

```sql
SELECT
    s1.id - 1 AS id,
    s1.student
FROM
    seat s1
WHERE
    s1.id MOD 2 = 0 UNION
SELECT
    s2.id + 1 AS id,
    s2.student
FROM
    seat s2
WHERE
    s2.id MOD 2 = 1
    AND s2.id != ( SELECT max( s3.id ) FROM seat s3 ) UNION
SELECT
    s4.id AS id,
    s4.student
FROM
    seat s4
WHERE
    s4.id MOD 2 = 1
    AND s4.id = ( SELECT max( s5.id ) FROM seat s5 )
ORDER BY
    id;
```



#### 627. 交换工资

##### 1. 题目

m 表示男性，f 表示女性。

```html
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | m   | 2500   |
| 2  | B    | f   | 1500   |
| 3  | C    | m   | 5500   |
| 4  | D    | f   | 500    |
```

只用一个 SQL 查询，**将 sex 字段反转**。

```html
| id | name | sex | salary |
|----|------|-----|--------|
| 1  | A    | f   | 2500   |
| 2  | B    | m   | 1500   |
| 3  | C    | f   | 5500   |
| 4  | D    | m   | 500    |
```

https://leetcode.com/problems/swap-salary/description/

##### 2. 数据

```sql
DROP TABLE
IF
    EXISTS salary;
CREATE TABLE salary ( id INT, NAME VARCHAR ( 100 ), sex CHAR ( 1 ), salary INT );
INSERT INTO salary ( id, NAME, sex, salary )
VALUES
    ( '1', 'A', 'm', '2500' ),
    ( '2', 'B', 'f', '1500' ),
    ( '3', 'C', 'm', '5500' ),
    ( '4', 'D', 'f', '500' );
```

##### 3. 题解

```sql
UPDATE salary
SET sex = CHAR ( ASCII(sex) ^ ASCII( 'm' ) ^ ASCII( 'f' ) );
```

```mysql
UPDATE salary
SET
    sex = CASE sex
        WHEN 'm' THEN 'f'
        ELSE 'm'
    END;
```

```mysql
UPDATE salary SET sex = CHAR(ASCII('m') + ASCII('f') - ASCII(sex));
```



