# Задания контрольной по SQL DQL, сгруппированные по типам

## Тип 1 (В1, В6)

Вывести число работников с зарплатой более 10000 в тех отделах, где число всех работников более одного.
Вывести: код отдела; название отдела; число работников.
Сортировать: код отдела.

```SQL
select d.department_id, d.department_name, count(e.employee_id) filter (where e.salary>10000)
from departments d
join employees e on d.department_id = e.department_id
group by d.department_id
having count(e.employee_id)>1
order by d.department_id
```

## Тип 2 (В1, В7)

Для каждой (!) страны подсчитать суммарную зарплату менеджеров, трудоустроенных в этой стране.
Вывести: код страны, название страны, суммарная зарплата менеджеров.
Сортировать: код страны.

```SQL
with dep_count as (
    select c.country_id, c.country_name, sum(e.salary) as sum
    from countries c
    join locations l on l.country_id = c.country_id
    join departments d on l.location_id = d.location_id
    join employees e on d.department_id = e.department_id
    where e.employee_id in (select manager_id from employees)
    group by c.country_id
)
select c.country_id, c.country_name, coalesce(dc.sum,0)  as avg
    from countries c
    left join dep_count dc on c.country_id = dc.country_id
    order by c.country_id
```

## Тип 3 (В1, В9)

Из страны (стран) с наибольшим количеством подразделений выбрать менеджера (менеджеров), оклад которого максимален.
Вывести: код работника, оклад.
Сортировать: код работника.

```sql
with count_dept as (
    select l.country_id, count(*) as cnd
    from departments d
    join locations l on l.location_id = d.location_id
    group by l.country_id
),
emp as (
    select e.employee_id, e.salary, l.country_id
    from employees e
    join departments d on d.department_id = e.department_id
    join locations l on l.location_id = d.location_id
    where l.country_id in (select country_id from count_dept
                           where cnd = (select max(cnd) from count_dept))
      and e.employee_id in (select manager_id from employees where manager_id is not null)
)
select employee_id, salary
from emp
where salary = (select max(salary) from emp e2 where e2.country_id = emp.country_id)
order by employee_id;
```

## Тип 4 (В2, В7)

Вывести все должности, не встречающиеся в США.
Вывести: код должности, название должности.
Сортировать: название должности.

```SQL
select j.job_id, j.job_title
from jobs j
where j.job_id not in (
    select e.job_id
    from employees e
    join departments d on e.department_id = d.department_id
    join locations l on d.location_id = l.location_id
    where l.country_id='US'
)
order by j.job_title
```

## Тип 5 (В2, В8)

Из стран, в которых работает хотя бы один сотрудник, выбрать страны, число работников в которых меньше, чем среднее число по всем таким странам.
Вывести: код страны, название страны, число работников.
Сортировать: код страны.

```SQL
with country_count as (
select c.country_id, count(e.employee_id) as cnt
from countries c
join locations l on c.country_id=l.country_id
join departments d on d.location_id=l.location_id
join employees e on e.department_id=d.department_id
group by c.country_id
having count(e.employee_id)>=1
),
avg_count as (
select avg(cnt) as avga
from country_count
)

select c.country_id, c.country_name, cc.cnt
from countries c
join country_count cc on c.country_id=cc.country_id, avg_count a
where cc.cnt<a.avga
order by c.country_id
```

## Тип 6 (В2, В8)

Из страны (стран), в которой проживает менеджер (менеджеры) с наименьшим стажем, выбрать работника (работников), в подчинении которого больше всего человек.
Вывести: код работника, число подчиненных сотрудников.
Сортировать: код работника.

```SQL
with min_hire_date as (
    select max(e.hire_date) as mx
    from employees e
    where employee_id in (select manager_id from employees)
), 
country_with_min_hire_date as (
    select l.country_id
    from employees e
    join departments d on e.department_id = d.department_id
    join locations l on d.location_id = l.location_id
    where e.hire_date = (select mx from min_hire_date)
    and employee_id in (select manager_id from employees)
),
manager_count as (
    select e.manager_id, l.country_id, count(e.employee_id) as cnt
    from employees e
    join employees m on e.manager_id = m.employee_id
    join departments d on m.department_id = d.department_id
    join locations l on d.location_id = l.location_id
    group by e.manager_id, l.country_id
)

select mc.manager_id, mc.cnt
from manager_count mc
where mc.country_id in (select country_id from country_with_min_hire_date) and
      mc.cnt = (select max(cnt) from manager_count m2 where mc.country_id = m2.country_id )
order by mc.manager_id
```

## Тип 7 (В3, В8)

**В3:**
Подсчитать количество работников из региона Америка на каждой (!) должности.
Вывести: код должности, название должности, количество работников.
Сортировать: число работников по убыванию, название должности.

**В8:**
Подсчитать количество работников из региона Америка на каждой (!) должности.
Вывести: код должности, название должности, количество работников.
Сортировать: название должности.

```SQL
with job_count as (
select j.job_id, count(e.employee_id) as cnt
from jobs j
left join employees e on j.job_id=e.job_id
left join departments d on e.department_id=d.department_id
left join locations l on d.location_id = l.location_id
left join countries c on l.country_id = c.country_id
left join regions r on c.region_id = r.region_id
where r.region_name = 'Americas'
group by j.job_id, e.job_id
)

select j.job_id, j.job_title, coalesce(jc.cnt, 0)
from jobs j 
left join job_count jc on j.job_id = jc.job_id
order by coalesce(jc.cnt, 0) desc, j.job_title
```

