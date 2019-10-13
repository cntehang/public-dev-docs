# Scala Best Practices

Best practices for Scala programming. Unless you have a strong reason and get approval from an experienced developer, please just follow these practices.

## 1. 不使用面向对象编程的类继承层级

除了定义 sum types 这一特殊场景，不要在自己的代码用定义类的继承层级。需要使用多态的地方请用 type class。

Only use subtypes (or subclasses in Java) in one well-defined scenario: to define [sum types](https://alvinalexander.com/scala/fp-book/algebraic-data-types-adts-in-scala). 虽然用的是类或对象的继承关系，其实只是用仅有一层的继承语法定义了数据的类型 （is-a 关系 ），相关的数据操作，通常定义到相应的 companion object 里面。

For other polymorphism scnearios, don't use subtype, i.e, don't use type hierarchy, use the type class pattern as the following for Scala 2. Scala 3 has better syntax.

```scala
// Step 1: use a trait to define a type class
// the type class trait defines all abstract methods
trait Named[T] {
  def name(a:T): String
  def addTitle(a: T, title: String): T
}

// Step 2: define the the companion object of the type class.
// It is used to call any method of the type class.
object Named {
  def apply[T](implicit instance: Named[T]) = instance
}

// Step 3: define the type enrichment class.
// The naming convention is to add `Ops` postfix.
// It extends AnyVal to minimize the overhead
implicit class NamedOps[T](val t: T) extends AnyVal {
  def name(implicit named: Named[T])  = named.name(t)
  def addTitle(title: String)(implicit named: Named[T]) = named.addTitle(t, title)
}

// To use the type class,define an implicit instance for each concrete type
case class Cat(name: String, age: Int)
object Cat {
  implicit val NamedCat = new Named[Cat] {
    def name(a: Cat) = a.name
    def addTitle(a: Cat, title: String) = a.copy(name = title + a.name)
  }
}

// Usage example 1: use the interface pattern when type enrichment class is not defined
val tom = Cat("Tom", 2)
println(Named[Cat].name(tom))

// Usage example 2: use with type enrichment
println(tom.name)

// Usage example 3: use context bound [T: Named]
// a type T should have an implicit instance of Named[T]
def showDear[T: Named](t: T) = t.addTitle("Dear ").name
println(showDear(tom))
```

References:

- This 2019 video of [Ploymorphism in Scala](https://scaladays.org/2019/lausanne/schedule/polymorphism-in-scala) gives a clear introdution to three different polymorphism (ad hoc, parametric, inclusive) and relavant concepts such as type variances, context bound and the design patterns (interface syntax and type enrichment) of type classes.
- The blog of [Returning the "Current" Type in Scala](https://tpolecat.github.io/2015/04/29/f-bounds.html) explains the related concepts (F-bounded types, self-type, `forSome` keyword).

## 2. 尽量使用函数式编程，尽量避免类（class）

尽量避免面向对象的 class 而尽可能采用函数式编程。用 case class 或 case object 来定义数据，用 object 或 compnion object 定义操作。
