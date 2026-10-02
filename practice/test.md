\*\*ВАРИАНТ 1.

1. Вывести число работников с зарплатой более 10000 в тех отделах, где число всех работников более одного.
   Вывести: код отдела; название отдела; число работников.
   Сортировать: код отдела.

```SQL
select d.department_id, d.department_name, count(e.employee_id) as emp_count
from departments d
left join employees e on d.department_id=e.department_id
where e.salary>10000
group by d.department_id
having count(e.employee_id)>1
order by d.department_id

select d.department_id, d.department_name,
       count(e.employee_id) filter (where e.salary > 10000) as emp_count
from departments d
join employees e on e.department_id = d.department_id
group by d.department_id, d.department_name
having count(*) > 1
order by d.department_id;
```

2. Для каждой (!) страны подсчитать суммарную зарплату менеджеров, трудоустроенных в этой стране.
   Вывести: код страны, название страны, суммарная зарплата менеджеров.
   Сортировать: код страны.

```SQL
with country_sum as (
    select c.country_id, sum(e.salary) as su
    from countries c
    left join locations l on c.country_id=l.country_id
    left join departments d on l.location_id = d.location_id
    left join employees e on d.department_id = e.department_id
    where e.employee_id in (select q.manager_id from employees q)
    group by c.country_id
)

select c.country_id, c.country_name, coalesce(s.su, 0)
from countries c
left join country_sum s on c.country_id=s.country_id
order by c.country_id
```

3. Из страны (стран) с наибольшим количеством подразделений выбрать менеджера (менеджеров), оклад которого максимален.
   Вывести: код работника, оклад.
   Сортировать: код работника.

```SQL
with count_dept as(
    select c.country_id, count(d.department_id) as cnd
    from countries c
    join locations l on c.country_id=l.country_id
    join departments d on l.location_id=d.location_id
  	group by c.country_id
),
max_dept as (
    select max(cnd) as mcdn
    from count_dept
),
emp as (
  select e.employee_id, e.salary
  from employees e
  join departments d on d.department_id=e.department_id
  join locations l on l.location_id=d.location_id
  join count_dept cd on l.country_id = cd.country_id, max_dept mcdn
  where cd.cnd=mcdn and e.employee_id in (select manager_id from employees)
)

select employee_id, salary
from emp
where salary=(select max(salary) from emp)
order by employee_id

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

\*\*

ВАРИАНТ 2.
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

ВАРИАНТ 3.
Подсчитать количество работников из региона Америка на каждой (!) должности.
Вывести: код должности, название должности, количество работников.
Сортировать: число работников по убыванию, название должности.

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
order by jc.cnt desc, j.job_id
```

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

ВАРИАНТ 4.
Выбрать тех менеджеров, которые ни разу не меняли должность.
Вывести: код работника, фамилия, название настоящей должности.
Сортировать: код работника.

```SQL
select e.employee_id, e.last_name, j.job_title
from employees e
join jobs j on e.job_id=j.job_id

where e.employee_id in (select manager_id from employees)
      and not exists (select 1 from job_history js2 where js2.employee_id =
      e.employee_id )
order by e.employee_id
```

Найти те отделы, все сотрудники которых имеют менеджеров из той же страны. Не учитывать отделы без сотрудников.
Вывести: код отдела, название отдела, страна.
Сортировать: код отдела.

```SQL
select d.department_id, d.department_name, l.country_id
from departments d
join locations l on l.location_id = d.location_id
where exists (
    select 1 from employees e
    where e.department_id = d.department_id
)
and not exists (
    select 1
    from employees e
    left join employees m on m.employee_id = e.manager_id
    left join departments dm on dm.department_id = m.department_id
    left join locations lm on lm.location_id = dm.location_id
    where e.department_id = d.department_id
      and lm.country_id is distinct from l.country_id
)
order by d.department_id;

select d.department_id, d.department_name, l.country_id
from departments d
join locations l on l.location_id = d.location_id
join employees e on e.department_id = d.department_id
left join employees m on m.employee_id = e.manager_id
left join departments dm on dm.department_id = m.department_id
left join locations lm on lm.location_id = dm.location_id
group by d.department_id, d.department_name, l.country_id
having bool_and(lm.country_id = l.country_id) is true
order by d.department_id;
```

Определить год (годы), в котором было трудоустроено больше всего менеджеров.
Вывести: год трудоустройства.

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

ВАРИАНТ 5.
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
where c.country_id='Americas'
      and e.employee_id in (select manager_id from employees)
group by e.employee_id
order by e.employee_id
```

Определить менеджера (менеджеров), который эффективнее всех продвинулся по карьерной лестнице. Под эффективностью будем понимать разницу максимальных зарплат на новой и старой должностях (из таблицы jobs).
Вывести: код работника, название отдела.
Сортировать: код работника, название отдела.

```SQL
with transitions as (
    select jh.employee_id,
           jh.job_id as old_job_id,
           coalesce(
               lead(jh.job_id) over (partition by jh.employee_id order by jh.start_date),
               e.job_id
           ) as new_job_id
    from job_history jh
    join employees e on e.employee_id = jh.employee_id
),
efficiency as (
    select t.employee_id,
           max(jnew.max_salary - jold.max_salary) as eff
    from transitions t
    join jobs jold on jold.job_id = t.old_job_id
    join jobs jnew on jnew.job_id = t.new_job_id
    where t.employee_id in (select manager_id from employees)
    group by t.employee_id
)
select e.employee_id, d.department_name
from efficiency ef
join employees e on e.employee_id = ef.employee_id
join departments d on d.department_id = e.department_id
where ef.eff = (select max(eff) from efficiency)
order by e.employee_id, d.department_name;
```
