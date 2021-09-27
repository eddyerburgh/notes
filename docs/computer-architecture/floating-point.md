---
layout: default
title: Floating point
description: Notes on floating point.
nav_order: 9
parent: Computer architecture
permalink: /computer-architecture/floating-point
---

<!-- prettier-ignore-start -->

# Floating point
{:.no_toc}

## Table of contents
{: .no_toc }

1. TOC
{:toc}

<!-- prettier-ignore-end -->

**Floating point** is an approximation of real-number arithmetic using a finite numbers of bits. IEEE 754 is the floating point standard that's used almost universally.

Floating point represents numbers similarly to scientific notation. A number is represented as:

$$(-1)^\text{sign} \times \text{significand} \times 2^{\text{exponent}}$$ {% cite IEEE754 -l 7 %}.

The **significand** (or mantissa) contains the significant digits {% cite IEEE754 -l 5 %}.

The **exponent** signifies the integer power that 2 is raised to (or 10 in the case of decimal floating point) {% cite IEEE754 -l 4 %}.

The **sign** denotes the sign of the significand.

There are multiple floating point formats used to store the \<sign, significand, exponent\> triple. The common formats are single-precision and double-precision binary.

Each format also contains representations for `+Inifinity`, `-Infinity`, `-0`, `+0`, `qNaN` (quiet NaN), and `sNaN` (signaling NaN) {% cite IEEE754 -l 8 %}.

### Single-precision binary

Single-precision binary representation uses 32 bits to encode a floating point number.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/floating-point/single-precision.svg" alt="">
  <figcaption><h4>Figure: single precision floating point</h4></figcaption>
</figure>

The most significant bit is the _sign bit_ {% cite IEEE754 -l 9 %}.

The 8-bit _biased exponent_ field stores the exponent in biased notation. The bits represent an unsigned integer. The exponent is calculated by subtracting the bias ($$127$$) from the stored value. $$\text{exponent} = \text{biased_exponent} - 127$$ {% cite IEEE754 -l 9,13 %}.

The 23-bit _trailing significand_ field represents the fractional part of the significand (i.e. the part that comes after the binary point). The significand has an inferred integer part of 1, unless the exponent fields are all 0s or all 1s.

For normal numbers, the value of the trailing significand is an unsigned integer where $$\text{significand} = 1 + 2^{-23} \times \text{trailing_significand_value} $$.

Infinity is represented by all bias exponent field bits set to 1 and all trailing significand fields set to 0. The sign bit determines whether it is `+Infinity` or `-Infinity` {% cite IEEE754 -l 9 %}.

`NaN` is represented by all bias exponent field bits as 1 and the trailing significand field being nonzero {% cite IEEE754 -l 9 %}.

Subnormal numbers are represented by 0 exponent and nonzero trailing significand (see [Subnormal numbers](#subnormal-numbers)) {% cite IEEE754 -l 9 %}.

Zero is represented by all 0s, with the sign bit indicating whether it is +0 or -0 {% cite IEEE754 -l 9 %}.

### Double-precision binary representation

Double-precision binary representation uses 64 bits to encode a floating point datum {% cite IEEE754 -l 13 %}.

<figure>
  <img src="{{site.baseurl}}/assets/img/computer-architecture/floating-point/double-precision.svg" alt="">
  <figcaption><h4>Figure: double precision floating point</h4></figcaption>
</figure>

Double precision uses an 11-bit _biased exponent_ field with a bias of 1023 {% cite IEEE754 -l 13 %}.

The _trailing significand_ field is 52 bits.

### Subnormal numbers

**Subnormal numbers** are numbers with a magnitude that is less than the minimum value that can be expressed by a format's exponent. Subnormal numbers follow different encoding rules {% cite IEEE754 -l 8 %}.

For example, in single-precision binary encoding a subnormal number is defined as any number with all bits set to 0 in the exponent and a nonzero value in the trailing significand field. Subnormal single-precision numbers are calculated as $$(-1)^\text{sign} \times 2^{-126} \times 2^{-23} \times \text{trailing_significand_value}$$ {% cite IEEE754 -l 9 %}.

## References

{% bibliography --cited_in_order %}
