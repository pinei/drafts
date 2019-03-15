# Java 9 New Features

Java 9 was released in 2017, more than 3 years after the release of Java 8. 

Here is some of the new features.

## Java 9 REPL (JShell)

A new tool called “jshell” was introduced with the platform. It is used to execute and test any Java constructs and APIs very easily in a interactive way. The shell takes single user inputs, evaluates them and returns the result to the user on display (REPL means read–eval–print loop).

<image for jshell>

Many platforms like Ruby, Python, PHP and Node already feature an interactive REPL for years.

## Interfaces with Private methods

In Java 8, we can provide method implementation in Interfaces using Default and Static methods. However we cannot create private methods in Interfaces.

From Java 9 onwards, we can write private methods in interfaces using private access modifier.

Despite those new features, there are still two main differences between Interface and Classes:
- Interfaces support multiple inheritance
- Classes can have state.

## Improved Javadoc

Javadoc now includes search in the API documentation itself. Its output is now HTML5 compliant.

## Java 9 module system

Modular JAR files now contains an additional module descriptor, which lives in `module-info.java`. Dependencies on other modules are expressed through `requires` statements. Additionally, `exports` statements control which packages are accessible to other modules.

All non-exported packages are encapsulated in the module by default.

Here's an example of a module descriptor, 

```java
module acme-business {
  exports com.acme;
  requires repository;
}
```

The Java platform itself has been modularized using its own module system as well.

When you have modules with explicit dependencies, and a modularized JDK, new possibilities arise. Instead of shipping your app with a fully loaded JDK installation, you can create a minimal runtime image optimized for your application.

That's made possible with the new jlink tool in Java 9.

# References

[9 new features in Java 9 (Pluralsight)](https://www.pluralsight.com/blog/software-development/java-9-new-features)

[Java 9 Features](https://www.journaldev.com/13121/java-9-features-with-examples)

[Java 9 Private methods in Interfaces](https://www.journaldev.com/12850/java-9-private-methods-interfaces)

[Java 9 Interface vs Class](https://stackoverflow.com/questions/44298261/java-9-interface-vs-class)