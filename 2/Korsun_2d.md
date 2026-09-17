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


6.Определите все должности встречающиеся в странах Европы
**Вывести:** название должности, страна.
**Сортировать:** страна, название должности.

```sql
SELECT DISTINCT j.job_title, c.country_name
FROM employees e
JOIN jobs j ON e.job_id = j.job_id
JOIN departments d ON e.department_id = d.department_id
JOIN locations l ON d.location_id = l.location_id
JOIN countries c ON l.country_id = c.country_id
JOIN regions r ON c.region_id = r.region_id
WHERE r.region_name = 'Europe'
ORDER BY c.country_name, j.job_title;
```


7.Для каждой страны подсчитать суммарную зарплату работников, трудоустроенных в подразделениях, расположенных в этой стране. Если для какой-то страны никто не трудоустроен, не включать ее в результат.
**Вывести:** код страны, название страны, суммарная зарплата.
**Сортировать:** код страны

```sql
SELECT c.country_id, c.country_name, SUM(e.salary) AS total_salary
FROM countries c
JOIN locations l ON l.country_id = c.country_id
JOIN departments d ON d.location_id = l.location_id
JOIN employees e ON e.department_id = d.department_id
GROUP BY c.country_id, c.country_name
ORDER BY c.country_id;
```


8.Определить год, в котором было трудоустроено больше всего человек.
**Вывести:** Год.
**Сортировать:** Год

```sql
SELECT EXTRACT(YEAR FROM hire_date)::int AS hire_year
FROM employees
GROUP BY EXTRACT(YEAR FROM hire_date)
HAVING COUNT(*) = (
    SELECT MAX(cnt) FROM (
        SELECT COUNT(*) AS cnt FROM employees
        GROUP BY EXTRACT(YEAR FROM hire_date)
    ) t
)
ORDER BY hire_year;
```


9.Определить работника (работников), который эффективнее всех продвинулся по карьерной лестнице. Под эффективностью будем понимать разницу максимальной зарплаты на новой должности и минимальной на старой должности.
**Вывести:** код работника, его фамилию и имя, название отдела.
**Сортировать:** Код работника

```sql
WITH job_seq AS (
    SELECT jh.employee_id, jh.job_id AS old_job_id,
           LEAD(jh.job_id) OVER (PARTITION BY jh.employee_id ORDER BY jh.start_date) AS next_job_id
    FROM job_history jh
),
transitions AS (
    SELECT js.employee_id, js.old_job_id,
           COALESCE(js.next_job_id, e.job_id) AS new_job_id
    FROM job_seq js
    JOIN employees e ON e.employee_id = js.employee_id
),
efficiency AS (
    SELECT t.employee_id,
           jnew.max_salary - jold.min_salary AS eff
    FROM transitions t
    JOIN jobs jold ON jold.job_id = t.old_job_id
    JOIN jobs jnew ON jnew.job_id = t.new_job_id
)
SELECT e.employee_id, e.last_name, e.first_name, d.department_name
FROM efficiency ef
JOIN employees e ON e.employee_id = ef.employee_id
JOIN departments d ON e.department_id = d.department_id
WHERE ef.eff = (SELECT MAX(eff) FROM efficiency)
ORDER BY e.employee_id;
```

При решении задания возник вопрос, как трактовать старую и новую
должность, если у сотрудника было больше одного изменения должности в job_history.

Поэтому прикрепляю второй варинат решения, где для каждого сотрудника берётся 
не последовательная пара должностей (как сделано в варианте выше), а весь набор его должностей, и MAX/MIN считаются по всему этому набору сразу 
(а не max по парам)

```sql
WITH efficiency AS (
    SELECT
        e.employee_id,
        MAX(j.max_salary) - MIN(j.min_salary) AS efficiency
    FROM employees e
    JOIN job_history jh ON jh.employee_id = e.employee_id
    JOIN jobs j ON j.job_id IN (jh.job_id, e.job_id)
    GROUP BY e.employee_id
)
SELECT e.employee_id, e.last_name, e.first_name, d.department_name
FROM efficiency ef
JOIN employees e ON e.employee_id = ef.employee_id
JOIN departments d ON d.department_id = e.department_id
WHERE ef.efficiency = (SELECT MAX(efficiency) FROM efficiency)
ORDER BY e.employee_id;
​```

10.Найти всех таких сотрудников, менеджер которых трудоустроен в другом отделе.
**Вывести:** код сотрудника, название отдела сотрудника, название страны сотрудника, код менеджера, название отдела менеджера, название страны менеджера.
**Сортировать:** Код работника.

```sql
SELECT e.employee_id,
       de.department_name AS emp_department,
       ce.country_name AS emp_country,
       m.employee_id AS manager_id,
       dm.department_name AS manager_department,
       cm.country_name AS manager_country
FROM employees e
JOIN departments de ON e.department_id = de.department_id
JOIN locations le ON de.location_id = le.location_id
JOIN countries ce ON le.country_id = ce.country_id
JOIN employees m ON e.manager_id = m.employee_id
JOIN departments dm ON m.department_id = dm.department_id
JOIN locations lm ON dm.location_id = lm.location_id
JOIN countries cm ON lm.country_id = cm.country_id
WHERE e.department_id <> m.department_id
ORDER BY e.employee_id;
```