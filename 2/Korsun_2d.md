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