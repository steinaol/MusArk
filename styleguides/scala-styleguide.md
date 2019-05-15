# MUSIT-Norway Scala Guide

Code is written once by its author, but read and modified multiple times by lots of other engineers. As most bugs actually come from future modification of the code, we need to optimize our codebase for long-term, global readability and maintainability. The best way to achieve this is by writing simple code.

## Guidelines

The MUSIT backend code base is formatted using the excellent [scalafmt](http://scalameta.org/scalafmt/) SBT plugin. It handles automatic formatting of all the Scala code, including the SBT build definitions. As such, there is not much to add to the guidelines from a code formatting perspective. To see the formatting rules used see the [.scalafmt.conf](https://gitlab.com/MUSIT-Norway/musit/blob/master/.scalafmt.conf) file in the `MUSIT-Norway/musit` repository.

Below follows a short list of guidelines that are not enforced automatically.

### Parantheses

* Methods should be declared with parentheses, unless they are accessors that have no side-effect (state mutation, I/O operations are considered side-effects).

  ```scala
  class Job {
    // Wrong: killJob changes state. Should have ().
    def killJob: Unit

    // Correct:
    def killJob(): Unit
  }
  ```

* Callsite should follow method declaration, i.e. if a method is declared with parentheses, call with parentheses. Note that this is not just syntactic. It can affect correctness when apply is defined in the return object.

  ```scala
  class Foo {
    def apply(args: String*): Int
  }

  class Bar {
    def foo: Foo
  }

  new Bar().foo    // This returns a Foo
  new Bar().foo()  // This returns an Int!
  ```

### Long literals
Suffix long literal values with uppercase L. It is often hard to differentiate lowercase l from 1.

```scala
val longValue = 5432L  // Do this

val longValue = 5432l  // Do NOT do this
```

### Documentation Style
Use Java docs style instead of Scala docs style.

```scala
Correct:
/**
 * This is correct JavaDoc comment. And
 * this is my second line, and if I keep typing, this would be
 * my third line.
 */
 
 Wrong:
 /** We don't use one liner comments like this one. */
 
 Wrong:
 /** In MUSIT, we don't use the ScalaDoc style so this
  * is not correct.
  */
```

### Imports

* Avoid using wildcard imports, unless you are importing more than 4 entities, or implicit methods. Wildcard imports make the code less robust to external changes.
* Always import packages using absolute paths (e.g. scala.util.Random) instead of relative ones (e.g. util.Random).

### Pattern matching

* For method whose entire body is a pattern match expression, put the match on the same line as the method declaration if possible to reduce one level of indentation.

  ```scala
  def test(msg: Message): Unit = msg match {
    case ...
  }
  ```

* When calling a function with a closure (or partial function), if there are multiple cases, indent and wrap them.

  ```scala
  list.map {
    case a: Foo =>  ...
    case b: Bar =>  ...
  }
  ```

* Unless _all_ cases fit within the max line length, each case body should be on a new line. Each case should then be separated with a blank line.

  ```scala
  foobar match {
    case a: Foo =>					// new line here
      // The first case body that is
      // either very long or has
      // several statements
    									// blank line here
    case b: Bar =>
      // The second case body 
  }
  ```

### Naming

We try to follow the standard Scala/Java naming conventions.

* Classes, traits, objects and enums should follow the Java class naming convention.

  ```scala
  class FooBar

  trait Money

  object MyObject
  ```

* Packages should follow the standard Java naming conventions. All lowercase ASCII letters.

  ```scala
  package no.uio.musit.service
  ```

* Methods and functions should be named using **camelCase**.

* Annotations should follow the Scala convention. _NOT_ the Java convention.

* Variables should be defined in **camelCase** style, and should have meaningful names.

  ```scala
  val timeout = 10 seconds
  val currUsr = "Darth Vader"
  ```

* In local scopes it is OK to use short (and 1 character) names.

  ```scala
  // These are all OK
  maybeString.foreach(s => println(s))
  
  maybeString.foreach(println)

  maybeInt.map(n => n * 2)
  ```

* It is OK to use unamed arguments in local scopes.

  ```scala
  maybeInt.map(_ * 2)
  ```

### Line length

* Try to keep lines below 80 characters in length.
* Max limit of lines is set to 90 characters. Anything longer than that will only be acceptable on a case by case basis.

### Methods

* Methods should not be longer than 50 lines of code.
* A class should contain no more than 30 methods/functions.

### Anonymous methods

Avoid excessive parentheses and curly braces for anonymous methods.

```scala
// Correct
list.map { item =>
  ...
}

// Correct
list.map(item => ...)

// Wrong
list.map(item => {
  ...
})

// Wrong
list.map { item => {
  ...
}}

// Wrong
list.map({ item => ... })
```

## Scala langugage features

### The apply method

Avoid defining apply methods on classes. These methods tend to make the code less readable, especially for people less familiar with Scala. It is also harder for IDEs (or grep) to trace. In the worst case, it can also affect correctness of the code in surprising ways, as demonstrated in Parentheses.

It is acceptable to define apply methods on companion objects as factory methods. In these cases, the apply method should return the companion class type.

```scala
object TreeNode {
  // This is OK
  def apply(name: String): TreeNode = ...

  // This is bad because it does not return a TreeNode
  def apply(name: String): String = ...
}
```

### The override modifier

Always add the `override` modifier for methods, both for overriding concrete methods and implementing abstract methods. The Scala compiler does not require override for implementing abstract methods. However, we should always add `override` to make it obvious, and to avoid accidental non-overrides due to non-matching signatures.

```scala
trait Parent {
  def hello(data: Map[String, String]): Unit = {
    print(data)
  }
}

class Child extends Parent {
  import scala.collection.Map

  // The following method does NOT override Parent.hello,
  // because the two Maps have different types.
  // If we added "override" modifier, the compiler would've caught it.
  def hello(data: Map[String, String]): Unit = {
    print("This is supposed to override the parent method, but it is actually not!")
  }
}
```

### Symbolic methods (operator overloading)

Do NOT use symbolic method names, unless you are defining them for natural arithmetic operations (e.g. +, -, *, /). Under no other circumstances should they be used. Symbolic method names make it very hard to understand the intent of the methods. Consider the following two examples:

```scala
// symbolic method names are hard to understand
channel ! msg
stream1 >>= stream2

// self-evident what is going on
channel.send(msg)
stream1.join(stream2)
```

### Type inference

Scala type inference, especially left-side type inference and closure inference, can make code more concise. That said, there are a few cases where explicit typing should be used:

* **Public methods should be explicitly typed**, otherwise the compiler's inferred type can often surprise you.
* **Implicit methods should be explicitly typed**, otherwise it can crash the Scala compiler with incremental compilation.
* **Variables or closures with non-obvious types should be explicitly typed**. A good litmus test is that explicit types should be used if a code reviewer cannot determine the type in 3 seconds.

### Recursion and tail recursion

Avoid using recursion, unless the problem can be naturally framed recursively (e.g. graph traversal, tree traversal).

For methods that are meant to be tail recursive, apply the `@tailrec` annotation to make sure the compiler can check it is tail recursive. You will be surprised how often seemingly tail recursive code is actually not tail recursive due to the use of closures and functional transformations.

Most code is easier to reason about with a simple loop and explicit state machines. Expressing it with tail recursions (and accumulators) can make it more verbose and harder to understand. For example, the following imperative code is more readable than the tail recursive version:

```scala
// Tail recursive version.
def max(data: Array[Int]): Int = {
  @tailrec
  def max0(data: Array[Int], pos: Int, max: Int): Int = {
    if (pos == data.length) {
      max
    } else {
      max0(data, pos + 1, if (data(pos) > max) data(pos) else max)
    }
  }
  max0(data, 0, Int.MinValue)
}

// Imperative version with explicit for loop
def max(data: Array[Int]): Int = {
  var max = Int.MinValue
  for (v <- data) {
    if (v > max) {
      max = v
    }
  }
  max
}
```

### Implicits

**Be very careful if you need to use implicits**. They have their use, but should be used sparingly.

Here are some scenarios where they are OK to use.

* When defining JSON formatters for the Play! Framework.
* Converters between domain types when this increases readability.
* ​Passing an `ExecutionContext` to methods/functions that operate on `Future`s

### Exception handling

In the MUSIT-Norway codebase an Exception is considered something that is exceptional for normal execution flow. And things like not being authenticated, authorised, trying to look up something that doesn't exist, etc. aren't _exceptional_ in any way. These things are in fact very common and should be handled in code using a type that can encode the failure scenario. Like `MusitResult`, `Option`, or  `Either`.

If you _do_ need to handle exceptions explicitly, there are a couple of general rules to abide by:

* Do **NOT** catch `Throwable` or `Exception`. Use `scala.util.control.NonFatal`. This ensures that we do not catch `NonLocalReturnControl`.
* **AVOID** using `Try` as the return type. Instead, prefer the result type `MusitResult`. This allows for dealing with failure situations in a better way.
* **AVOID** throwing exceptions. There are some cases where it is the correct thing to do. But these are typically outside of the normal execution flow of the application.


### Options

* Use `Option` when the value can vbe empty. **NEVER** use `null`.

* When constructing an `Option`, use `Option` rather than `Some` to guard against `null` values. If you are explicitly defining the value to return, it is OK to use `Some` since the value will clearly not be `null`.

  ```scala
  // Correct
  def foo(arg: String): Option[String] = Option(transform(arg))
    
  // Correct
  def maybeFoo: Option[String] = Some("Hello, world!")
    

  // Wrong
  // arg can be null and may cause a null pointer exception.
  def foo(arg: String): Option[String] = Some(transform(arg)) 
    
    
  ```

* Do not use `None` to represent exceptions. Instead use `MusitResult`.

* Do not call `get` directly on an `Option`, unless you are 100% sure the `Option` has some value.


### Private fields

In Scala `private` fields are still accessible by other instances of the same class. So if you want the variable to only be accessible for the current instance, use `private[this]`.


