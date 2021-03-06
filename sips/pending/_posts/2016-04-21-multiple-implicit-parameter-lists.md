---
layout: sip
disqus: true
title: SIP-NN - SIP Allow multiple implicit parameter lists
---

**By: Miles Sabin and Merlin Göttlinger**

## History

| Date          | Version       |
|---------------|---------------|
| Apr 21st 2016 | Initial Draft |

## Motivation

In Scala the implicit parameters of a function are limited to being placed in the single very last parameter list of a function. This prevents later parameters from using dependant types of former ones as this is only allowed between, but not inside a single parameter list. Another issue introduced by this restriction is, that if you want to explicitly specify one of the implicit parameters you have to specify all of them by hand (or write `implicitly` in place of every other parameter that should be filled by the compiler).

## Motivating Examples

### Examples

For regular parameter lists it is possible to write the following:
{% highlight scala %}
trait A { type B }

def works(a: A)(b: a.B) = "works"
{% endhighlight %}
Due to the restriction of being contained in a single parameter list the implicits can't benefit from this goodness. A common workaround is to pull the type member `B` into a type parameter of a new `Aux` object and having this as a type parameter of your function:
{% highlight scala %}
trait A { type B }
object A { type Aux[D] = A { type B = D } }

def works[B](implicit a: A.Aux[B], b: B) = "works"
{% endhighlight %}
While this works it introduces unnecessary complexity because our method shouldn't need to have type parameters and we shouldn't need the `Aux` object just to use the type member of `A` for implicits.

With the proposed change of allowing multiple implicit parameter lists the example would again be reduced to the following concise and readable example:
{% highlight scala %}
trait A { type B }

def worksWithImplicits(implicit a: A)(b: a.B) = "pls make me work"
{% endhighlight %}

Another benefit mentioned before would be that implicit parameters, that are expected to often be filled in manually can be put in separate lists.
{% highlight scala %}
def now(implicit a: Int, b: String) = b * a
now(1, implicitly)

def then(implicit a: Int)(b: String) = b * a
then(1)
{% endhighlight %}

## Implementation

The implementation of this change is as simple as removing the artificial restriction of having only one parameter list in the `Parser`. A pull request (#5108) from Miles Sabin already implements this change as well as an accompanying test file. The change should not affect any existing code as every Scala statement that was valid before will still be valid after the change. There will even be binary compatibility in both directions.

## Drawbacks

The current simple implementation may not make it obvious from first glance that parameter lists coming after a implicit list are also implicit.

## References

1. [Miles' PR][1]

[1]: https://github.com/scala/scala/pull/5108 "PR#5108"
