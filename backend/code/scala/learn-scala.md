# Learn Scala

## Youtube Channels

- [Scala Days Conferences](https://www.youtube.com/channel/UCOHg8YCiyMVRRxb3mJT_0Mg)

## Courses

- ["Monad And All That", in scala](https://github.com/xieyuheng/pracat/blob/master/docs/monad-and-all-that.md)

## Introduction

- [Essential Scala: Six Core Principles for Learning Scala](https://www.youtube.com/watch?v=J8wUy1XxL5o)
  - It is essential to program like keyboard cat!
  - discuss six fundamental concepts that underly effective Scala; How can programmers quickly and effectively learn to write idiomatic Scala?
    - Expressions, types, and values
    - Objects and classes
    - Algebraic data types
    - Structural recursion
    - Sequencing computations
    - Type classes
  - Use type class, when you need common behavior, but do not have (useful) common type.
    (to achieve polymorphism without class hierarchy)
    (so called ad-hoc polymorphism)

## Martin Odersky

- [FOSDEM 2009 Scala - A Scalable Language](https://www.youtube.com/watch?v=zqFryHC018k)
  - In this talk the author describes the design principles of the Scala programming language,
    which has scalability as its primary design objective.

- ["Working Hard to Keep It Simple" - OSCON Java 2011](https://www.youtube.com/watch?v=3jg1AheF4n0)
  - Why we need functional programming and actor model
  - Object oriented programming is as important as functional programming

- [Plain Functional Programming, 2017, at Devoxx](https://www.youtube.com/watch?v=YXDm3WHZT5g)
  - Kleisli triples v.s. implicit function type
  - with a reference to [Strategic Scala Style: Principle of Least Power](http://www.lihaoyi.com/post/StrategicScalaStylePrincipleofLeastPower.html)

- [From DOT to Dotty by Martin Odersky](https://www.youtube.com/watch?v=iobC5yGRWoo)
  - About the theoretic foundation of Scala, i.e. DOT (with dependent type restricted to path)
  - implicit function type in dotty (scala 3.x)

- [Keynote - What to Leave Implicit by Martin Odersky](https://www.youtube.com/watch?v=Oij5V7LQJsA)
  - Scala Days Chicago 2017

- [ScalaUA 2019 - Video Q&A session with Martin Odersky, Creator of Scala, EPFL, Lightbend](https://www.youtube.com/watch?v=wm2DhYrZVno)

## Database

### Slick

- Slick: https://github.com/slick/slick

- [Polymorphic record types in a lifted embedding - by Stefan Zeiger](https://www.youtube.com/watch?v=tS6N5AaZTLA)
  - Scala Days New York, 2016
  - an overview of the "lifted embedding" at the core of the Scala DSL in Slick.
  - "lifted embedding" is a tech to enable type check of target language in scala's type system
    - "embedding" means designing AST and writing interpreter in scala
    - "lifted" means `T` to `F[T]`

### Doobie

- Doobie: https://github.com/tpolecat/doobie

- [Pure Functional Database Programming with Fixpoint Types—Rob Norris](https://www.youtube.com/watch?v=7xSfLPD6tiQ)

## DOT (Dependent Object Type)

- So called "Dependent Object Type" means dependent type restricted to path.

- [DOT: Scala Types from Theory to Practice—Nada Amin](https://www.youtube.com/watch?v=fjj_fv346lY)
  - Scala World, 2017

## Type System

- [Daniel Beskin at #ScalaUA - Compile Time Logic Programming in Scala](https://www.youtube.com/watch?v=wHrdrRvC1Wg)
  - implicitly searching type is like prolog's searching

    | prolog           | scala type system                                            |
    |------------------|--------------------------------------------------------------|
    | atom and functor | trait with or without type arguments                         |
    | fact             | implicit value                                               |
    | rule             | chained implicit (implicit function with implicit arguments) |

  - read chained implicit backward and think of prolog.
  - Examples:
    - `CanBuildFrom`
    - implicit instances of typeclasses
    - Akka stream graph's `ClosedShape`

## Web App

- [Monadic Logging and You - NE Scala 2016](https://www.youtube.com/watch?v=t-YX55ZF4g0)
  - log without sync
  - log less
  - when to log ?

## Misc

- [The Last 10 Percent by Stefan Zeiger](https://www.youtube.com/watch?v=RmEMUwfQoSc)
  - about publishing open source scala library, which is infinitely more complicated than npm.
    (I would rather writing scripts to use libraries locally than to use this)
  - scala's npmjs.org is Sonatype,
    - sbt's docs about using it: https://www.scala-sbt.org/1.x/docs/Using-Sonatype.html
    - which direct you to another docs at: https://central.sonatype.org/pages/ossrh-guide.html
      which says:
      ```
      Sonatype uses JIRA to manage requests.
      - Create your JIRA account
      - Create a New Project ticket
        This triggers creation of your repositories.
        Normally, the process takes less than 2 business days.
      ```
