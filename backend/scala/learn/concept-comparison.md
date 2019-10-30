# Concept Comparison

## 1 Scala vs Java’s Type Hierarchy

Although they are written with initial capitals, Scala’s Int, Double, Float, Long, Short, Byte, Boolen and Char refer to exactly the same things as int, double, float, long, short, byte, boolean, and char in Java.

In Scala all of these types act like objects with methods and fields. However, once your code is compiled, a Scala Int is exactly the same as a Java int. This makes interoperability between the two languages a breeze.

Scala has the means to define optional values that are checked by the compiler. This removes the necessity of using null, making our programs much safer.

## 2 Literal Objects vs Object Literal

`Literal Objects`: A literal expression represents a fixed value that stands “for itself”.

- 42
- true
- 'a'
- "a"
- null

`Object Litreal`: An `Object Litreal` is build with declaration

```scala
object Test {}
```

## 3 Methods vs Fields

The defference between them is subtle:

- a field gives a name to a value,
- a method gives a name to a computation that produces a value

When an object is first loaded, Scala runs through its definitions and calculates the values of each of its fields. The body of a method, on the other hand, is evaluated every time we call the method.

## 4 Declarations vs Expressions

- Declarations are different to expressions.
- Declarations bind names to values
- Expressions evaluate to values

## 5 Conditionals vs Expressions

Scala’s if statement has the same syntax as Java’s. One important difference is

- Scala’s conditional is an expression—it has a type and returns a value.

## 6 Object vs Any

- `Any` is the grand supertype of all Scala types, including `AnyVal` and `AnyRef`
- `java.lang.Object` is equal to `AnyRef` as supertype of all `Scala`, `Java` classes, as well as array, but `AnyVal` like Int and Boolean are not included

## 7 Methods vs Functions

A function is like a method:

- we can call it with parameters and it evaluates to a result

Unlike a method:

- a function is value. We can pass a function to a method or to another function. We can return a func􏰀on from a method, and so on.

## 8 `var` vs `val`

- `val` fields are immutable—they are initialized once after which we cannot change their values
- `var` is for defining mutable fields

## 9 `==` vs `eq`

- `==` operator is different from Java’s—it delegates to equals rather than comparing values on reference identity.
- `eq` is with the same behaviour as Java’s `==`. However, it is rarely used in application code

## 10 Traits vs Java Interfaces

Traits are very much like Java 8’s interfaces with `default methods`. If you have not used Java 8, you can think of traits as being like a cross between interfaces and abstract classes.

## 11 Traits vs Classes

Like a class, a trait is a named set of field and method definitions. However, it differs from a class in a few important ways:

**A trait cannot have a constructor**--we can’t create objects directly from a trait. Instead we can use a trait to create a class, and then create objects from that class

**Traits can define abstract methods that have names and type signatures but no implementation.**

## 12 Case Object vs Object

Here's one difference - case objects extend the Serializable trait, so they can be serialized. Regular objects cannot by default.

- [SO Thread: difference-between-case-object-and-object](https://stackoverflow.com/questions/5270752/difference-between-case-object-and-object)

## 13 Generics vs Traits

Inheritance-based approaches—traits and classes—allow us to create permanent data structures with specific types and names. We can name every field and method and implement use-case-specific code in each class. Inheritance is therefore better suited to modelling significant aspects of our programs that are reused in many areas of our codebase.

Generic data structures—Tuples, Options, Eithers, and so on—are extremely broad and general purpose. There are a wide range of predefined classes in the Scala standard library that we can use to quickly model relationships between data in our code. These classes are therefore better suited to quick, one-off pieces of data manipulation where defining our own types would introduce unnecessary verbosity to our codebase.

## 14 Abstract Data Type vs Algebraic Data type

Refer to this [so thread](https://stackoverflow.com/questions/42833270/what-is-between-abstract-data-types-and-algebraic-data-types):

> They really talk about fundamentally different things. Some data structures can actually be seen as abstract data types implemented as algebraic data types! Like the commom linked list abstract data type in Prelude, which is implemented as the [] algebraic data type. the interface it offers is consing and unconsing, but it's implemented as multiple constructors with one having multiple fields -- sums and products.

## 15 Import Statements in Scala vs Others

The concept of importing the code is more flexible in scala than Java or C++.

- The import statements can be anywhere. It could be at the start of class, with in a class or object or with in method or block.
- Members can be renamed or hidden while importing.
- Packages, Classes or objects can be imported into the current scope.

Refer to [Scala/Import](https://en.wikibooks.org/wiki/Scala/Import).
