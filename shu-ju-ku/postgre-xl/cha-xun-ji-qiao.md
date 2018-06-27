# 查询技巧

## 聚合

### 窗口函数

[https://www.postgres-xl.org/documentation/tutorial-window.html](https://www.postgres-xl.org/documentation/tutorial-window.html)

聚合函数可以通过`OVER (PARTITION BY x)`对数据分组，如下例子：计算每个部门的平均工资

```text

SELECT depname, empno, salary, avg(salary) OVER (PARTITION BY depname) FROM empsalary;  depname  | empno | salary |          avg          -----------+-------+--------+----------------------- develop   |    11 |   5200 | 5020.0000000000000000 develop   |     7 |   4200 | 5020.0000000000000000 develop   |     9 |   4500 | 5020.0000000000000000 develop   |     8 |   6000 | 5020.0000000000000000 develop   |    10 |   5200 | 5020.0000000000000000 personnel |     5 |   3500 | 3700.0000000000000000 personnel |     2 |   3900 | 3700.0000000000000000 sales     |     3 |   4800 | 4866.6666666666666667 sales     |     1 |   5000 | 4866.6666666666666667 sales     |     4 |   4800 | 4866.6666666666666667(10 rows)
```

还可以通过`OVER (PARTITION BY x ORDER BY y)`对每个分组进行排序，如下例子：对每个部门的工资进行排序

```text
SELECT depname, empno, salary,       rank() OVER (PARTITION BY depname ORDER BY salary DESC)FROM empsalary;  depname  | empno | salary | rank -----------+-------+--------+------ develop   |     8 |   6000 |    1 develop   |    10 |   5200 |    2 develop   |    11 |   5200 |    2 develop   |     9 |   4500 |    4 develop   |     7 |   4200 |    5 personnel |     2 |   3900 |    1 personnel |     5 |   3500 |    2 sales     |     1 |   5000 |    1 sales     |     4 |   4800 |    2 sales     |     3 |   4800 |    2(10 rows)
```

`OVER` 中不带`ORDER BY`时，`SUM`函数会汇总所有记录

```text

SELECT salary, sum(salary) OVER () FROM empsalary; salary |  sum  --------+-------   5200 | 47100   5000 | 47100   3500 | 47100   4800 | 47100   3900 | 47100   4200 | 47100   4500 | 47100   4800 | 47100   6000 | 47100   5200 | 47100(10 rows)
```

如果`OVER`中带有`ORDER BY`时，`SUM`函数会从上到下逐条汇总

```text

SELECT salary, sum(salary) OVER (ORDER BY salary) FROM empsalary; salary |  sum  --------+-------   3500 |  3500   3900 |  7400   4200 | 11600   4500 | 16100   4800 | 25700   4800 | 25700   5000 | 30700   5200 | 41100   5200 | 41100   6000 | 47100(10 rows)
```





