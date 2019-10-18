# Learn Scala

## Basic Tips

- [Interpret Code Block and Curly Braces](https://www.geekabyte.io/2018/03/an-alternative-way-to-interpret-usage.html)

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
- [Options in Futures, how to unsuck them](https://www.youtube.com/watch?v=hGMndafDcc8): about monad transformer

## Some Resources

### Blogs

- [Richard Imaoka -- Akka, Concurrency, etc.](https://richardimaoka.github.io/blog)
- [Akka HTTP Request and Response models](https://richardimaoka.github.io/blog/akka-http-request-response-model/)

### Youtube Channels

- [Scala Days Conferences](https://www.youtube.com/channel/UCOHg8YCiyMVRRxb3mJT_0Mg)

### Martin Odersky

- [A Scalable Language - FOSDEM 2009 Scala](https://www.youtube.com/watch?v=zqFryHC018k): In this talk the author describes the design principles of the Scala programming language, which has scalability as its primary design objective.
- [Working Hard to Keep It Simple - OSCON Java 2011](https://www.youtube.com/watch?v=3jg1AheF4n0):
  - Why we need functional programming and actor model
  - Object oriented programming is as important as functional programming
- [What to Leave Implicit - ScalaDay 2017](https://www.youtube.com/watch?v=Oij5V7LQJsA): use implict correctly.
- [Plain Functional Programming - 2017](https://www.youtube.com/watch?v=YXDm3WHZT5g): with a reference to [Strategic Scala Style: Principle of Least Power](http://www.lihaoyi.com/post/StrategicScalaStylePrincipleofLeastPower.html)
- [From DOT to Dotty by Martin Odersky - VoxxedDays 2017](https://www.youtube.com/watch?v=iobC5yGRWoo): About the theoretic foundation of Scala, i.e. DOT (with dependent type restricted to path), implicit function type in dotty (scala 3.x).
- [Video Q&A session with Martin Odersky- ScalaUA 2019](https://www.youtube.com/watch?v=wm2DhYrZVno)

## Slick

- [Slick](https://github.com/slick/slick)

- [Polymorphic record types in a lifted embedding - by Stefan Zeiger](https://www.youtube.com/watch?v=tS6N5AaZTLA)

  - Scala Days New York, 2016
  - an overview of the "lifted embedding" at the core of the Scala DSL in Slick.
  - "lifted embedding" is a tech to enable type check of target language in scala's type system
    - "embedding" means designing AST and writing interpreter in scala
    - "lifted" means `T` to `F[T]`

- [Reactive Database Mapping with Scala and Slick - Jacek Kunicki](https://www.youtube.com/watch?v=Ksobupg60Vk)
  - example usage of slick

## Akka

- [Programming Reactive Systems](https://courses.edx.org/courses/course-v1:EPFLx+scala-reactiveX+2T2019/course/): free edX Akka Actor and Streaming course.
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

## Logging

- [Monadic Logging and You - NE Scala 2016](https://www.youtube.com/watch?v=t-YX55ZF4g0)

## Scalajs

- [From first principles: Why I bet on Scala.js](http://www.lihaoyi.com/post/FromfirstprinciplesWhyIbetonScalajs.html)
- [Hands-on Scala.js](http://www.lihaoyi.com/hands-on-scala-js/)
- [Youtube video: Full Stack Scala with the Play Framework and Scala.js](https://youtu.be/NJVL2IsGXZ4)
- [Play Framework with Scala.js Template](https://github.com/vmunier/play-scalajs.g8)
