---
layout: post
title:  "Java 8's new Optional type is worthless"
date:   2015-10-14 22:34:02
categories: java optional
---
When Java 8 was released, I was excited to see the new [Optional](https://docs.oracle.com/javase/8/docs/api/java/util/Optional.html) type.  The null reference is a common source of errors in most languages (see: [billion
dollar mistake](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)) and [option types](https://en.wikipedia.org/wiki/Option_type) are a solution to this problem.  After spending some time with Java's option type, I've come to the conclusion that it doesn't actually solve anything. 

### First, a little background...

Suppose we have a method that calls out to a database to fetch a record.  If that record doesn't exist, it's a common practice to return null.  Intuitive enough, right?

The problem is, there's no good way for the caller to know that the function may return null without digging into the code.  Inevitibly, the null reference is passed around and eventually
dereferenced.  Boom! -- NullReferenceException.  This is where the option type comes in.

At a high level, the option type is simple --  It wraps a type `T`, which may or may not be present.

Let's pretend our hypothetical database method now returns an `Optional<DbRecord>`.  If the record is present, it will be accessable via the option type.  If not, the option type can indicate that.  A programmer looking at the code for
the first time will know, just by looking at the return type, "Hey, this method may not return a record!  I'll have to handle that scenario."

### It's all about the guarantees

Static type systems offer a lot of guarantees.  If we create a string variable and attempt to assign an integer to it, our program won't compile.  The compiler is the
first line of defense when it comes to preventing common errors.  The more errors the compiler can prevent, the better.

Most languages with a first-class option type
have slick type systems.  The option type is typically implemented in such a way that the programmer *must* handle the possibility of a present and non-present value.  The constructs that make this possible, algebraic data types and pattern matching, don't
exist in Java.  Because of this, one of the greatest benefits of the option type is lost entirely -- A compiler that *guarantees* we handle both scenarios.

But wait, there's more...

Another great thing about pattern matching, and how it relates to option types, is you can only access the wrapped value if it exists.  This affords us another guarantee -- We
can never access a non-present (essentially null) value.

Again, this is not the case in Java.  If we call `Optional.get()` when no value is
present, a `NoSuchElementException` is thrown.  This defeats the purpose entirely!  We're essentially trading one unchecked exception for another.

The last flaw with Java's implementation is a bit ironic -- It's possible for the optional reference itself to be null.  So if we want to write safe code, we still need to do null checks in addition to an `isPresent()` check.  We get zero guarantees, require an additional check, and have
an additional bit of indirection in accessing our value.

### What they should have done

You might think I'm advocating changes to Java's type system.  I'm not.  Java is at a point in it's life where being pragmatic trumps being cutting edge and fancy. Oracle should have
realized that Java's type system is incapable of properly implementing an option type and taken a different approach.

It turns out a more pragmatic solution *does* exist, but it's not great, primarily because there's no first-class support.  That solution is static analysis.

Sun understood this nearly a decade ago when [JSR-305](https://jcp.org/en/jsr/detail?id=305) was introduced.  This proposed set of *standard* annotations would help to aid static
analysis tools in identifying null reference errors.  They solve the same problem that the option type is meant to solve, in a way that doesn't require
drastic changes to the type system to implement effectively.

As it stands, some tools still rely on these annotations which are now vended by a 3rd party.  Some tools interpret the annotations differently than others, or use
their own similar-yet-different annotations.  Lacking a standard, it's impossible to have reliable analysis across multiple projects.

Unfortunately, Oracle is not very keen on static analysis.  In fact, they're [on record stating](https://blogs.oracle.com/maryanndavidson/entry/mandated_third_party_static_analysis) that "Despite the marketing claims of the third parties that do this, 'third party
code review' is not 'industry best practice.'"

[Some](http://research.google.com/pubs/pub34339.html) [folks](http://fbinfer.com/) [may](http://research.microsoft.com/en-us/projects/slam/) [disagree](http://clang-analyzer.llvm.org/).
