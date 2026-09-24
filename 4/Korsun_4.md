 

1. Посчитать среднюю зарплату работников, работающих на должности с наибольшим разбросом между минимальным и максимальным порогом зарплаты (min_salary, max_salary). Если таких должностей несколько, то посчитать среднюю зарплату для каждой из них.

**Вывести:** код должности, средняя зарплата.
**Сортировать:** код должности.

```sql
SELECT e.job_id, AVG(e.salary) AS avg_salary
FROM employees e
WHERE e.job_id IN (
    SELECT job_id
    FROM jobs
    WHERE max_salary - min_salary = (SELECT MAX(max_salary - min_salary) FROM jobs)
)
GROUP BY e.job_id
ORDER BY e.job_id;
```

2. Отобразить список всех работников, которые работают в двух странах с наименьшим суммарным заработком сотрудников.

**Вывести:** код сотрудника.
**Сортировать:** код сотрудника.

```sql
WITH country_sum AS (
    SELECT l.country_id, SUM(e.salary) AS total
    FROM employees e
    JOIN departments d ON d.department_id = e.department_id
    JOIN locations l ON l.location_id = d.location_id
    GROUP BY l.country_id
    ORDER BY total
    LIMIT 2
)
SELECT e.employee_id
FROM employees e
JOIN departments d ON d.department_id = e.department_id
JOIN locations l ON l.location_id = d.location_id
WHERE l.country_id IN (SELECT country_id FROM country_sum)
ORDER BY e.employee_id;
```

3. Разбить все зарплаты на 4 уровня: 0‒5000, >5000‒10000, >10000‒15000, >15001‒1000000. Для каждого определить: количество работников, суммарную зарплату, среднюю зарплату, количество различных должностей у сотрудников в группе.

**Вывести:** номер группы, количество работников, суммарная зарплата, средняя зарплата, количество различных должностей.
**Сортировать:** номер группы.

Пояснение: границы групп включают верхнее значение (до 5000, до 10000, до 15000), а «>15001» в условии считается опечаткой, поэтому четвёртая группа — всё, что больше 15000.

```sql
SELECT grp,
       COUNT(*)               AS emp_count,
       SUM(salary)            AS total_salary,
       ROUND(AVG(salary), 2)  AS avg_salary,
       COUNT(DISTINCT job_id) AS job_count
FROM (
    SELECT salary, job_id,
           CASE
               WHEN salary <= 5000  THEN 1
               WHEN salary <= 10000 THEN 2
               WHEN salary <= 15000 THEN 3
               ELSE 4
           END AS grp
    FROM employees
) t
GROUP BY grp
ORDER BY grp;
```

4. Посчитать среднюю зарплату работников, работающих на должности с наименьшим разбросом между минимальным и максимальным порогом зарплаты (min_salary, max_salary). Если таких должностей несколько, то посчитать среднюю для каждой из них.

**Вывести:** код должности, средняя зарплата работников по данной должности.
**Сортировать:** код должности.

```sql
SELECT e.job_id, AVG(e.salary) AS avg_salary
FROM employees e
WHERE e.job_id IN (
    SELECT job_id
    FROM jobs
    WHERE max_salary - min_salary = (SELECT MIN(max_salary - min_salary) FROM jobs)
)
GROUP BY e.job_id
ORDER BY e.job_id;
```

5. Среди однофамильцев выбрать работника с зарплатой выше средней по его отделу. Если у человека нет однофамильцев, в результат его не выводить.

**Вывести:** фамилия, код работника, зарплата.
**Сортировать:** фамилия, код работника.

```sql
SELECT e.last_name, e.employee_id, e.salary
FROM employees e
WHERE EXISTS (
        SELECT 1
        FROM employees o
        WHERE o.last_name = e.last_name
          AND o.employee_id <> e.employee_id
      )
  AND e.salary > (
        SELECT AVG(x.salary)
        FROM employees x
        WHERE x.department_id = e.department_id
      )
ORDER BY e.last_name, e.employee_id;
```

6. Среди первых трех самых высокооплачиваемых сотрудников отобрать того, у которого больше всего подчиненных.

**Вывести:** код работника.
**Сортировать:** код работника.

