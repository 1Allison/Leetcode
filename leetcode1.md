# Database
### 175.组合两个表
#### 题目描述

表1: Person
```
+-------------+---------+
| 列名         | 类型     |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
```
PersonId 是上表主键

表2: Address
```
+-------------+---------+
| 列名         | 类型    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
```
AddressId 是上表主键
 

编写一个 SQL 查询，满足条件：无论 person 是否有地址信息，都需要基于上述两表提供 person 的以下信息：

FirstName, LastName, City, State

#### 答案
```
SELECT Person.FirstName,Person.LastName,Address.City,Address.State FROM Person left join Address ON Person.PersonId=Address.PersonId
```
因为表 Address 中的 personId 是表 Person 的外关键字，所以我们可以连接这两个表来获取一个人的地址信息。

但是由于题目中无论 person 是否有地址信息都要输出结果。因此使用`left join`。

### 176.第二高的薪水
#### 题目描述
编写一个 SQL 查询，获取 Employee 表中第二高的薪水（Salary） 。
```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```

例如上述 Employee 表，SQL查询应该返回 200 作为第二高的薪水。如果不存在第二高的薪水，那么查询应返回 null。
```
+---------------------+
| SecondHighestSalary |
+---------------------+
| 200                 |
+---------------------+
```

#### 答案
```
SELECT (SELECT DISTINCT Salary FROM Employee Order by Salary desc limit 1 offset 1)AS SecondHighestSalary;
```

####  关键点
1. 首先应使用`DISTINCT字`段，实现去重。并且因为若表中只有2个`salary`值为200，返回结果应为`null`。若不使用`DISTINCT`，则返回200。

2. limit y offset x 分句表示: 跳过 x 条数据，读取 y 条数据。

3. 将查询到的结果放在临时表中，在子查询中,如果查询不到会返回`null`。

#### 另解
这种解法可以更方便我们理解下一道题目（选出第n高）
利用ifnull函数，它的参数有两个，如果不是null，返回第一个参数，如果为null返回第二个参数。但这个在leetcode中提交与测试数据不符合，它希望我们返回的表头
```
{"headers": ["SecondHighestSalary"], "values": [[200]]}
```
但输出
```
{"headers": ["IFNULL((SELECT DISTINCT Salary FROM Employee Order by Salary desc limit 1,1),NULL)"], "values": [[200]]}
```
其实这种思路也可以解决这一问题，同时更方便我们对于下一道题的理解。

### 177.第n高的薪水
#### 题目描述
编写一个 SQL 查询，`获取 Employee` 表中第 n 高的薪水（Salary）。

```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+
```
例如上述 Employee 表，`n = 2 `时，应返回第二高的薪水 `200`。如果不存在第 n 高的薪水，那么查询应返回 `null`。
```
+------------------------+
| getNthHighestSalary(2) |
+------------------------+
| 200                    |
+------------------------+
```

#### 答案
```
CREATE FUNCTION getNthHighestSalary(N INT) RETURNS INT
BEGIN
    declare m INT;
    set m=N-1; 
  RETURN (
      # Write your MySQL query statement below.
    SELECT IFNULL((SELECT DISTINCT Salary FROM Employee Order by Salary desc limit m,1),NULL)
  );
END
```
#### 关键点
1. 不能直接用`limit N-1`是因为`limit`和`offset`字段后面只接受正整数（意味着0、负数、小数都不行）或者单一变量（意味着不能用表达式）。因此我们自己定义一个单变量m使`m=N-1`。
2. MySQL自定函数中的参数是静态参数，即要先定义后使用。先用declare定义类型，后通过set进行赋值 。
3. 与上题另解思路类似，利用`IFNULL`函数得到结果。
4. `limit m,1`表示从第m+1行开始，显示一行。

### 178.分数排名
#### 题目
编写一个 SQL 查询来实现分数排名。

如果两个分数相同，则两个分数排名（Rank）相同。请注意，平分后的下一个名次应该是下一个连续的整数值。换句话说，名次之间不应该有“间隔”。
```
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
例如，根据上述给定的 Scores 表，你的查询应该返回（按分数从高到低排列）：
```
+-------+------+
| Score | Rank |
+-------+------+
| 4.00  | 1    |
| 4.00  | 1    |
| 3.85  | 2    |
| 3.65  | 3    |
| 3.65  | 3    |
| 3.50  | 4    |
+-------+------+
```
重要提示：对于 MySQL 解决方案，如果要转义用作列名的保留字，可以在关键字之前和之后使用撇号。例如 `Rank`

难点：如何排名？

##### 思路1：`DENSERANK` 函数
```
select score, DENSE_RANK() OVER (ORDER BY Score DESC) as 'Rank'
from Scores;
```
内置的三个窗口函数：

现在给定五个成绩：99，99，85，80，75。
DENSE_RANK()。如果使用 DENSE_RANK() 进行排名会得到：1，1，2，3，4。

RANK()。如果使用 RANK() 进行排名会得到：1，1，3，4，5。

ROW_NUMBER()。如果使用 ROW_NUMBER() 进行排名会得到：1，2，3，4，5。

##### 思路2 直接思维

```
SELECT Score, (SELECT count(DISTINCT score) FROM Scores WHERE score >= s.score) AS 'Rank' FROM Scores s ORDER BY Score DESC ;
```

后面的都没怎么看懂，先这样。



SELECT
     a.NAME AS Employee    
FROM Employee AS a JOIN Employee AS b
     ON a.ManagerId = b.Id
     AND a.Salary > b.Salary
;
### 180.连续出现的数字
#### 题目
编写一个 SQL 查询，查找所有至少连续出现三次的数字。
```
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
例如，给定上面的 Logs 表， 1 是唯一连续出现至少三次的数字。
```
+-----------------+
| ConsecutiveNums |
+-----------------+
| 1               |
+-----------------+
```

