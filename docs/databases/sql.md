---
layout: default
title: SQL
description: Notes on SQL.
nav_order: 2
parent: Databases
permalink: /databases/sql
---

<!-- prettier-ignore-start -->

# SQL
{:.no_toc}

If you work with data then learning SQL is a must.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

SQL (Structured Query Language) is a domain-specific language used for managing and querying data stored in RDMSes.

SQL is based on relational algebra and is often broken into multiple parts:

- DDL (Data Definition Language)
- DML (Data Manipulation Language)

{% cite database-system-concepts -l 65-6 %}

## DDL

The SQL DDL is used for defining and modifying tables and table schemas.

### Types

SQL supports many built-in types.

Some common types include:

- `char(n)` An n-length character string.
- `varchar(n)` A variable-length character string with maximum length n.
- `int` An integer (machine-dependant size).
- `smallint` A small integer (machine-dependant size).
- `numeric(p, d)` A fixed-point number with a sign bit, p digits, and d of the p digits to the right of the decimal point.
- `double` A double precision floating point (machine-dependant size).
- `float(n)` A float with at least n precision.
- `text(n)` Holds a string with a maximum length of n or 65,535 bytes.

{% cite database-system-concepts -l 67 %}

Each type may include a special null value.

### Create table

Schemas can be defined using the create table command.

The create table command specifies column names as well as their type.

In addition, create table supports default values and optional integrity constraints:

```sql
create table employees (
  employee_id int primary key,
  preferred_name varchar(250),
  job_id int not null,
  display_job_title varchar(250),
  salary int default 0,
  start_date datetime not null
);
```

You can also define indexes: `create index ix_salary index on employees(salary);` {% cite database-system-concepts -l 164 %}.

Tables can be deleted with the drop table command: `drop table employees;`.

The alter table statement can be used to add, delete, or modify columns to an existing table:

```sql
alter table
  employees
add
  first_name varchar(100),
add
  last_name varchar(100);
```

### Constraints

Constraints can be defined on columns. SQL prevents updates to the database that violate a constraint {% cite database-system-concepts -l 69 %}.

The primary key constraint specifies that a column (or a set of columns) uniquely identifies the relation and is nonnull:

```sql
create table employees (
  employee_id int primary key,
  -- ...
);
```

The foreign key constraint specifies a column (or a set of columns) that must correspond to the primary key values of columns in another table:

```sql
create table referrals (
  employee_id int,
  -- ...
  foreign key (employee_id) references employees(employee_id)
);
```

<!-- Integrity constraints ensure that changes made to a database do not result in data inconsistency 145.

Security constraints guard against unauthorized users 145. -->

The check clause can be used to ensure column values satisfy a predicate:

```sql
create table employees (
  -- ...
  salary int check (salary >= 18000),
);
```

## Queries

Queries are performed using the select statement, which produces a table as a result (the result set).

```sql
select * from employees;
```

Select statements are made up of multiple clauses.

The select clause defines the columns that are selected. It also be used to derive new columns:

```sql
select
  employee_id as id,
  join_date,
  round(salary / 12) as monthly_salary
from
  employees;

```

The from clause defines which table (rowset) to read the data from. If multiple tables are defined (e.g. `from employees, candidates`) then the rowset is a Cartesian product of the tables.

The where clause takes a predicate and applies it to the result table:

```sql
select
  *
from
  employees
where
  salary > 100000;
```

The predicate can use logical connectives `and`, `or`, and `not`.

### Grouping

The group by clause forms groups of rows based on the defined grouping columns:

```sql
select
  major_version,
  minor_version
from
  app_releases
group by
  major_version,
  minor_version
```

### Aggregation

Aggregate functions take a collection of values as input and return a single value.

There are 5 builtin SQL aggregate functions:

- `avg`
- `min`
- `max`
- `sum`
- `count`

{% cite database-system-concepts -l 91 %}

The input to `avg` and `sum` must be numbers but the other functions can operate on collections of nonnumeric data types {% cite database-system-concepts -l 91 %}.

The following query returns a single row with the average employee salary:

```sql
select
  avg(salary) as average_salary
from
  employees;
```

In general, aggregate functions ignore null values in their input (except `count(*)`). This behavior can be confusing and so it's best to avoid nulls in your table {% cite database-system-concepts -l 96 %}.

### Having clause

The having clause operates on groups (as opposed to the where clause which operates on individual rows before they are grouped).

```sql
select
  major_version,
  minor_version,
  count(patch_version) as patch_count
from
  app_releases
group by
  major_version,
  minor_version
having
  patch_count > 1
```

### Order by clause

Select results can be ordered with the order by clause.

