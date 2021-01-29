---
layout: default
title: The relational model
description: Notes on the relational model.
nav_order: 5
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

<!-- Relational algebra consists of a set of operations. It takes one or two relations as input and produces a new relation as a result 48. -->

Relational algebra consists of a set of operations that can be composed into a relational-algebra expression {% cite database-system-concepts -l 50 %}.

Operations include:

- Select ($$\sigma$$)
- Project ($$\Pi$$)
- Cartesian product ($$\times$$)
- Rename ($$\rho$$)
- Join ($$\bowtie$$)

### Select operation

The **select operation** ($$\sigma$$) selects tuples that satisfy a given predicate. The predicate is indicated as a subscript to the operator and the argument relation is denoted in parentheses, e.g., $$\sigma_{\text{dept_name}="Physics"}(\text{instructor})$$ {% cite database-system-concepts -l 49 %}.

The select predicate supports comparison operators (=, ≠, <, ≤, >, and ≥). Predicates can be combined using the connectives and ∧, or (V) and not ¬ {% cite database-system-concepts -l 49 %}.

### Project

The **project operation** ($$\Pi$$) returns an argument relation with some attributes omitted. The attributes to be kept are part of the subscript to the operator, e.g., $$\Pi_{\text{ID}, \text{name}, \text{salary}}(\text{instructor})$$ 49.

Expressions involving attributes can be included in the project operation, e.g., $$\Pi_{\text{ID}, \text{name}, {\text{salary} / 12}}(\text{instructor})$$ {% cite database-system-concepts -l 50 %}.

### Cartesian product

The **Cartesian-product operation** ($$\times$$) combines information from two relations. e.g., $$r_1 \times r_2$$.

As opposed to a Cartesian-product of sets (which would produce pairs of tuples), a Cartesian product of relations concatenates the tuples into a single tuple {% cite database-system-concepts -l 51 %}.

### Rename

The **rename operation** ($$\rho$$) can be used to rename relations and/or attributes.

$$\rho _{a/b}(R)$$ returns a relation identical to $$R$$ except attributes $$b$$ are renamed to $$a$$.

### Natural join

The **natural join operation** ($$\bowtie$$) is a binary operator. The result of the natural join $$R \bowtie S$$ is a set of all combinations of tuples in $$R$$ and $$S$$ that are equal on their common attribute names.

### Set operators

Relations also support set operators like union, set difference, and intersection, for compatible relations (where the definition of compatible relation depends on the set operation) {% cite database-system-concepts -l 54 %}.

### Assignment

The **assignment operation** (←) assigns a relation to a relation variable with a name (like assigning a variable in a programming language). The relation variable can then be used in future operations {% cite database-system-concepts -l 55-6 %}.

## References

{% bibliography --cited_in_order %}
