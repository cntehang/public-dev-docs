# Learn Scala

## Basic Tips

- [Interpret Code Block and Curly Braces](https://www.geekabyte.io/2018/03/an-alternative-way-to-interpret-usage.html)

## Blogs

- [Richard Imaoka -- Akka, Concurrency, etc.](https://richardimaoka.github.io/blog)
  - [Akka HTTP Request and Response models](https://richardimaoka.github.io/blog/akka-http-request-response-model/)

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

- [Reactive Database Mapping with Scala and Slick - Jacek Kunicki](https://www.youtube.com/watch?v=Ksobupg60Vk)
  - example usage of slick

### Doobie

- [Doobie](https://github.com/tpolecat/doobie)

- [Pure Functional Database Programming with Fixpoint Types—Rob Norris](https://www.youtube.com/watch?v=7xSfLPD6tiQ)

## Akka

- [8 Akka Anti Patterns you'd better be aware of](https://www.youtube.com/watch?v=h3mulWmX1Oo)

  - Do not pass mutable reference in message, use immutable message, for async call use "ask and pipe" pattern.
  - Do not always use flat actor hierarchies, use hierarchies and actor model's error handling, let it crash, "error should be handled out of band in a parallel process, they are not part of the main app."
  - Do not use too many actor systems, each actor system has at least one dispatcher backed by a thread pool.
  - Do not log to much, do not use block log, turn debug log off in production, do not log to file.
    - maybe [use elastic to aggregate logs](https://www.elastic.co/products/log-monitoring)
  - Do not use to much vm or docker, be close to hardware, it will be easier to config the dispatcher right.
  - Do not block, do not wait for future.
  - Do not use akka remoting, use akka cluster.
  - Do not use java serialization, use protobuf or Avro.

- [Islands in the Stream Integrating Akka Streams and Akka Actors](https://www.youtube.com/watch?v=qaiwalDyayA)

  - [Blog](https://blog.colinbreck.com/integrating-akka-streams-and-akka-actors-part-iv/)
  - [Code example](https://github.com/pbernet/akka_streams_tutorial)

- [Patterns for Streaming Telemetry with Akka Streams by Colin Breck](https://www.youtube.com/watch?v=ilhImUjF53A)

- [Colin Breck | From Time Series Database to Key Operational Technology for the Enterprise](https://www.youtube.com/watch?v=3APiIht6oDY)

### backpressure

- [Backpressure explained — the resisted flow of data through software, by Jay Phelps](https://medium.com/@jayphelps/backpressure-explained-the-flow-of-data-through-software-2350b3e77ce7)

- The Wikipedia definition:
  Resistance or force opposing the desired flow of fluid through pipes.

- For software:
  Resistance or force opposing the desired flow of data through software.

- handling backpressure:
  - Control the producer (slow down/speed up is decided by consumer)
    - with pull-based streams, the consumer controls the producer
    - with push-based streams, the producer is in control
      and pushes data to the consumer when it’s available.
  - Buffer (accumulate incoming data spikes temporarily)
    - buffering is dangerous if unbounded.
      it is often better to start dropping
      than to fall over completely (run out of memory).
  - Drop (sample a percentage of the incoming data)
  - Ignore the backpressure
    - which, to be honest, is not a bad idea
      if the backpressure isn’t causing critical issues.
      Introducing more complexity comes at a cost too.

## Type System

- [Daniel Beskin at #ScalaUA - Compile Time Logic Programming in Scala](https://www.youtube.com/watch?v=wHrdrRvC1Wg)

  - implicitly searching type is like prolog's searching

    | prolog           | scala type system                                            |
    | ---------------- | ------------------------------------------------------------ |
    | atom and functor | trait with or without type arguments                         |
    | fact             | implicit value                                               |
    | rule             | chained implicit (implicit function with implicit arguments) |

  - read chained implicit backward and think of prolog.
  - Examples:
    - `CanBuildFrom`
    - implicit instances of typeclasses
    - Akka stream graph's `ClosedShape`

- [Existential Types — Make OOP Great Again! by Julien Richard Foy](https://www.youtube.com/watch?v=6j5kZj17aUw)

  - how to do abstraction in type? Parameters vs abstract members.

    ```scala
    trait Group {
      type Element
      def id: Element
      def add(x: Element, y: Element): Element
      def neg(x: Element): Element
    }

    case object RationalGroup extends Group {
      case class Element(numerator: Int, denominator: Int)
      def id: Element = Element(0, 0)
      def add(x: Element, y: Element): Element =
        Element(
          numerator = x.numerator * y.denominator +
            y.numerator * x.denominator,
          denominator = x.denominator * y.denominator)
      def neg(x: Element): Element =
        Element(- x.numerator, x.denominator)
    }
    ```

  - when to use abstract members?
    - abstract members provide a way to make a type opaque from "outside"
    - when you have a lots of wildcard types like `C[_]`
    - finally tagless encoding

### DOT (Dependent Object Type)

- So called "Dependent Object Type" means dependent type restricted to path.

- [DOT: Scala Types from Theory to Practice—Nada Amin](https://www.youtube.com/watch?v=fjj_fv346lY)
  - Scala World, 2017

## Monad

- [Options in Futures, how to unsuck them](https://www.youtube.com/watch?v=hGMndafDcc8)
  - about monad transformer

## Web App

- [Monadic Logging and You - NE Scala 2016](https://www.youtube.com/watch?v=t-YX55ZF4g0)
  - log without sync
  - log less
  - when to log ?

## Scalajs

- [From first principles: Why I bet on Scala.js](http://www.lihaoyi.com/post/FromfirstprinciplesWhyIbetonScalajs.html)
- [Hands-on Scala.js](http://www.lihaoyi.com/hands-on-scala-js/)
- [Youtube video: Full Stack Scala with the Play Framework and Scala.js](https://youtu.be/NJVL2IsGXZ4)
- [Play Framework with Scala.js Template](https://github.com/vmunier/play-scalajs.g8)

## Misc

- [The Last 10 Percent by Stefan Zeiger](https://www.youtube.com/watch?v=RmEMUwfQoSc)

  - about publishing open source scala library, which is infinitely more complicated than npm.
    (I would rather writing scripts to use libraries locally than to use this)
  - scala's npmjs.org is Sonatype,

    - sbt's [docs about using it](https://www.scala-sbt.org/1.x/docs/Using-Sonatype.html)
    - which direct you to [another doc](https://central.sonatype.org/pages/ossrh-guide.html)
      which says:

      ```text
      Sonatype uses JIRA to manage requests.
      - Create your JIRA account
      - Create a New Project ticket
        This triggers creation of your repositories.
        Normally, the process takes less than 2 business days.
      ```