For `order by C1, C2, ...`, results are ordered first by C1, in case of ties they are ordered by C2, etc.

The sort order is ascending by default. The `desc` keyword sets the sort order to descending:

```sql
select
  *
from
  log_events
order by
  event_time desc,
  event_name asc;
```

### Subqueries

SQL supports nested subqueries, i.e. select statements nested within select statement.

In the where clause:

```sql
select
  *
from
  employees
where
  employee_id in (
    select
      employee_id
    from
      referrals
    where
      hiring_decision = 'hire'
  );
```

In the from clause (supported by most SQL implementations):

```sql
select
  dept_name,
  avg_salary
from
  (
    select
      dept_name,
      avg(salary) as avg_salary
    from
      employees
    group by
      dept_name
  )
where
  avg_salary > 120000;
```

A **correlated subquery** uses values of its outer query. These can be slow.

### Scalar subqueries

**Scalar subqueries** are subqueries that result in a single column and a single row.

Scalar subqueries can be used wherever expressions returning a single value are permitted:

```sql
select
  *
from
  employee
where
  salary > (
    select
      avg(salary)
    from
      employees
  );
```

Runtime errors can occur when using scalar subqueries, since it's not always possible to determine whether a query will return a single row at compile time {% cite database-system-concepts -l 107 %}.

## Joins

A join clause allows you to combine rows from two or more tables.

There are different classes of join.

### Cross join

A cross join returns the Cartesian product of rows from tables in the join.

```sql
select
  *
from
  employees
  cross join referrals;
```

Cross joins are performed implicitly in from statements with multiple tables:

```sql
select * from employee, referrals;
```

### Inner join

An inner join returns returns all rows from the Cartesian product of two tables that satisfy a given join predicate.

```sql
select
  employee_id,
  referral_id
from
  employees
  inner join referrals on employees.employee_id = referrals.employee_id;
```

An **equi-join** is a type of join that uses only equality comparisons in the join predicate.

A **natural join** ($$r \bowtie s$$) is a special case of equi-join where the result is a combination of rows that have matching values for columns with the same names. The resulting table of a natural join contains a concatenation of the rows with matching values but with only one column for each of the shared columns {% cite database-system-concepts -l 127 %}.

```sql
select employee_id, referral_id from employees natural join referrals;
```

### Outer join

Outer joins performs a join between two tables but preserves unmatched rows.

A full outer join preserves all non-matching rows in both tables.

```sql
select employee_id, referral_id from employees full outer join referrals;
```

A left outer join preserves rows in the table to the left of the join operator.

```sql
select employee_id, referral_id from employees left outer join referrals;
```

A right outer join preserves rows in the table to the right of the join operator.

```sql
select employee_id, referral_id from employees right outer join referrals;
```

## Modifications

SQL provides an interface for inserting, updating, and deleting rows.

### Deletion

The delete statement is used to delete rows:

```sql
delete from
  employees
where
  employee_id = 12
```

### Insertion

The insert into statement is used to insert rows:

```sql
insert into
  employees (employee_id, job_id, start_date, salary)
values
  (2, 1, NOW(), 90000);
```

### Updates

The update statement is used to update rows:

```sql
update
  employees
set
  salary = salary * 1.05
where
  salary < 500000;
```

## Views

A view is the result set of a stored query on the data.

```sql
create view high_earners as
select
  employee_id
from
  employees
where
  salary > 1000000;

select * from high_earners;
```

A **materialized view** is a cached view. The implementation is vendor-specific, as it's not standardized in SQL {% cite database-system-concepts -l 140 %}.

In some implementations, a materialized view is recalculated when one of its underlying dependencies is modified: either immediately or lazily when the view is accessed. An alternative approach is to update materialized views periodically {% cite database-system-concepts -l 140 %}.

## Transactions

Transactions are a sequence of SQL statements that are either applied atomically or rolled back in the case of failure {% cite database-system-concepts -l 144 %}.

An example PostgreSQL transaction:

```sql
begin;

update
  accounts
set
  balance = balance - 1000
where
  account_id = 1;

update
  accounts
set
  balance = balance + 1000
where
  account_id = 2;

commit;
```

## Authorization

SQL supports authorization.

Authorization privileges includes:

- select
- insert
- update
- delete

{% cite database-system-concepts -l 166 %}

Users can be authorized to have none, all, or a combination of privileges on specified parts of a database using the grant statement:

```sql
grant delete on employees to alice;
```

Roles are also supported:

```sql
create role accountant;
grant accountant to bob;

grant select on employees to accountant;
```

Privileges can be revoked with the revoke statement:

```sql
revoke grant option for select on referrals from bob;
```

## References

{% bibliography --cited_in_order %}
