# Scala Best Practices

Best practices for Scala programming.

## Best Practices

- Prefer vars in combination with immutable data structures, instead of vals in combination with mutable data structures.

## Case Class vs. Case Object

A case class can have arguments therefore can have different instances initialized from different arugments. A case object has no argument and can only be a singleton. A case object can be used as enumeration values in pattern matching. It's often used a fixed Actor message.
