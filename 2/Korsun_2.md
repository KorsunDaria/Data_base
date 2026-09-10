6.Определить месяцы между самым первым трудоустроившимся работником и самым последним
**Вывести:** Количество месяцев.
**Сортировать:** Количество месяцев.

```sql
SELECT 
  EXTRACT(YEAR FROM AGE(MAX(hire_date), MIN(hire_date))) * 12
+ EXTRACT(MONTH FROM AGE(MAX(hire_date), MIN(hire_date))) AS months_count
FROM employees
ORDER BY months_count;
```

7.Для каждого отдела определить число сотрудников с окладом более среднего оклада по организации.
**Вывести:** код отдела, число сотрудников.
**Сортировать:** число сотрудников по убыванию, код отдела.

```sql
SELECT department_id, COUNT(*) AS emp_count
FROM employees
WHERE salary > (SELECT AVG(salary) FROM employees)
GROUP BY department_id
ORDER BY emp_count DESC, department_id;
```

8.Для каждого отдела, в котором более двух сотрудников, определить максимальный оклад среди сотрудников.
**Вывести:** код отдела, максимальный оклад.
**Сортировать:** оклад, код отдела.

```sql
SELECT department_id, MAX(salary) AS max_salary
FROM employees
GROUP BY department_id
HAVING COUNT(*) > 2
ORDER BY max_salary, department_id;
```

9.Найти трех сотрудников с максимальными окладами, не превышающими медианный.
**Вывести:** фамилия, имя, оклад.
**Сортировать:** оклад по убыванию, фамилия, имя.

```sql
SELECT last_name, first_name, salary
FROM employees
WHERE salary <= (
    SELECT PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY salary)
    FROM employees
)
ORDER BY salary DESC, last_name, first_name
LIMIT 3;
```

10.Определить месяц, в котором было трудоустроено больше всего человек.
**Вывести:** Порядковый номер месяца
**Сортировать:** Порядковый номер месяца

```sql
WITH counts AS (
  SELECT EXTRACT(MONTH FROM hire_date) AS month_num, COUNT(*) AS cnt
  FROM employees
  GROUP BY month_num
)
SELECT month_num
FROM counts
WHERE cnt = (SELECT MAX(cnt) FROM counts)
ORDER BY month_num;
```

1.Определить суммарную зарплату работников из Великобритании (страна с названием "United Kingdom").
**Вывести:** Сумма

```sql
SELECT SUM(e.salary) AS total_salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id
WHERE c.country_name = 'United Kingdom';
```

2.Для каждого сотрудника из Канады найти данные о менеджере
**Вывести:** фамилию и имя, оклад работника, фамилию и имя, оклад его менеджера
**Сортировать:** фамилия, имя работника.

```sql
SELECT e.last_name, e.first_name, e.salary,
       m.last_name AS mgr_last_name, m.first_name AS mgr_first_name, m.salary AS mgr_salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id
JOIN employees m ON e.manager_id = m.employee_id
WHERE c.country_name = 'Canada'
ORDER BY e.last_name, e.first_name;
```

3.Для стран, в которой трудоустроен хотя бы один сотрудник, подсчитать суммарную зарплату работников, трудоустроенных в подразделениях, расположенных в этой стране.
**Вывести:** название страны, сумма зарплат.
**Сортировать:** сумма зарплат по убыванию, код страны.

```sql
SELECT c.country_name, SUM(e.salary) AS total_salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id
GROUP BY c.country_id, c.country_name
ORDER BY total_salary DESC, c.country_id;
```

4.Определить количество работников из Европы (регион с именем "Europe").
**Вывести:** Количество работников.

```sql
SELECT COUNT(*) AS emp_count
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id
JOIN regions r ON c.region_id = r.region_id
WHERE r.region_name = 'Europe';
```

5.Определить среднюю зарплату работников из Европы (регион с именем "Europe").
**Вывести:** Количество
**Сортировка:** Количество

```sql
SELECT AVG(e.salary) AS avg_salary
FROM employees e
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id
JOIN regions r ON c.region_id = r.region_id
WHERE r.region_name = 'Europe';
```