## Тип 8 (В3, В9)

Среди менеджеров, у которых есть хотя бы один подчиненный из Европы, найти менеджера (менеджеров), имеющего наименьший стаж.
Вывести: код работника, дата приема на работу.
Сортировать: код работника.

```SQL
with count_table as (
select e.employee_id, e.hire_date
from employees e
where EXISTS (
    select e2.employee_id 
    from employees e2 
    join departments d on e2.department_id=d.department_id
    join locations l on d.location_id = l.location_id
    join countries c on l.country_id = c.country_id
    join regions r on c.region_id = r.region_id 
    where r.region_name = 'Europe' and e2.manager_id = e.employee_id)
    and e.employee_id in (select manager_id from employees))

select ct.employee_id, ct.hire_date
from count_table ct
where ct.hire_date = (select max(ct2.hire_date) from count_table ct2)
order by ct.employee_id
```

## Тип 9 (В3, В10)

Определить месяц, в котором было трудоустроено больше всего менеджеров.
Вывести: месяц трудоустройства.

```SQL

with month_count as (
select extract(month from hire_date) as mon, count(e.employee_id) as cnt
from employees e
where e.employee_id in (select manager_id from employees)
group by mon)

select mc.mon
from month_count mc
where mc.cnt = (select max(cnt) from month_count )
```

## Тип 10 (В4, В9)

**В4:**
Выбрать тех менеджеров, которые ни разу не меняли должность.
Вывести: код работника, фамилия, название настоящей должности.
Сортировать: код работника.

**В9:**
Выбрать тех менеджеров, которые ни разу не меняли должность.
Вывести: код работника, Фамилия, название настоящей должности.
Сортировать: код работника.

```sql
select e.employee_id, e.last_name, j.job_title
from employees e
join jobs j on e.job_id=j.job_id

where e.employee_id in (select manager_id from employees)
      and not exists (select 1 from job_history js2 where js2.employee_id =
      e.employee_id )
order by e.employee_id
```

## Тип 11 (В4, В10)

Найти те отделы, все сотрудники которых имеют менеджеров из той же страны. Не учитывать отделы без сотрудников.
Вывести: код отдела, название отдела, страна.
Сортировать: код отдела.

```SQL
select d.department_id, d.department_name, l.country_id
from departments d
join locations l on l.location_id = d.location_id
join employees e on e.department_id = d.department_id
left join employees m on m.employee_id = e.manager_id
left join departments dm on dm.department_id = m.department_id
left join locations lm on lm.location_id = dm.location_id
group by d.department_id, d.department_name, l.country_id
having bool_and(coalesce(lm.country_id = l.country_id, false)) is true
order by d.department_id;
```

## Тип 12 (В4, В6)

**В4:**
Определить год (годы), в котором было трудоустроено больше всего менеджеров.
Вывести: год трудоустройства.

**В6:**
Определить год (годы), в котором было трудоустроено больше всего менеджеров.
Вывести: год трудоустройства.
Сортировать: год трудоустройства.

```SQL
with year_count as (
select extract(year from hire_date) as mon, count(e.employee_id) as cnt
from employees e
where e.employee_id in (select manager_id from employees)
group by mon)

select yc.mon
from year_count yc
where yc.cnt = (select max(cnt) from year_count )
```

## Тип 13 (В5, В10)

Вывести тех работников, менеджеры которых трудоустроены в другой стране.
Вывести: код работника, название страны работника, код работника (менеджера), название страны менеджера.
Сортировать: код работника.

```SQL
select e.employee_id, ce.country_name, m.employee_id, cm.country_name
from employees e
join employees m on e.manager_id = m.employee_id
join departments de on de.department_id = e.department_id
join departments dm on dm.department_id = m.department_id
join locations le on de.location_id = le.location_id
join locations lm on dm.location_id = lm.location_id
join countries ce on le.country_id = ce.country_id
join countries cm on lm.country_id = cm.country_id
where ce.country_id <> cm.country_id
order by e.employee_id
```

## Тип 14 (В5, В6)

Выбрать менеджеров из Америки, которые хотя бы раз меняли должность.
Вывести: код работника, число изменений должности.
Сортировать: код работника.

```SQL
select e.employee_id, count(*) as cnt
from employees e
join departments d on e.department_id=d.department_id
join locations l on d.location_id = l.location_id
join countries c on l.country_id = c.country_id
join job_history jh on jh.employee_id = e.employee_id
join regions r on c.region_id = r.region_id
where r.region_name='Americas'
      and e.employee_id in (select manager_id from employees)
group by e.employee_id
order by e.employee_id
```

## Тип 15 (В5, В7)

**В5:**
Определить менеджера (менеджеров), который эффективнее всех продвинулся по карьерной лестнице. Под эффективностью будем понимать разницу максимальных зарплат на новой и старой должностях (из таблицы jobs).
Вывести: код работника, название отдела.
Сортировать: код работника, название отдела.

**В7:**
Определить менеджера (менеджеров), который эффективнее всех продвинулся по карьерной лестнице. Под эффективностью будем понимать разницу максимальных зарплат на новой и старой должностях.
Вывести: код работника, название отдела.
Сортировать: код работника, название отдела.
