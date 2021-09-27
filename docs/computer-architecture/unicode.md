---
layout: default
title: Unicode
description: Notes on the Unicode Standard.
nav_order: 5
parent: Computer architecture
permalink: /computer-architecture/unicode
---

<!-- prettier-ignore-start -->

# Unicode

{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

## Introduction

**Unicode** is a standard for encoding text that supports most of the world's writing systems.

Unicode provides a fundamental set of characters which are each given numeric **code points**. Text elements (like words) are represented using a sequence of one or more code points {% cite unicode-13 -l 1, 7 %}.

Code points are represented as U+PREFIX, where PREFIX is the code point in hexadecimal. For example, U+0061 (a) or U+2260 (≠) {% cite unicode-13 -l 29-30 %}.

**Glyphs** are the shapes that characters can take when rendered and a **font** is a collection of glyphs with a mapping from Unicode characters to glyphs. Unicode itself doesn't specify glyphs, it only deals with abstract characters {% cite unicode-13 -l 15 %}.

Unicode allows dynamic composition where new forms with modifying marks can be created from a combination of base characters followed by combining characters, e.g. ộ can be represented as:

1. U+006f (o) + U+0302 (◌̂) + U+0323 (◌̣)
2. U+006f (o) + U+0323 (◌̣) + U+0302 (◌̂)
3. U+00f4 (ô) + U+0323 (◌̣)
4. U+1ecd (ọ) + U+0302 (◌̂)
5. U+1ed9 (ộ)

The Unicode Standard provides mappings from static precomposed text elements to equivalent dynamically composed sequences of characters. This makes it so that different character sequences can be considered equivalent in string comparisons {% cite unicode-13 -l 23 %}.

A **grapheme cluster** is a text element that users would perceive as a single character (i.e. "any combining character sequence that contains only nonspacing combining marks"). Graphene clusters are intended to be language-independent {% cite unicode-13 -l 60 %}.

As well as defining the abstract characters, Unicode also defines character semantics that include properties like casing, combination, and directionality (there are over 100 different character properties) {% cite unicode-13 -l 18 %}.

The range of code points defined by Unicode (the Unicode codespace) can be divided into 17 planes with each plane consisting of 64K code points. Plane 0 is the Basic Multilingual Plane which contains the common-use characters for all modern-day writing scripts {% cite unicode-13 -l 43 %}.

## Representing unicode

Unicode code points are stored as **code units** {% cite unicode-13 -l 33 %}.

Unicode 13.0 provides three encoding forms that determine how a code point is represented in code units: UTF-8, UTF-16, and UTF-32 {% cite unicode-13 -l 33 %}.

UTF-32 is the simplest form—each unicode code point is represented as a fixed-width 32-bit code unit, but it's also not space-efficient (common ASCII characters only use X of X a single byte) {% cite unicode-13 -l 35 %}.

The most popular encoding on the web is UTF-8.

### UTF-8

UTF-8 is a variable-width encoding. A character can be expressed with either one, two, three, or four bytes {% cite unicode-13 -l 35 %}.

<!-- TODO: More about about how code points are converted to UTF-8 -->

One of the benefits of UTF-8 is that it stores code points 0x00 ... 0x7F so that they are indistinguishable from ASCII {% cite unicode-13 -l 37 %}.

UTF-8 is also more compact than UTF-16 or 32 for representing western languages (although it takes more space than UTF-16 to represent many Asian languages) {% cite unicode-13 -l 37 %}.

## References

{% bibliography --cited_in_order %}
