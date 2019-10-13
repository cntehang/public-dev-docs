# Scala Best Practices

Best practices for Scala programming. Unless you have a strong reason and get approval from an experienced developer, please just follow these practices.

## 1. Subtype and Type Class

Subtypes (or subclasses in Java) are only in one well-defined scenario: to define [sum types](https://alvinalexander.com/scala/fp-book/algebraic-data-types-adts-in-scala). If all subtypes are knonwn, use `sealed trait` or `seatled abstract class` as the base type and use `final case object/class` as the sum types.

For other polymorphism scnearios, don't use subtype, i.e, don't use type hierarchy, use the type class pattern as the following for Scala 2 (Scala 3 has better syntax):

```scala
// the type class
trait Showable[T] {
    def show(t: T): String
}

// the companion object defines the type class interface that is used to call any method of the type class.
object Showable {
    def apply[T](implicit showable: Showable[T]) = showable
}

// the type enrichment class. The naming convention is to add `Ops` postfix.
// It extends AnyVal to minimize the overhead
implicit class ShowableOps[T](val t: T) extends AnyVal {
    define show(implicit showable: Showable[T]): String = showable.show(t)
}

// define an instance for each concrete type SomeType
implict val myInstance: Showable[SomeType] = new Showable[SomeType] {
    def show(t: SomeType) = ...
}

// To use the interface pattern when type enrichment class is not defined
Showable[SomeType].show(instance)

// With type enrichment
instance.show()
```

References:

- This 2019 video of [Ploymorphism in Scala](https://scaladays.org/2019/lausanne/schedule/polymorphism-in-scala) gives a clear introdution to three different polymorphism (ad hoc, parametric, inclusive) and relavant concepts such as type variances, context bound and the design patterns (interface syntax and type enrichment) of type classes.
- The blog of [Returning the "Current" Type in Scala](https://tpolecat.github.io/2015/04/29/f-bounds.html) explains the related concepts (F-bounded types, self-type, `forSome` keyword).
