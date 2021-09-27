---
layout: default
title: The relational model
description: Notes on the relational model.
nav_order: 1
parent: Databases
permalink: /databases/relational-model
---

<!-- prettier-ignore-start -->

# The relational model
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

The **relational model** is an approach to managing data, introduced by Edgar Codd in 1969.

In the relational model, tables are known as relations. A relation is a set of tuples (rows) and each relation has a number of attributes (columns) {% cite database-system-concepts -l 37-9 %}.

Each attribute has a domain which specifies the valid values for the attribute. Domains must be atomic {% cite database-system-concepts -l 40 %}.

The null value is a special value signifying that the value is unknown or unspecified {% cite database-system-concepts -l 40 %}.

SQL is loosely based on relational algebra. Most databases use relational algebra or something based on relational algebra for their internal representation of queries.

## Database schema

A database schema is the logical design of a database {% cite database-system-concepts -l 41 %}.

A relation schema specifies type information for relations. A schema includes an ordered set of attributes, as well as the domain of each attribute.

Schemas are often written in the form $$\text{relation}(\text{attribute_name_a}, ..., \text{attribute_name_n})$$.

## Keys

Keys are used to identify individual tuples.

A **superkey** is a set of one or more attributes that collectively uniquely identify a tuple in a relation {% cite database-system-concepts -l 43 %}.

A **candidate key** is a superkey where no proper subset is a superkey. A relation may have multiple candidate keys {% cite database-system-concepts -l 44 %}.

A **primary key** is a candidate key chosen as the primary identifier for tuples in a relation. By convention, primary keys are listed before other attributes in a schema and are underlined {% cite database-system-concepts -l 44 %}.

For example, consider a department relation with a primary key of building and room number:

$$\text{classroom}(\underline{\text{building}}, \underline{\text{room_number}}, \text{capacity})$$

{% cite database-system-concepts -l 44 %}

A **foreign key** is a set of attributes in a table that refers to the primary key of another table {% cite database-system-concepts -l 45 %}.

<!-- TODO: ## Schema Diagrams -->

## Relational algebra

Relational algebra uses algebraic structures for modelling data and defining queries on it.

Relational algebra consists of a set of operations that can be composed into a relational-algebra expression {% cite database-system-concepts -l 50 %}.

The fundamental operations include:

- Select ($$\sigma$$)
- Project ($$\Pi$$)
- Set-union ($$\cup$$)
- Set-difference ($$-$$)
- Cartesian product ($$\times$$)
- Rename ($$\rho$$)

### Select operation

The **select operation** ($$\sigma_{\text{p}}(\text{r})$$) selects tuples that satisfy a given predicate $$p$$ from a relation $$r$$.

For example, $$\sigma_{\text{name}="Paddington"}(\text{station})$$.

The select predicate supports comparison operators (=, ≠, <, ≤, >, and ≥). Predicates can be combined using the connectives and (∧), or (∨) and not (¬) {% cite database-system-concepts -l 49 %}.

### Project

The **project operation** $$\Pi_{\text{a}, \text{b}, ... }(\text{r})$$ returns a relation with the set of tuples from the argument relation $$r$$ with only the specified attributes remaining.

Expressions involving attributes can be included in the project operation, e.g. $$\Pi_{\text{ID}, \text{name}, {\text{salary} / 12}}(\text{employee})$$ will return a tuple containing the ID, name, and monthly pay of an employee.

### Set-union

The **set-union operation** ($$r \cup s$$) returns all tuples from two relations $$r$$ and $$s$$.

$$r$$ and $$s$$ must have compatible schemas:

- $$r$$ and $$s$$ must have the same arity.
- For each attribute $$r[i]$$ $$s[i]$$ must have the same domain.

### Set-difference

The **set-difference operation** ($$ r - s $$) returns all tuples that are only in $$r$$ but not in $$s$$.

$$r$$ and $$s$$ must have compatible schemas (as defined in the set-union section).

### Cartesian product

The **Cartesian-product operation** ($$r \times s$$) returns each tuple in $$r$$ concatenated with each tuple of $$s$$.

There are no constraints on the schemas of the two relations.

In the case of overlapping attribute names, the attribute names are distinguished by prepending the relation name. e.g. The schema of $$r(\text{a}, \text{b}) \times s(\text{b}, \text{c})$$ could be written as $$r(\text{a}, \text{r.b}, \text{s.b}, \text{c} )$$.

_Note: as opposed to a Cartesian-product of sets which would produce pairs of tuples, a Cartesian product of relations concatenates the tuples into a single tuple {% cite database-system-concepts -l 51 %}._

### Rename

The **rename operation** ($$\rho _{x}(E)$$) can be used to rename relations and/or attributes.

$$E$$ is an expression that produces a relation ($$E$$ can also be a named relation or relation variable). $$x$$ is the new name of the relation.

Rename applies within a relation expression. It does not create a relation variable.

$$\rho _{a/b}(R)$$ returns a relation identical to $$r$$ except attributes $$b$$ are renamed to $$a$$.

### Set-intersection

The **set-intersection operation** ($$ r \cap s $$) returns all tuples that are only in both $$r$$ and $$s$$.

$$r$$ and $$s$$ must have compatible schemas (as defined in the set-union section).

### Natural join

The **natural join operation** ($$r \bowtie s$$) returns a set of all combinations of tuples in $$r$$ and $$s$$ that are equal on their common attribute names.

### Assignment

The **assignment operation** (←) assigns a relation to a relation variable with a name (like assigning a variable in a programming language). The relation variable can then be used in future operations {% cite database-system-concepts -l 55-6 %}.

## References

{% bibliography --cited_in_order %}
