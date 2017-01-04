---
layout: post
title: Category theory and abstract algebra on Scala
date: 2017-01-04T12:19:00.000Z
categories: blog
---

This article aims to serve as a gentle introduction to abstract algebra and category theory concepts for engineers, with examples in the Scala programming language. Firstly, we will review some of the major properties in Scala's type system that allow category theory to be implemented. Then we will review the formal definitions of Monad and Functor and its implementations in Scala.<br>

## Introduction

I will start by reviewing on of the features that make Scala's one of the most advanced and powerful type systems. In my humble opinion, this feature should be the starting point for programmers that want to learn category theory, since it by its means that monads and functors can be implemented.<br>
Higher-kinded types is an abstraction over types. As Adriaan Moors clearly explains in [this post][SO01], higher-kinded types are easier to understand if seen as a higher-order polymorphism. This can be shown in the following example:

```scala
// Generic type, T would be the concrete type replaced by the compiler at runtime:
class Container[T] 
// Generic type, where M[_] is a one-argument type constructor:
class Foo[M[_]] 

trait Monad[F[_]] extends FlatMap[F] with Applicative[F] {
  override def map[A, B](fa: F[A])(f: A => B): F[B] =
    flatMap(fa)(a => pure(f(a)))    
```

[SO01]: http://stackoverflow.com/a/6427289/5089400
