---
layout: default
title: Compilers
description: An introduction to compilers.
has_children: true
has_toc: false
nav_order: 1
permalink: /compilers
---

<!-- prettier-ignore-start -->

# Compilers
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

A compiler takes a program as input and produces an executable (e.g. assembly code code or bytecode).

A compiler generally has five phases:

1. Lexical analysis
2. Parsing
3. Semantic analysis (e.g. types and scope rules)
4. Optimization
5. Code generation (translation)

These notes cover the five phases.

Many compilers are separated into a frontend and a backend. The frontend typically translates the source program into an IR (intermediate representation), the backend translates IR into target output languages. LLVM uses the frontend/backend concept to enable user-provided frontends to access the LLVM toolchain for optimization and codegen.
