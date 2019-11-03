# Style Guide for Scala Code

**In general, observe the style of existing code and respect it.**

Use [Scalafmt](https://scalameta.org/scalafmt/) in both IDE and sbt build.

## Naming

- Use intention-revealing names which don't need explanation.

  - use `index` instead of `i`

- Constant names should be in upper camel case.
  - use `HotelId` instead of `HOTEL_ID`

## Formating

- Maximal line length is limited to 80 characters.

- Code blocks should not exceed 30 lines.

## `val` & `var`

- Be explicit, use `var` only when really needed.

## `class` & `trait`

- When a class extending a trait,
  only `override` for method with default method,
  do not use `override` for abstract method of the trait.

## `case class` v.s. `class`

- Prefer `final case class` over `class`.
- More at: https://books.underscore.io/essential-scala/essential-scala.html#case-classes
