---
layout: post
title: About sparsal
date: 2017-01-14T12:00:00.000Z
categories: blog
---

Two weeks ago I started coding an implementation of the Matching Pursuit algorithm, by Stéphane Mallat and Zhifeng Zhang.
This led to creating the Sparse Approximation Library - sparsal, for people looking for some of the algorithms, 
implemented in a human-readable, functional style.
<br>
The project lives here: [https://github.com/underscorenico/sparsal](https://github.com/underscorenico/sparsal)

## Overview 
I came across the Matching Pursuit algorithm during an internship at INRIA - Bordeaux. Matching Pursuit is a greedy algorithm that decomposes a signal into "atoms" from an overcomplete dictionary, also called atomic decomposition. It is a valuable tool for dimensionality reduction and feature extraction. One of its extension (an orthogonal version), the Orthogonal Matching Pursuit, is mainly used for generating sparse approximation of signals.
<br>
The goal of sparsal is to provide students and general users an "easy" introduction to sparse approximation methods, writen in a functional style and easy to debug, modify and play with. Every implemented algorithm is written in a "naive"-way, without any attention paid to performance. Nonetheless, optimized implementations of some algorithms (like [MPTK][mptk]) will be available eventually.

[mptk]: http://mptk.irisa.fr/

## Why scala?
Scala is the most complete of the modern programming languages. It is purely object-oriented, functional (functions are first class citizens), with a complete standard library made of mutable and immutable data structures and a syntax suitable for representing mathematical structures and functions. Also, features as pattern matching and higher kinded types allow for new complex types. It comes therefore with no surprise that the chosen language of implementation for sparsal is scala.
<br>
Let's take for instance the discrete dyadic gabor dictionary described in section VI of [this seminal paper][mp], it can be generated in a readable way (without assessing performance issues):

[mp]: https://www.di.ens.fr/~mallat/papiers/MallatPursuit93.pdf


{% highlight scala %}

  private def basis(u: Int, ω: Int, s: Int): Int => Double =
    n => exp(-Pi * pow((n - u).toDouble / s.toDouble, 2)) * sin(2.0 * Pi * (ω.toDouble / N.toDouble) * (n - u).toDouble)

  /**
    * Dyadic sampling of the basis function to generate the dictionary
    */
  override val atoms: RealMatrix = {

    var i: Int = 0
    for {
      j: Int <- 1 until (log(N) / log(2)).toInt
      p: Int <- 0 until (N * pow(2, -j + 1)).toInt
      k: Int <- 0 until (1 * pow(2, j + 1)).toInt
    } yield {
      for {
        n: Int <- 0 until N
      } yield {
        matrix.setEntry(n, i, basis((p * pow(a, j) * Δu).toInt, (k * pow(a, -j) * Δξ).toInt, pow(a, j).toInt)(n))
      }
      // Each atom must have unit norm
      val col: RealVector = matrix.getColumnVector(i)
      matrix.setColumnVector(i, col.mapMultiply(col.getNorm))
      // Update index
      i += 1
    }
    matrix
  }
  
{% endhighlight %} 

We can see that for comprehensions scala's feature is used here to generate the dictionary, where its generators (j, p and k) are implemented in a mathematical-like syntax very similar to as described in the paper.
