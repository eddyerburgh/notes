---
layout: default
title: Lexical analysis
description: Notes on lexical analysis.
has_children: true
has_toc: false
nav_order: 1
parent: Compilers
permalink: /compilers/lexical-analysis
---

<!-- prettier-ignore-start -->

# Lexical analysis
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

Lexical analysis (lexing) is the process of converting a sequence of characters into a sequence of tokens. A program that performs lexing is known as a lexer. Lexical analysis is normally the first phase of a compiler.

A **token** consists of a token name and an optional value, e.g. \<identifier, x\> {% cite the-dragon-book -l 111 %}.

A **lexeme** is a sequence of characters in the input source that matches the pattern for a token and is identified as an instance of that token {% cite the-dragon-book -l 111 %}.

A **pattern** is a description of the form that lexemes can take.

Common token names include:

1. Keywords
2. Operators
3. Identifiers
4. Literals
5. Separators

Usually a separate symbol table is maintained by the lexer which contains additional information about identifiers (such as type, line location, etc.) {% cite the-dragon-book -l 112 %}.

## Formal languages

A formal language is a set of strings drawn from an **alphabet** (usually represented as $$\Sigma$$), which is a finite set of characters {% cite the-dragon-book -l 117 %}.

A meaning function $$L$$ maps syntax to semantics. The $$L$$ mapping is many-to-one, which enables optimizations without changing the semantics of the language.

The empty string (denoted $$\epsilon$$) is a string of length 0.

## Regular expressions

A regular expression is a sequence of characters that defines a search pattern. Regular expressions can be used for specifying lexeme patterns {% cite the-dragon-book -l 116 %}.

Regular expressions consist of constants, which denote sets of strings, and operator symbols, which denote operations.

Common operations include:

1. Alternation ($$A \mid B = \{\, c \mid c \in A \vee c \in B \,\}$$)
2. Concatenation ($$A B = \{\, cd \mid c \in A \wedge d \in B \,\}$$)
3. Kleene closure ($$A^* = \cup_{i=0}^{\infty} A^i$$)
4. Positive closure($$A^+ = \cup_{i=1}^{\infty} A^i $$)

A **regular set** is a language that can be defined by a regular expression {% cite the-dragon-book -l 122 %}.

## Lexical grammar

A **lexical grammar** is a set of rules defining the syntax of tokens.

In the case that a substring matches multiple patterns, the following conventions are followed:

1. Maximal match: if there are two possible substrings, take the longer one.
2. Rule priority: Regular expressions identifying tokens are written down in sequence. If two regular expressions match the longest string, the first regular expression in the sequence takes precedence.

## Finite automata

Finite automata (or finite state machines) are abstract machines that can be in exactly one state at a time. They can be used to determine matches for a regular expression.

Finite automata can be represented as diagrams of nodes, where each node represents a state.

<figure>
  <img src="{{site.baseurl}}/assets/img/compilers/lexical-analysis/finite-automata.svg" alt="">
  <figcaption><h4>Figure: Finite automata that accepts 1 or 70</h4></figcaption>
</figure>

They can also be represented with a transition table, where the rows correspond to a state and the columns correspond to the input symbols and their associated next state {% cite the-dragon-book -l 148 %}.

A transition between two states is known as a move.

The input pointer only advances in finite automata, although in an epsilon move a transition can occur without advancing the input pointer.

A finite automata can be defined to accept a string $$s$$ iff there is a path in the transition graph from the start state to a final state so that the symbols along the path spell out $$s$$ {% cite the-dragon-book -l 149 %}.

There are two types of finite automata: nondeterministic and deterministic.

NFAs (Nondeterministic Finite Automata) have no restrictions on their edges. A symbol label can have several edges, and the empty string epsilon is a supported edge label {% cite the-dragon-book -l 147 %}.

NFAs consists of:

1. A finite set of states $$S$$.
2. An input alphabet $$\Sigma$$.
3. A transition function that gives for each state and each symbol in $$\Sigma \cup \epsilon$$ a set of next states.
4. A start state $$s_0$$ in $$S$$.
5. A set of states $$F$$ in $$S$$ which are final states.

{% cite the-dragon-book -l 147 %}

A DFA (Deterministic Finite Automata) is a special case of an NFA with the following conditions:

1. There are no moves for an empty string input.
2. There is only one edge per input symbol for a given state .

{% cite the-dragon-book -l 151 %}

A regex pattern can be converted to an NFA. An NFA can then be converted to a DFA, which is simpler to implement and computationally less expensive to run (although it often requires more space) {% cite the-dragon-book -l 150 %}.

## References

{% bibliography --cited_in_order %}
