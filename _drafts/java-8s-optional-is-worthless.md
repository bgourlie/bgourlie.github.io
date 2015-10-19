---
layout: post
title:  "Java 8's new Optional type is worthless"
date:   2015-10-14 22:34:02
categories: java optional
---
When Java 8 was released, I was excited to see the new [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) type.  The NullReferenceException is a common source of errors (see: [billion
dollar mistake](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)), and
it's nice to have a first-class type to avoid them.  After spending some time with Java's option type, I've come to the conclusion that it does nothing to solve the problem at
hand.

### First, a little background...

Perhaps you have a method that calls out to a database to fetch a record.  If that record doesn't exist, we can indicate that by returning null.  Pretty intuitive, right?
The problem is, there's no good way for the caller to know that the function may return null without digging into the code.  This is where [option types](https://en.wikipedia.org/wiki/Option_type) comes in.

At a high level, the option type is simple --  It wraps a type `T`, which may or may not be present.

Let's pretend our hypothetical database method now returns an `Optional<DbRecord>`.  If the record is present, it will be accessable via the Optional value.  If not, Optional can indicate that.  A programmer looking at the code for
the first time will know, just by looking at the return type, "Hey, this method may not return a record!  I'll have to handle that scenario."

### It's all about the guarantees

Static type systems offer a lot of guarantees.  If we create a string variable and attempt to assign an integer to it, our program won't compile.  The compiler is the
first line of defense when it comes to preventing common errors, and the more errors the compiler can prevent, the better.

Most languages with a first-class option type
have really slick type systems.  The option type is typically implemented in such a way that the programmer *must* handle the possibility of a present and non-present value.  The constructs that make this possible, algebraic data types and pattern matching, don't
exist in Java.  Because of this, one of the greatest benefits of the option type is lost entirely -- A compiler that *guarantees* we handle both scenarios.

But wait, there's more...

Another great thing about pattern matching, and how it relates to option types, is you can only access the wrapped value if it exists.  This affords us another guarantee -- We
can never access a non-present (essentially null) value.  Again, this is not the case in java.  If we call `Optional.get()` when no value is
present, a `NoSuchElementException` is thrown.  This defeats the purpose entirely!  We're essentially trading the `NullReferenceException` for `NoSuchElementException`.

The last flaw with Java's optional implementation is a bit ironic:  It's entirely possible for optional values to be null.  Again, this is a consequence of Java's type system.  In
this case, it's a lack of value types.  So if we want to write safe code, we still need to do null checks.  In addition, we need to do the isPresent() check.  We get zero guarantees, require an additional check, and have a
an additional bit of indirection in accessing our value.

### What they should have done

You might think I'm advocating changes to Java's type system.  I'm not.  Java is at a point in it's life where being pragmatic trumps being cutting edge and fancy. Oracle should have
realized that solutions to the null problem already exist, but they're not great, primarily because there is no first-class support.  That solution is static analysis.

Sun understood this nearly a decade ago when [JSR-305](https://jcp.org/en/jsr/detail?id=305) was introduced.  This proposed standard set of annotations would help to aid static
analysis tools in determining places that `NullReferenceExceptions` could occur.  They solve the same problem that the optional type is meant to solve, in a much more pragmatic
fashion.

As it stands, some tools rely on these abandoned annotations which are now vended by a 3rd party.  Different tools may interpret the annotations differently, or use
their own similar-yet-different annotations.  Lacking a standard, it's impossible to have reliable analysis across multiple projects.  I would like to see Oracle support and
official set of annotations.