#### 答案
三表连接并且检查连续出现的三个数字是否相同

```
SELECT DISTINCT
    l1.Num AS ConsecutiveNums
FROM
    Logs l1,
    Logs l2,
    Logs l3
WHERE
    l1.Id = l2.Id - 1
    AND l2.Id = l3.Id - 1
    AND l1.Num = l2.Num
    AND l2.Num = l3.Num
;
```
关键点：
row_number() over (partition by col1 order by col2),表示根据col1分组，在分组内部根据col2排序

### 182.查找重复的电子邮箱
#### 题目
编写一个 SQL 查询，查找 Person 表中所有重复的电子邮箱。

示例：
```
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+
```
根据以上输入，你的查询应返回以下结果：
```
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```
说明：所有电子邮箱都是小写字母。

#### 答案
```
select email from person group by email having count(email)>1
```
#### 关键点

1、where 后不能跟聚合函数，因为where执行顺序大于聚合函数。
2、where 子句的作用是在对查询结果进行分组前，将不符合where条件的行去掉，即在分组之前过滤数据，条件中不能包含聚组函数，使用where条件显示特定的行。
3、having 子句的作用是筛选满足条件的组，即在**分组之后**过滤数据，条件中经常包含聚组函数，使用having 条件显示特定的组，也可以使用多个分组标准进行分组。

### 183.从不订购的客户
某网站包含两个表，Customers 表和 Orders 表。编写一个 SQL 查询，找出所有从不订购任何东西的客户。
Customers 表：
```
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
```
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+
```
例如给定上述表格，你的查询应返回：
```
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```
#### 答案
若customers表中的id不在（not in）oders表中的CustomerId中，则从customers选出姓名。
```
select customers.name as 'Customers'
from customers
where customers.id not in
(
    select customerid from orders
);
```

### 184 部门工资最高的员工

#### 题目
Employee 表包含所有员工信息，每个员工有其对应的 Id, salary 和 department Id。
```
+----+-------+--------+--------------+
| Id | Name  | Salary | DepartmentId |
+----+-------+--------+--------------+
| 1  | Joe   | 70000  | 1            |
| 2  | Jim   | 90000  | 1            |
| 3  | Henry | 80000  | 2            |
| 4  | Sam   | 60000  | 2            |
| 5  | Max   | 90000  | 1            |
+----+-------+--------+--------------+
```
Department 表包含公司所有部门的信息。
```
+----+----------+
| Id | Name     |
+----+----------+
| 1  | IT       |
| 2  | Sales    |
+----+----------+
```
编写一个 SQL 查询，找出每个部门工资最高的员工。对于上述表，您的 SQL 查询应返回以下行（行的顺序无关紧要）。
```
+------------+----------+--------+
| Department | Employee | Salary |
+------------+----------+--------+
| IT         | Max      | 90000  |
| IT         | Jim      | 90000  |
| Sales      | Henry    | 80000  |
+------------+----------+--------+
```
解释：

Max 和 Jim 在 IT 部门的工资都是最高的，Henry 在销售部的工资最高。


#### 答案

有可能有多个员工同时拥有最高工资
首先先在部门内查询最高工资，然后，我们可以把表 Employee 和 Department 连接，再在这张临时表里用 IN 语句查询部门名字和工资的关系。
```
SELECT
    Department.name AS 'Department',
    Employee.name AS 'Employee',
    Salary
FROM
    Employee
        JOIN
    Department ON Employee.DepartmentId = Department.Id
WHERE
    (Employee.DepartmentId , Salary) IN
    (   SELECT
            DepartmentId, MAX(Salary)
        FROM
            Employee
        GROUP BY DepartmentId
	)
;
```


mask<<1 mask的值不改变
mask<<=1 等价于mask=mask<<1 由于mask被重新赋值了，所以mask的值改变了，左移了2

### 191 位1的个数
#### 题目
编写一个函数，输入是一个无符号整数（以二进制串的形式），返回其二进制表达式中数字位数为 '1' 的个数（也被称为汉明重量）。

 

提示：

请注意，在某些语言（如 Java）中，没有无符号整数类型。在这种情况下，输入和输出都将被指定为有符号整数类型，并且不应影响您的实现，因为无论整数是有符号的还是无符号的，其内部的二进制表示形式都是相同的。
在 Java 中，编译器使用二进制补码记法来表示有符号整数。因此，在上面的 示例 3 中，输入表示有符号整数 -3。
 

进阶：

如果多次调用这个函数，你将如何优化你的算法？
 

示例：

输入：00000000000000000000000000001011

输出：3

解释：输入的二进制串 00000000000000000000000000001011 中，共有三位为 '1'。


提示：
输入必须是长度为 32 的 二进制串 。

#### 答案
```
class Solution(object):
    def hammingWeight(self, n):
        """
        :type n: int
        :rtype: int
        """
        n = bin(n) 
        #返回一个整数的二进制表示
        count = 0
        for c in n:
            if c == "1":
                count += 1
        return count  
```