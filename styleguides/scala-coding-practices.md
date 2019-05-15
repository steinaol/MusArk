# Writing Scala code for the MUSIT project


## Code Quality

TL;DR

1. Keep It Simple Stupid (KISS). Avoid unnecessary complexity.
2. Scala is statically typed, so you shouldn't fear refactoring.
3. Do not implement code that isn't required or necessary.
4. Follow the [MUSIT-Norway Scala Styleguide](scala-styleguide.md) for Scala.

### Readability

One of the key identifiers of a high quality code base is _readability_. So what defines a readable code base?

Readability can be defined by how easy it is to _reason_ about what is going on. E.g. it is easier to reason about a function that does one thing, than a function that does several things depending on the value of some argument. The latter case will have many potential branches of execution while the first one will (obviously) have only one.

Keeping code complexity low is very important. Scala is a very powerful language
with an abundance of features and possibilities. Code that is unnecessarily complex is arguably harder to read and reason about than a simple solution.

Another key part of readability is to follow the defined coding style and naming conventions that are defined for a project. This will ensure that code, regardless of author, will feel familiar.

### Testability

Being able to test the code is essential to ensure quality over time. With Scala there are quite a few things that you get for free thanks to the strong type system etc. But, behaviour should never the less be verified through testing.

### Premature generalisation

Do _not_ implement generalisations of concepts before there is an explisit need for them. It might be tempting to generalise the implementation for a concept from the start. But, in reality, these generalisations tend to be made without all the necessary information. Resulting in incorrect implementations and complex, hard to read, code.

As a general rule, it is better to duplicate the implementation unless the pattern of generalisation is self-evident. Generalisations / abstractions should _never_ result in functions that loose precision of the types involved, or results in several execution forks.

