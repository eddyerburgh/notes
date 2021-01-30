---
layout: default
title: Operating systems
description: An introduction to operating systems.
has_children: true
has_toc: false
nav_order: 6
permalink: /operating-systems
---

<!-- prettier-ignore-start -->

# Operating systems
{:.no_toc}

As a software engineer, most of the code you write will be run by an operating system. Understanding the internals of operating systems will help you write more performant programs.

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

An operating system (OS) is the software that manages a computer’s resources for the computer's users and their applications {% cite oses-principles-and-practice -l 4 %}.

An OS provides:

- Resource allocation
- Isolation
- Communication

An OS is responsible for **resource allocation**. It must allocate finite memory and processors to running applications.

An OS must provide **isolation**. A bug in one program shouldn't bring down the system, and users shouldn't be able to access or change another user's data {% cite oses-principles-and-practice -l 9 %}.

Sometimes isolated programs need to communicate with each other. An OS provides mechanisms for **communication** between running programs {% cite oses-principles-and-practice -l 9 %}.

## References

{% bibliography --cited_in_order %}
