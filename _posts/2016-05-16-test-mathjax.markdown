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

Text|Rendered expression
\$\$\int_{a}^{b} x^2 dx\$\$|$$\int_{a}^{b} x^2 dx$$

Multiple integrals:

Text|Rendered expression
\$\$\iint_V \mu(u,v) \,du\,dv\$\$|$$\iint_V \mu(u,v) \,du\,dv$$

Text|Rendered expression
\$\$\iiiint_V \mu(t,u,v,w) \,dt\,du\,dv\,dw\$\$|$$\iiiint_V \mu(t,u,v,w) \,dt\,du\,dv\,dw$$


[mathjax-site]: https://www.mathjax.org/
