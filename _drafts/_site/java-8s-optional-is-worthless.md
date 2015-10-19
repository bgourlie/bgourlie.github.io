#### Usage of `Optional` in place of possibly `null` values

The inventor of the null reference has famously called it his "[billion dollar mistake](http://www.infoq.com/presentations/Null-References-The-Billion-Dollar-Mistake-Tony-Hoare)." Some modern languages have chosen to eliminate the null reference entirely, while established languages are introducing types to represent a possibly absent value.  As of Java 8, there's an `Optional<T>` type to serve this purpose.

In theory, using `Optional<T>` where a value can be absent and `T` otherwise should afford us the ability to operate under certain assumptions.  On one hand, when passing just `T` to a method, we know there isn't a need to check for null.  On the other hand, when passing `Optional<T>`, we know to handle the possibility that the value isn't present.  

Unfortunately, Java's `Optional` doesn't afford us these assumptions.  Comparing two examples, we see that the same pitfalls exist whether passing `null` or `Optional` values:

** Using `null` **

    public void Foo(String bar) {
    	int len = bar.length(); // Possible NullReferenceException
        System.out.println("Len is " + len);
    }

** Using `Optional` **

    public void Foo(Optional<String> bar) {
    	int len = bar.get().length(); // Possible NullReferenceException, NoSuchElementException
        System.out.println("Len is " + len);
	}

As you can see, there's no real benefit to using `Optional` in this scenario, and in fact, the `Optional` version can throw multiple unchecked exceptions.  Let's dress up both examples with annotations and assume we're using a [future version of FindBugs with `Optional` support](http://sourceforge.net/p/findbugs/feature-requests/302/), using static analysis to prevent these errors from happening:

** Using `null` **

    public void Foo(@Nullable String bar) {
    	int len = bar != null ? bar.length() : 0; // Findbugs will force a null check
        System.out.println("Len is " + len);
    }

** Using `Optional` **

    public void Foo(@Nonnull Optional<String> bar) {
    	int len = bar.isPresent() ? bar.get().length() : 0;  // FindBugs will force an isPresent() check
    	// alternatively:
    	// int len = bar.orElse("").length()
        System.out.println("Len is " + len);
	}

After introducing `@Nullable` and `@Nonnull` annotations and using static analysis to prevent the same class of errors in both examples, what benefit does `Optional` afford us?  The `@Nullable` annotation is just as self-documenting as `Optional<T>`.  We do a `null` or `isPresent()` check either way.  So what gives?

As it turns out, Java's type system isn't capable of implementing an effective `Optional` type.  A proper `Optional` type should *never* be null, and this should be enforced by the compiler.  Additionally, the `get()` method has the potential to throw a `NoSuchElementException`.  How is this any better than a `NullReferenceException`?

It's a bit of a mystery as to why the `Optional` type was introduced in the first place.  Java developers would be better served if Oracle revived the now dormant [Annotations for Software Defect Detection](https://jcp.org/en/jsr/detail?id=305) and supported these annotations with static analysis done via javac.  As it stands, we have various non-standard annotations and multiple 3rd party static analysis tools to aid us in writing sound Java code.  `Optional` does nothing to improve the situation.
