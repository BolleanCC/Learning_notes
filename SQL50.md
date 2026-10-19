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

