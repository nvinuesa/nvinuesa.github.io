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

# Overview 
I came across the Matching Pursuit algorithm during an internship at INRIA - Bordeaux. Matching Pursuit is a greedy algorithm that decomposes a signal into "atoms" from an overcomplete dictionary, also called atomic decomposition. It is a valuable tool for dimensionality reduction and feature extraction. One of its extension (an orthogonal version), the Orthogonal Matching Pursuit, is mainly used for generating sparse approximation of signals.
<br>
The goal of sparsal is to provide students and general users an "easy" introduction to sparse approximation methods, writen in a functional style and easy to debug, modify and play with. Every implemented algorithm is written in a "naive"-way, without any attention paid to performance. Nonetheless, optimized implementations of some algorithms (like [MPTK][mptk]) will be available eventually.

[mptk]: http://mptk.irisa.fr/
