---
layout: post
title: Playing with MathJax
date: 2016-05-16T17:22:01.000Z
categories: blog
---

In this post I will be playing with some of the amazing features proposed by [MathJax][mathjax-site].

Pixyll theme uses kramdown markdown by default, and kramdown supports LaTeX expressions and renders them using MathJax. Therefore inserting LaTeX code will be automatically rendered.
Following a list of examples:

# Integrals

Example rendering a quite simple integral:

$$\int_{a}^{b} x^2 dx$$

Multiple integrals:

$$\iint_V \mu(u,v) \,du\,dv$$

$$\iiiint_V \mu(t,u,v,w) \,dt\,du\,dv\,dw$$

Contour integrals:

$$\oiint_V f(s,t) \,ds\,dt$$

$$\ointctrclockwise_V f(u) \,du$$


[mathjax-site]: https://www.mathjax.org/
