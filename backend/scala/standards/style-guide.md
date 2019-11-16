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

## Prefer `final case class` over `class`

## Type Annotation

- Explicitly type-annoted public members

## Error Representation

- `Option` for potential absence of a value
- `Either` for potential failing computatoin
- `Try` to deal with code that throw

## Tools

- scapegoat
- wartremover
- scala linter
- scalastyle
- scalafix
- [Recommended Scalac flags](https://tpolecat.github.io/2017/04/25/scalac-flags.html)
