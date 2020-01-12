---
layout: default
title: Object-oriented software design
description: Notes on object-oriented software design.
has_children: true
has_toc: false
parent: Software design
permalink: /software-design/object-oriented-software-design
---

<!-- prettier-ignore-start -->

# Object-oriented software design
{:.no_toc}

## Table of contents
{: .no_toc  }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

OO (Object-Oriented) software design patterns are descriptions of communicating objects and classes used to solve a general design problem {% cite the-gang-of-four -l 3 %}.

There are four elements of design patterns:

- **Name**—used to identify the pattern.
- **Problem**—when to use the pattern.
- **Solution**—the elements of the design and how to use them.
- **Consequences**—the results of applying the pattern.

{% cite the-gang-of-four -l 3 %}

## Object-oriented terminology

An **operation** is some action. A class receives a request for an operation from a **client** {% cite the-gang-of-four -l 13 %}.

A **signature** is the name, parameters, and return value of an object’s operations {% cite the-gang-of-four -l 13 %}.

An object's **interface** is the set of all signatures defined on the object {% cite the-gang-of-four -l 13 %}.

A **type** is the name of a specific interface. For example, an object has the type `Window` if it accepts all requests for the operations defined on the `Window` interface {% cite the-gang-of-four -l 13 %}.

A **supertype** is a type that other types inherit part of their interface from {% cite the-gang-of-four -l 13 %}.

A **subtype** is a type that inherits part of its interface from a supertype {% cite the-gang-of-four -l 13 %}.

**Dynamic binding** is the run-time association of a request to an object and one of its operations {% cite the-gang-of-four -l 14 %}.

**Polymorphism** is the ability of an object to take on many forms. Polymorphism means that one object can be substituted for another as long as it has the same interface {% cite the-gang-of-four -l 14 %}.

Objects are created by **instantiating** a class. An **instance** of a class is an object created by instantiating the class {% cite the-gang-of-four -l 15 %}.

**Class inheritance** is where new classes can be defined in terms of existing classes {% cite the-gang-of-four -l 15 %}.

A **subclass** can inherit from a **parent class**. The subclass will include the definitions of all the data and operations of the parent class, but it can override operations defined by its parent class. {% cite the-gang-of-four -l 15 %}.

An **abstract class** is a class that is only used to define a common interface for subclasses. **Abstract operations** are operations declared by an abstract class but not implemented by it. **Concrete classes** are classes that aren't abstract {% cite the-gang-of-four -l 15 %}.

A **mixin class** is "a class that's intended to provide an optional interface or functionality to other classes" {% cite the-gang-of-four -l 16 %}.

**Parameterized types**, also known as generics or templates, are a way of defining types without specifying all of the types that it uses. The unspecified types are then provided as parameters when the type is used {% cite the-gang-of-four -l 21-2 %}.

## Inheritance

**Class inheritance** "defines an object's implementation in terms of another object's implementation" {% cite the-gang-of-four -l 17 %}.

Class inheritance is defined statically at compile-time. It's easy to use class inheritance because it's supported directly by OO languages {% cite the-gang-of-four -l 19 %}.

There are two disadvantages of class inheritance:

1. You can't change the inherited implementations at run-time.
2. The implementation of the subclass becomes bound to the parent class, breaking encapsulation. Almost all changes to the parent class will require changes to the subclass.

{% cite the-gang-of-four -l 19 %}

## Object composition

**Object composition** is an alternative to class inheritance where new functionality is obtained by assembling objects to get more complex functionality {% cite the-gang-of-four -l 18 %}.

Object composition is defined dynamically at run-time by objects acquiring references to other objects. The advantage of this is that objects are accessed entirely through their interfaces, so encapsulation isn't broken {% cite the-gang-of-four -l 19 %}.

One downside of object composition is that it's more difficult to understand dynamic programs than it is to understand static-based programs {% cite the-gang-of-four -l 19 %}.

### Delegation

In **delegation**, two objects are involved in handling a request—a "receiving object delegates operations to its delegate". This is similar to subclasses deferring requests to parent classes {% cite the-gang-of-four -l 20-1 %}.

For example, consider a `Window` class. Instead of making Window inherit from a `Rectangle` class, it could keep a `Rectangle` instance variable and delegate `Rectangle` behavior to its instance variable {% cite the-gang-of-four -l 20-1 %}.

## Designing for change

Systems change. The design of your system determines whether change is easy or difficult.

Design patterns help you avoid expensive changes to existing classes and tests. Design patterns allow some parts of the system to change independently of other parts of the system {% cite the-gang-of-four -l 24 %}.

Some of the common causes of redesign are:

- Creating an object by specifying a class explicitly.
- Dependence on specific operations.
- Dependence on hardware or the software platform.
- Dependence on representations or implementations.
- Algorithmic dependencies.
- Tight coupling.
- Extending functionality by subclassing.
- Inability to change classes yourself (e.g., the class is maintained in a repository you don’t have access to).

{% cite the-gang-of-four -l 24-5 %}

## References

{% bibliography --cited_in_order %}