```sql
WITH top3 AS (
    SELECT employee_id
    FROM employees
    ORDER BY salary DESC, employee_id
    LIMIT 3
),
cnt AS (
    SELECT t.employee_id, COUNT(s.employee_id) AS sub_count
    FROM top3 t
    LEFT JOIN employees s ON s.manager_id = t.employee_id
    GROUP BY t.employee_id
)
SELECT employee_id
FROM cnt
WHERE sub_count = (SELECT MAX(sub_count) FROM cnt)
ORDER BY employee_id;
```

7. Среди первых трех лидирующих по количеству подчиненных менеджеров выбрать менеджера с наименьшим стажем.

**Вывести:** код работника.
**Сортировать:** код работника.

```sql
WITH top3 AS (
    SELECT m.employee_id, m.hire_date, COUNT(*) AS sub_count
    FROM employees m
    JOIN employees s ON s.manager_id = m.employee_id
    GROUP BY m.employee_id, m.hire_date
    ORDER BY sub_count DESC, m.employee_id
    LIMIT 3
)
SELECT employee_id
FROM top3
WHERE hire_date = (SELECT MAX(hire_date) FROM top3)
ORDER BY employee_id;
```

8. Среди работников, у которых разница зарплаты с их менеджером менее 5000 выбрать того, который был трудоустроен раньше остальных.

**Вывести:** фамилия сотрудника, имя.

Пояснение: разница зарплат берётся по модулю, а если раньше всех приняты несколько сотрудников, выводятся все можно указать limit 1, чтобы было строгу по условию и выводился ровно 1 человек

```sql
WITH cand AS (
    SELECT e.last_name, e.first_name, e.hire_date
    FROM employees e
    JOIN employees m ON m.employee_id = e.manager_id
    WHERE ABS(m.salary - e.salary) < 5000
)
SELECT last_name, first_name
FROM cand
WHERE hire_date = (SELECT MIN(hire_date) FROM cand);
```

9. Из страны, в которой проживает сотрудник с наибольшим стажем (если таких несколько, то рассмотреть страну для каждого из них), выбрать работника, с наибольшей зарплатой его подчиненных.

**Вывести:** ИД работника.
**Сортировать:** ИД работника.

Пояснение: «наибольшая зарплата подчинённых» понята как максимальная суммарная зарплата прямых подчинённых, а лидер определяется отдельно для каждой найденной страны.

```sql
WITH oldest_country AS (
    SELECT DISTINCT l.country_id
    FROM employees e
    JOIN departments d ON d.department_id = e.department_id
    JOIN locations l ON l.location_id = d.location_id
    WHERE e.hire_date = (SELECT MIN(hire_date) FROM employees)
),
cand AS (
    SELECT m.employee_id, l.country_id, SUM(s.salary) AS sum_sub_salary
    FROM employees m
    JOIN departments d ON d.department_id = m.department_id
    JOIN locations l ON l.location_id = d.location_id
    JOIN employees s ON s.manager_id = m.employee_id
    WHERE l.country_id IN (SELECT country_id FROM oldest_country)
    GROUP BY m.employee_id, l.country_id
)
SELECT c.employee_id
FROM cand c
WHERE c.sum_sub_salary = (
    SELECT MAX(c2.sum_sub_salary)
    FROM cand c2
    WHERE c2.country_id = c.country_id
)
ORDER BY c.employee_id;
```

10. Вывести названия всех отделов, в которых наименьшая зарплата выше средней зарплаты в Америке (region_name = "Americas").

**Вывести:** название отдела.
**Сортировать:** название отдела.

```sql
SELECT d.department_name
FROM departments d
JOIN employees e ON e.department_id = d.department_id
GROUP BY d.department_id, d.department_name
HAVING MIN(e.salary) > (
    SELECT AVG(e2.salary)
    FROM employees e2
    JOIN departments d2 ON d2.department_id = e2.department_id
    JOIN locations l2 ON l2.location_id = d2.location_id
    JOIN countries c2 ON c2.country_id = l2.country_id
    JOIN regions r2 ON r2.region_id = c2.region_id
    WHERE r2.region_name = 'Americas'
)
ORDER BY d.department_name;
```
