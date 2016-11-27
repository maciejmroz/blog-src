+++
title = "Functional programming trojan horse"
date = "2013-09-22"
slug = "functional-programming-trojan-horse"
categories = [ "Programming", "Server Side" ]
tags = [ "Scala", "JVM"]
+++

All of game server code at my current company is using C++, and we are still starting new projects using existing C++ framework. I gave a presentation on our server architecture a few months ago, available [here](http://www.slideshare.net/maciejmroz/architektura-serwera-gier-online) (it's in Polish, it was local event here in Kraków). After giving the presentation I was approached by a guy (sorry, don't remember the name) who said something like: "It's cool you did all that in C++. It'd be interesting to see you talk next year about how you rewritten the whole thing in Java :)". I instantly answered: "Right now I'd do it with Scala and Akka".

I wasn't even thinking about it, it was absolutely obvious: if I were to do it from scratch, it'd do it in Scala. It felt a bit like an inception coming to the surface of my mind. It became pretty clear to me to that the only reason I am working on C++ server technology is because of the mild lockin: the benefits of rewriting server framework are smaller than gains. It suits our needs very well, programmers are familiar with it, so why change something that just works? However, it remains true that not only it would take less time to implement using Scala and Akka, but Akka is in some areas vastly better than what we did in C++.

## What's so special about Scala?

I would never advocate use of programming language because of a cool name (admit it, it is cool :) ). The reason I actually decided to learn Scala is very simple: for the kind of problems I'm dealing with (complex server side systems), JVM as a platform has won. Sure, there are alternatives, but platform is much more than a language: it's also a vm, tools, libraries and community. On these fronts, JVM is a winner by wide margin. Because I generally dislike Java for being so verbose (it's 21st century, and we are expected to write code in simplified version of C++?), after playing for a short while with Clojure (which is awesome, just as any other Lisp dialect out there :) ), I quickly settled on Scala. The fact that functional programming fits my way of thinking about solving problems helped a lot, too :) So far I have spent about six months learning Scala, and still feel a lot like a novice. However, I know my way around it enough to have an opinion: I think it is the most important programming language created in past 10 years. Scala also is what Java should have been in the first place, but failed to ever become.

Word of warning: Scala is a big language, and learning it will take a lot of effort. While on the surface it is "Java with sane syntax and traits" it is also a functional programming language. Sooner or later you are going to hear words like functor or monad. The good thing is that you can get started very quickly and learn new concepts as you go. I will not cover everything there is to know about Scala, only the parts I found to be the most important. In the end, if you wish to learn any new programming language, you'll have to do it on your own.

## Syntax

Over the years I have developed a fair bit of negative attitude towards languages with a lot of boilerplate. In fact I believe that longer code is the code with more bugs, regardless of programmer skill level. At the same time, I am a big fan of strong and static typing, which obviously creates a conflict. Both Java and C++ are very far from being expressive. Scala does a lot to remove boilerplate typically found in such languages, and allows the code to look much more similar to dynamic languages. You can write code like this: 
    
{{< highlight scala >}}
    val knownPermissionNames = Permissions.values.map(_.toString)
{{< /highlight >}}

This line of code produces a collection of String objects. However, collection type is never named explicitly here. The type is inferred by the compiler. Type inference saves a lot of completely unnecessary typing, but that's not all you can see in code above. If you noticed that there's no semicolon up there, congratulations. Most of the time, there's no need to use it, because it also inferred by the compiler. What else is up there? Anonymous function passed to map, using shortened syntax with underscore character acting as placeholder for function argument. That's still not all of the good stuff: in many cases you can omit parentheses on function calls, and there is no need for explicit return from function - the function value is simply the value of its last statement. In short: **a lot** less typing, without sacrificing compile time type safety, and most of the time, actually improving code readability. What's not to like? :)

## Functional goodness

Scala is not only an OO language, but also a functional one. That means functions are "first class" citizens of the language: they can be stored as values, passed around, and returned from other functions. Scala supports both partial function application and currying, which really comes in handy. You can define anonymous functions. Also functions can be nested inside one another (quite useful because nested function has access to enclosing scope). The compiler is capable of optimizing out tail recursion, which makes it nice alternative to using "while" loop (and btw: it is the only kind of loop that Scala really has, "for" keyword is not a loop!), and is just as efficient at bytecode level. Now, trying to argue functional vs OO is not my point here. Use it if you want to, or when you are ready to :) You'll end up with code that's loosely coupled, short, readable and maintainable. These are very good goals to have.

It's time to talk about another very important concept closely tied to functional programming, which is data immutability. While immutability in Scala is not enforced, it is considered good programming style and is mostly default. Working with immutable data has a lot of advantages - there's no real need to encapsulate state in objects, because there's no way to mutate the state. For the same reason, there's no need for doing any defensive copies. Once you start to think in terms of data transformation rather than mutation, programming becomes simpler: code is easier to understand and easier to reason about. Immutable data structures are thread safe by nature - you can pass them around like there's no tomorrow. What I found very quickly when I started playing with Scala, was that it was very natural programming style to keep data and code that operates on it separate instead of tightly coupling them inside the same class (i.e. by using case class and its companion object). Scala has pretty awesome standard collection library, and most collections have mutable and immutable versions (with immutable being default). Learning to use collections effectively is probably going to take some time, but simply using map, fold and filter operations can get you very far very easily. Again, you can learn less commonly used methods on standard collections gradually (and it is absolutely worth it!).

