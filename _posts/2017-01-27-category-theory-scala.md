---
layout: post
title: Category theory and abstract algebra on Scala
date: 2017-01-27T12:19:00.000Z
categories: blog
---

This article aims to serve as a gentle introduction to abstract algebra and category theory concepts for engineers, with examples in the Scala programming language. Firstly, we will review some of the major properties in Scala's type system that allow category theory to be implemented. Then we will review the formal definitions of Monad and Functor and its implementations in Scala.<br>

## Introduction

I will start by reviewing on of the features that make Scala's one of the most advanced and powerful type systems. In my humble opinion, this feature should be the starting point for programmers that want to learn category theory, since it by its means that monads and functors can be implemented.<br>
Higher-kinded types is an abstraction over types. As Adriaan Moors clearly explains in [this post][SO01], higher-kinded types are easier to understand if seen as a higher-order polymorphism. This can be shown in the following example:


{% highlight scala %} 
// Generic type, T would be the concrete type replaced by the compiler at runtime:
class Container[T] 
// Generic type, where F[_] is a one-argument type constructor:
class Foo[F[_]]
// Generic type, where F[_,_] is a two argument type constructor:
class Bar[F[_,_]]
{% endhighlight %} 

The second and third classes illustrate its purpose. The class Foo takes as parameter a type constructor taking one parameter, while Bar takes as parameter a type constructor taking two parameters.
<br>
This (seemingly) simple feature allows us to develop powerful abstractions (not only over types, as classic java polymorphism, but over type constructors), and gives birth to advanced and complex abstract algebra libraries that we will cover in the next sessions.
<br>
More information about Higher-kinded types can be found in [this twitter blog][TW01] and in this wonderful [TypeLevel post][TY01].

[SO01]: http://stackoverflow.com/a/6427289/5089400
[TW01]: https://twitter.github.io/scala_school/advanced-types.html#higher
[TY01]: http://typelevel.org/blog/2016/08/21/hkts-moving-forward.html

## Category

Before going any further, lets review the formal definition of category.
<br>
A category  $$ \mathcal C $$ consists of:
<br>

* a set of objects $$ ob(\mathcal C) $$
* for each pair $$ X,Y \in ob(\mathcal C) $$ a set of morphisms (or arrows, or maps) $$ hom(\mathcal C) $$ or $$ hom_c(X,Y) $$
* for each triple $$ X,Y,Z \in ob(\mathcal C)$$ a binary operation $$ hom(X,Y) \times hom(Y,Z) \rightarrow hom(X,Z) $$ noted as $$ f,g \rightarrow f \circ g $$, also referred as composition of morphisms

subject to the following conditions:

* composition of morphisms is associative: $$ (f \circ g) \circ h = f \circ (g \circ h) $$
* for every element $$ X \in ob(\mathcal C) $$ there exists a morphism $$ iD_x \in hom(\mathcal C) $$ such that $$ iD_x \circ f = f $$ and $$ g \circ iD_x = g $$

In simple words, in software engineering we could see categories as a way to represent objects and the relations that link them (the morphisms).

## Functor

But what about the mappings (or relations, or morphisms) that link one category to another?
<br>
This crazy idea would kinda like if we wanted to have a second degree of generics ;)
<br>
Well, actually Scala allows this, and calling it "generics of a higher kind" would not be a huge mistake. As presented in the introduction, we can have higher kinded types that could mimic the relationship between categories.
<br>
Firstly, lets review the formal definition of a functor.
<br>
<br>
Given categories $$ \mathcal C $$ and $$ \mathcal D $$, a functor $$ \mathcal F $$ from $$ \mathcal C $$ to $$ \mathcal D $$:

* maps each element $$ X \in ob(\mathcal C) $$ to an element $$ \mathcal F(X) $$ in $$ \mathcal D $$
* maps each morphism $$ f: X \rightarrow Y $$ in $$ \mathcal C $$ to a morphism $$ \mathcal F(f): \mathcal F(X) \rightarrow \mathcal F(Y) $$ in $$ \mathcal D $$

subject to:

* $$ \mathcal F(id_X) = id_{F(X)} $$ for every element $$ X \in ob(\mathcal C) $$
* $$ \mathcal F(f \circ g) = \mathcal F(f)  \circ \mathcal F(g) $$ for all morphisms $$ f: X \rightarrow Y $$ and $$ g: Y \rightarrow Z $$ in $$ \mathcal C $$

For us, software engineers, this means that a functor is just a morphism between categories, also called structures preserving map. Basically, a functor would just define a ```map``` method.
<br>
This is the functor definition in [scalaz][scalaz]:

{% highlight scala %}
trait Functor[F[_]] extends InvariantFunctor[F] { self =>

  /** Lift `f` into `F` and apply to `F[A]`. */
  def map[A, B](fa: F[A])(f: A => B): F[B]
  ...
}
{% endhighlight %} 

and in [cats][cats]:
{% highlight scala %}
/**
  * Functor.
  *
  * The name is short for "covariant functor".
  *
  * Must obey the laws defined in cats.laws.FunctorLaws.
  */
@typeclass trait Functor[F[_]] extends functor.Invariant[F] { self =>
  def map[A, B](fa: F[A])(f: A => B): F[B]
}
{% endhighlight %} 

[scalaz]: https://github.com/scalaz/scalaz
[cats]: https://github.com/typelevel/cats

## Monad

Similar to the functor, the monad is an abstraction for an operation. But instead of the map, it is an abstraction of the flatMap operation.
<br>
Havin this abstraction is of a huge importance. Since *monadic* structures provide us with the ability to wrap context information, and eventually return a subset of the parent type.
<br>
Scala example:

{% highlight scala %}
trait Monad[M[_]] {
  def flatMap[A, B](fa: M[A])(f: A => M[B]): M[B]
  def unit[A](a: => A): M[A]
}
{% endhighlight %} 

As shown above, the ```flatMap``` operation returns a subset of the parent type M[_]: M[B].
<br>
The importance of this category relies also in that many of the data structures we are used to in functional programming have *monadic* properties; like ```Option```, ```Future``` and ```Try```.

## Conclusions

We have reviewed some of the most basic but also most important concepts in abstract algebra used in category theory. 
<br>
These mathematical frameworks provide us (engineers) with useful tools in our continuously pursuit of abstraction, factorization and readability in code.
<br>
