SELECT
    A,
    B,
    COUNT(C)
FROM TableName
GROUP BY A, B;

这是合法的。

因为：

A：普通列 → 需要分组。
B：普通列 → 需要分组。
C：位于 COUNT() 内 → 不需要分组。

你可以简单记成：

SELECT 中没有被聚合函数包裹的普通列，通常都要写进 GROUP BY。

这条规则不仅适用于 COUNT()，也适用于 SUM()、AVG()、MAX()、MIN()。
三、还有一个重要知识点：HAVING

学习聚合函数和 GROUP BY 时，最好顺便掌握 HAVING。

假设题目要求：

找出平均工资超过 6000 的部门
SELECT
    department_id,
    AVG(salary) AS avg_salary
FROM Employees
GROUP BY department_id
HAVING AVG(salary) > 6000;
结果：
| department_id | avg_salary |
| ------------- | ---------- |
| 20            | 7500       |
为什么这里使用 HAVING，而不是 WHERE？

因为：

WHERE：在分组之前过滤原始数据。
HAVING：在分组聚合之后过滤分组结果。

例如：
WHERE salary > 6000
筛选的是工资超过 6000 的员工。

而：
HAVING AVG(salary) > 6000
筛选的是平均工资超过 6000 的部门。

SQL 面试需要记住的核心规则
| SQL 关键字       | 作用       | 执行顺序（逻辑上） |
| ------------- | -------- | --------- |
| `FROM / JOIN` | 获取并连接数据  | 1         |
| `WHERE`       | 过滤原始记录   | 2         |
| `GROUP BY`    | 对数据分组    | 3         |
| `HAVING`      | 过滤分组结果   | 4         |
| `SELECT`      | 计算并选择输出列 | 5         |
| `ORDER BY`    | 排序       | 6         |
| `LIMIT`       | 限制返回行数   | 7         |
最重要的是记住：

WHERE → GROUP BY → HAVING

也就是：

先过滤原始数据 → 再分组聚合 → 最后过滤聚合结果。

1. 什么是 SELF JOIN？

SELF JOIN（自连接）是指一张表与自己进行 JOIN 操作，用来比较同一张表中的不同记录。

普通 JOIN：
SELECT *
FROM Employees e
JOIN Departments d
    ON e.department_id = d.id;

SELF JOIN：
SELECT *
FROM Employees e1
JOIN Employees e2
    ON e1.manager_id = e2.id;

核心区别：

普通 JOIN：连接两张不同的表。
SELF JOIN：同一张表连接自己。

2. SELF JOIN 的基本语法
SELECT a.column1, b.column2
FROM table_name a
JOIN table_name b
    ON a.column = b.column;

3. SELF JOIN 的执行逻辑

假设有一张员工表：

Employees
| id | name    | manager_id |
| -- | ------- | ---------- |
| 1  | Alice   | NULL       |
| 2  | Bob     | 1          |
| 3  | Charlie | 1          |
| 4  | David   | 2          |
其中：

Alice 是 Bob 和 Charlie 的经理。
Bob 是 David 的经理。
Alice 没有经理。

现在要求：
查询每个员工及其经理的名字。

SQL：
SELECT
    e1.name AS employee,
    e2.name AS manager
FROM Employees e1
JOIN Employees e2
    ON e1.manager_id = e2.id;

4. SELF JOIN + LEFT JOIN

如果题目要求：

返回所有员工，包括没有经理的员工。

应该使用：
SELECT
    e1.name AS employee,
    e2.name AS manager
FROM Employees e1
LEFT JOIN Employees e2
    ON e1.manager_id = e2.id;
结果：
| employee | manager |
| -------- | ------- |
| Alice    | NULL    |
| Bob      | Alice   |
| Charlie  | Alice   |
| David    | Bob     |
区别：
| JOIN 类型    | 行为                     |
| ---------- | ---------------------- |
| INNER JOIN | 只返回成功匹配的记录             |
| LEFT JOIN  | 保留左侧表的全部记录，匹配不到则填 NULL |

SELF JOIN 通用答题模板
模板一：比较同一张表中的两条记录
SELECT a.column
FROM table_name a
JOIN table_name b
    ON a.related_column = b.column
WHERE a.value > b.value;
适用：

员工与经理比较。
同类商品比较。
不同日期的数据比较。
模板二：保留所有左侧记录
SELECT a.column, b.column
FROM table_name a
LEFT JOIN table_name b
    ON a.related_column = b.column;
适用：

查询所有员工及其经理。
查询所有分类及其父分类。
查询有或没有关联记录的数据。
模板三：查找不重复的记录组合
SELECT a.id, b.id
FROM table_name a
JOIN table_name b
    ON a.category = b.category
    AND a.id < b.id;
适用：

同类商品两两配对。
同一部门员工两两配对。
同组数据中的不同记录组合。