Important feature that complements immutable data is pattern matching. It may initially look like "switch on steroids" but it is an understatement. Pattern matching is one of the most powerful features of the language. Because you are passing around data structures, you can easily match on the structure or part of it, and extract what's needed, without writing extra utility functions or doing any intrusive modifications to classes holding the data (when data is immutable, there aren't many reasons not to have everything public by default ...). Simple pattern matching looks like this (code from my pet project): 
    
{{< highlight scala >}}
    teamModificationResult match {
        case models.TeamModificationNone(userId, role) => (...)
        case models.TeamModificationMemberAddSuccess(userId, role) => (...)
        (...)
    }
{{< /highlight >}}


You can do this kind of matching on arbitratily nested data structures, and destructure exactly the needed parts. Not enough? Destructuring process is actually programmable through feature called extractors. Conceptually, you can imagine that patterns are not really some fixed compile time constructs, but your own code. If you want to pattern match on string containing JSON encoded data, you can do it if you want to. Standard Scala regular expressions are also extractors, by the way :)

When talking about functional programming parts of the language, I believe it is important to mention for comprehensions. Scala ‘for’ is in fact syntactic abstraction over map, flatMap, filter and foreach methods. This is very important because it makes it possible to use for comprehensions with non-collection types. The type that implements map and flatMap methods is essentially what math people call a monad. Hence, you’ll cometimes see for comprehensions being referred to as monadic comprehensions. Worry not, while it initially is a bit mind bending, it is not rocket science :) Option[A] is an example of monad that is not a collection type. Because it has map and flatMap methods, it can be used in for comprehension. What’s important to remember is that ‘for’ in Scala is just a syntax sugar, and has nothing to do with loops!

## Error handling

In functional language, Java style exceptions are simply out of place. While there have been many discussions over the years about merits of using exceptions (or not), because exceptions break program control flow, they make it a lot harder to reason about program correctness.

Scala does support Java exceptions but it is mostly used to interface with Java code. In functional programming world, there are better solutions.
The first very important and very commonly used thing is Option generic type. Option[T] can hold value of type T, or nothing (called None). When you return Option[T] you say “I’ll return a value of type T, but you can’t be sure it will really be there”. This may or may not be an error, sometimes not returning a valid value is entirely expected.

Why not simply pass around null references? :) The answer is obvious to any programmer out there: we don’t always check for null … and our code crashes a lot because of that :)

Option[T] has two interesting properties:

* Option[A] can be transformed into Option[B] by using (A) => B function passed to map, or (A) => Option[B] function passed to flatMap. That means you can write your code as if “null case” doesn’t exist and code happy path only when doing computations. Option[T] propagates None value for you. That also means Option[A] is a monad and can be used with for comprehensions, as mentioned earlier.
* When you actually want to get value out of Option, you have to do it manually. You can still get an exception at this stage (if you use plain get), but typically you’ll use pattern matching (on both value and None cases) or getOrElse method. Essentially the language encourages programmer to handle the None case, instead of encouraging programmer to assume everything is great and null will never be there.

In general, using null in Scala is a bad style, there’s simply no need for it.

The second similar but a bit more complex type is called Either[A,B]. As name suggests, it holds either A or B. It is very common for Either to hold value or an error (which can even be an exception object, but it is passed around, not thrown). It can be transformed in similar way as Option by using mapping functions.

## What else is there?

A lot. Most importantly concurrency features: actors, futures, parallel collections, software transactional memory. Also libraries like scalaz or shapeless that push functionals part of Scala pretty hard. I did not talk about OO parts at all, not because I don’t consider them important, but because it’s the first thing you’ll run into (along with constructs like cake pattern) when starting to learn the language.

In the title of this blog post I named Scala a ‘trojan horse’. That’s exactly what it is. While technological advancement makes writing certain classes of software easier and easier, it also pushes us towards solving problems we wouldn’t think of trying to solve 10 years ago. The biggest challenge right now is in distributed and concurrent systems. This is where we need every bit of correctness and abstraction we can get from compiler and language. This is really, really hard stuff and this is exactly where functional languages shine. However, languages like Haskell are alien to current breed of programmers who grew up writing imperative code most of their life (if not all of it). Distributed and concurrent systems are also exactly where the growth is right now (there are many trends that overlap: mobile devices, “Internet of things”, ubiquituous cloud computing, “big data” technologies etc). Scala is the bridge between old world and new one. The destination is not known (if history taught us anything, Scala will not be the last word and will be replaced by something better) but right now Scala seems to be the language best suited to face the future.