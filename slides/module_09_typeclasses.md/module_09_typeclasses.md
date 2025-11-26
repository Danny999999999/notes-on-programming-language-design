---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 8 (short): Typeclasses 

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# Last module

We learned about records.

Finally, something that looks like a struct.

There were some weird wrinkles about field names though [can anyone recall?].

---

# This module

We're going to learn about typeclasses

This is a strange topic, because you already know a lot about them, but don't realize it.

However, you will hear the word "class" and think that it involves classes, and it doesn't.

So what you think you know, you don't know, but you actually know a lot about this subject that you don't know that you know.

---

# What is a class?

Let's talk about classes.

The regular kind. The ones that Java and C++ have.

[What is a class?]

---

# Some valid answers:

1. A set of methods that are bundled with data that only they are allowed to modify
2. A "blueprint" for constructing instances that have certain fields and methods.
3. A way of separating public data or methods from private data or methods
4. A way of grouping data fields together (basically a struct with functions)

---

# Why haven't we seen these in Haskell?

Classes are actually uncommon in functional languages.

They exist, but usually are only used when interacting with non-functional code. 

In functional code, we don't really benefit from them.

Why?

---

# The 3 "pillars" of OO

In several early OO-textbooks, there were three key ideas that often defined OO programming:
1. Encapsulation
2. Inheritance (alternatively, "Abstraction")
3. Polymorphism

[What do these mean?]

People started calling these "the three pillars of OO" for some reason. In my opinion, only encapsulation and polymorphism seem necessary for a language to "feel" OO. "Abstraction" is a vague term, and as a concept is common to all programming languages and paradigms.

---

# The 3 "pillars" of OO (2)

Classes end up being a tool that satisfies all three features:
1. Classes serve as "encapsulation boundaries". They are allowed to declare that certain data is off-limits to everyone but them.
2. Classes are the things which perform inheritance. The object of inheritance was originally envisioned (by early OO language designers) as a way to reuse code. All the fields and methods of the parent class are now usable on/by the child class. (ignoring C++'s private/protected inheritance which is more akin to composition)
3. Classes can have "overridable" (aka "virtual") methods, that do different things when called on instance of inheriting classes. This enables a form of polymorphism.

---

# With me so far?

<!-- _class: invert questions -->


---

# One tool for 3 features?

Is that really good? 

On the one hand, it's kind of cool that one language tool can satisfy so many requirements and enable so many abilities.

On the other hand, it kind of makes classes feel "overloaded". They do all these different things and it might be the case that we could fulfill the other requirements better if we had specific language tools for each one. (I have an example of this)

Let's start with encapsulation. What is it for? Do we still need it?

---

# Encapsulation

Encapsulation means keeping data hidden away with clear boundaries on how and when it can be accessed. Imagine the data in a "capsule". 

*Why* do we want to hide data away? Why do we want to stop it from being accessed? Why do we want to stop it from being modified?

[Thoughts]

---

# Two reasons

There are two main benefits of encapsulation:
1. To reduce coupling. I.e., external code shouldn't need to know the type or encoding of internal fields, so you can freely change them without breaking anything.
2. To reduce the danger of shared, mutable data being changed unexpectantly and violating invariants. 

---

# Coupling

The coupling issue is still something we have to worry about, even though we don't have OO classes in Haskell.

For example, I can write pi as a constant:
```haskell
pi :: Float
pi = 3.1415926
```

And I can write trig functions that might be used with it:
```haskell
sin :: Float -> Float
sin ... = ...

main = print $ sin $ pi / 4.0
```

---

# Coupling (2)

But a 32-bit Float is not really enough room to store more than 6-9 significant figures. Maybe we want a more accurate approximation of pi.

If we replace `pi :: Float` with `pi :: Double`, we create type errors. We can no longer pass `pi / 4.0` to the `sin` function, because that function takes a `Float`, not a `Double`.

The `sin $ pi / 4.0` expression and the `pi` constant are *coupled*, meaning that changing one of them can require the other to change.

---

# Coupling (3)

Pi is such a fundamental constant, that lots of things are coupled with it.

This isn't really a problem: it's a constant, so it doesn't really change. We can just give enough digits for most problems and if someone actually needs more they can [compute the extra digits they need](https://web.archive.org/web/20120310205303/http://www.math.hmc.edu/funfacts/ffiles/20010.5.shtml) using a function, forcing a different kind of type error.

But what about an internal value that no one else needs?

---

# Coupling (4)

Consider making a C++ class to store data for a video game character:
```c++
class Character {
private:
    int hp; // "hit points". how much damage can be taken.
    int mp; // "magic points". how many spells can be casted.
};
```

If we later want to change `hp` and `mp` to be floats to allow fractional values, we can.

We made these fields private. No one else can use them.

We might still need getters so we can, e.g., draw a health bar. This isn't perfect. We can't remove all forms of coupling, but encapsulation lets us reduce it.

---

# Does Haskell have coupling?

Haskell still has this issue. Changing code also forces changes to coupled code.

However, there is a feature we talked about briefly: [modules](https://en.wikibooks.org/wiki/Haskell/Modules).

This feature allows function definitions to be hidden inside a module. Modules end up acting kind of like classes, except only containing static methods/fields.

Therefore, hidden definitions can be changed, and external code (outside the module) won't break as long as the exported module definitions don't change.

But what about the other reason for encapsulation? Shared mutable state?

---

# Shared mutable state

Let's break down this term: "shared mutable state"
- State: data that determines what the program will do.
- Mutable: changeable. I.e., not constant.
- Shared: multiple functions/procedures/modules have access to it.

This kind of data is very dangerous. Consider a variable that stores whether the missle cruiser is ready to launch the missiles. It would be very problematic if that variable suddenly changed because one procedure thought it should be ready but another was currently running (e.g., missiles ignited before the missile bays opened).

---

# Shared mutable state

Huge numbers of bugs are caused by unexpected state changes. 

Some language delicately tease out the different kinds of state. For example, Rust makes it easy to have shared immutable state, and non-shared mutable state, but requires safeguards for shared mutable state.

Haskell takes a different approach: there is no mutability.

So shared mutable state is impossible, because Haskell does not allow anything to change. Problem solved.

---

# The result

Therefore, we don't really need the same encapsulation techniques, like classes.

If all we want to do is hide data to reduce coupling, we can use a module, which is much simpler than a class.

We don't need an encapsulation mechanism beyond that because if nothing can change anywhere, there is no need to have a special mechanism for change.

As a result, the language is just simpler.

---

# Questions?

<!-- _class: invert questions -->

---

# Something is missing

There is, however, an important facility that we haven't talked about. Something where, if Haskell didn't have it, it would be notably less ergonomic than even Java.

I'm talking about subtype polymorphism.

Note the term *subtype* here. We've been seeing a lot of polymorphism in Haskell, but not this specific kind of it.

---

# What even is polymorphism?

Yeah, title, good question.

[What is it?]

---

# The roots of the term "polymorphism"

If you're familiar with Greek etymologies (or if you play a lot of RPGs) you probably know that it means "many forms (ism)". 

But many forms of what?

Many forms of code. Of function definitions.

Polymorphism means that a function or operator will be different depending on the types of the underlying values.

This is a vague definition, and there are actually many different kinds of polymorphism, so let's take a look at some more concrete examples.

---

# Parametric polymorphism

We've been seeing polymorphism a lot. It's how we can write function types like:
```haskell
(.) :: (b -> c) -> (a -> b) -> a -> c
```

This is a kind of polymorphism: *parametric polymorphism*

The idea here is that the composition operator `.` will generate a different kind of function depending on the types of the functions it is given as arguments.

`a`, `b`, and `c` are *type parameters*. Basically variables that are filled in with types.

---

# Why do this?

Normally, when you search for examples polymorphism in computer science, you end up with descriptions of how it enables code that is more *flexible*. This is kind of misleading in my opinion.

For example, in this Java code, the `+` operator is polymorphic:
```java
var i = 7 + 3;
var s = "hello " + "world";
```

It's true that this is a kind of polymorphism. Specifically, *ad-hoc* polymorphism. The operator has two different meanings: addition for number types and concatenation for strings. It only has those meanings and they are completely different. It's polymorphic, but we can't really extend it. 

---

# Why do this? (2)

Back to *parametric* polymorphism, why do this?
```haskell
(.) :: (b -> c) -> (a -> b) -> a -> c
```

If we wanted flexibility, why wouldn't the language just do this?

```haskell
(.) :: (Anything -> Anything) -> (Anything -> Anything) -> Anything -> Anything
```

The reason: this kind of polymorphism makes the code *less* flexible. We specifically want to exclude certain kinds of types.

We want the return type of the first function to *have* to be the same as the input type of the second function. We don't want the user to have a choice, because they might compose two incompatible functions.

---

# So what is polymorphism really?

For this reason, I don't like to define polymorphism in terms of what a function *does*. 

Instead, I prefer to define it in terms of its *type*.

Therefore, my definition of polymorphism is this:
*When a function changes its concrete type depending on the types of its operands.*

By concrete type, I mean the most specific type. For example, in `"hello " + "world"` in Java, `+` becomes a function from a pair of strings to a string.

This definition covers all the different kinds of polymorphism we've seen. 

---

# Questions

<!-- _class: invert questions -->

---

# Different kinds of polymorphism

Now let's revisit the different kinds of polymorphism and introduce the one we care about for this lecture:

---

## 1. Ad-hoc polymorphism

This is  when an operator or function changes its type depending on the type of its arguments, but in a way that is specific to that exact combination of types.

For example, `1 + 2.5` in C is interpreted as a `double` because of the specific type coercion rules in C. The `1` ends up getting converted to `1.0`. 

Other languages have different rules. For example, in Rust the operands all have to have the same type. You can't add a float to an integer without converting one of the argumetns.

---

## 1. Ad-hoc polymorphism (2) 

This kind of polymorphism doesn't have to be hardcoded.

C++ allows operators to be overloaded in an ad-hoc way. You can say "whenever we add a Foo to a Bar, use this definition of `+` instead."

Example:
```c++
Foo operator + (Bar a, Baz b) {
    ...
}
```

This says "whenever we add a `Bar` to a `Baz`, call this function and the result it returns will be a `Foo`."

This seems incredible flexible, but the types have to be known at compile time. Otherwise the compiler will select the wrong overload.

---

## 2. Parametric polymorphism

This is the kind of polymorphism where types can be parameters:
```haskell
someParametriclyPolymorphicFunction :: a -> b -> c -> d
```

We've talked about this, but please don't think it only happens in Haskell.

In Java and C++, type parameters appear in angle brackets:
```java
var l = new ArrayList<String>();
l.add("hello");
```

Parametric types are called *generics* and Java and *templates* in C++.

Here, the `ArrayList` class uses parametric polymorphism. It's not just an `ArrayList`, it's an `ArrayList` *of* `String`s. 

---

## 2. Parameteric polymorphism

This ability makes it so that the `add` method can only take a particular type.

In this case, it takes `String`s, because we defined our `ArrayList` to be of that type.

However, if we had made an `ArrayList<Integer>`, then the `add` method would take an `Integer` instead. 

We use parametric polymorphism to allow type rules to be connected between different variables. Here, we're saying the input type of the `add` function has to be the same as the type of data stored in the array list.

Before parametric polymorphism was added to Java, `ArrayLists` could store anything, and the `add` method took only an `Object`. This meant that you couldn't rely on the array only having a particular type in it, and you had to cast whenever you got something out of the array, which increased type errors.

---

## 3. Subtype polymorphism

Now we get to the primary subject of this module.

Subtype polymorphism is when a function can be called on a bunch of different types, but it does something different for each one.

This is how interfaces work in Java (or typescript for my CS 220 students). Let's remind ourself about interfaces first, then see how there's a similar concept in Haskell.

---

## 3. Subtype polymorphism (Interfaces)

```java
interface Barkable {
    void bark();
}
```

What does this mean?

`Barkable` is an interface. You cannot *create* a borkable like this:
```java
var something = new Barkable(); // wrong, this is an error
```

Why? Because `Barkable` just means "something that can `bark`". We have not defined what barking means, only that it is some kind of thing that we might want to do. So Java wouldn't know what would happen if we wrote `something.bark()`. We haven't defined that yet.

---

## 3. Subtype polymorphism (Interfaces 2)

In order to use the interface, we have to implement it, which in Java, is something that only a class can do:

```java
class Labrador implements Barkable {
    public void bark() {
        System.out.println("woof!");
    }
}
```

`Labrador` is something that we can create. This is permitted:
```java
var sparky = new Labrador();
sparky.bark();
```

---

## 3. Subtype polymorphism (Interfaces 3)

Where this gets interesting is when we have multiple options:

```java
class Chihuahua implements Barkable {
    public void bark() {
        System.out.println("yarp!");
    }
}
```

Now, there are two different functions (technically methods, but let's keep using the term *function*): one of them belongs to `Labrador`, and the other to `Chihuahua`.

Both do different things. They print different messages. But they have the same type, so one can be substituted for another.

---

# Questions?

<!-- _class: invert questions -->

---

# Why the big deal?

You might wonder why this would be such a big deal as to get a special name.

It's because this feature requires a lot more language support than you might think.

Consider this snippet:

```java
Barkable sparky = new Labrador();
Barkable princess = new Chihuahua();
sparky.bark();
princess.bark();
```

This will print:
```
woof!
yarp!
```

---

# Why the big deal? (2)

Here's the big question I want to ask: how does Java know what to do when we write `whatever.bark();`?

That is, [how does it know which specific method to call]?

---

# Not quite what you think

You might think "oh, it sees that you wrote `sparky = new Labrador()`, so it knows that `sparky` is a labrador.

That is probably true in this particular case. There is an optimization called *devirtualization* where the compiler will try to trace the flow of execution so it can know for sure which method to call.

But it can't always work that way. What about here?
```java
Barkable barker;
if (Coin.flip() == Coin.HEADS) {
    barker = new Labrador();
} else {
    barker = new Chihuahua();
}
barker.bark(); // which one gets called? 
```

---

# Not quite what you think (2)

Don't say "oh, it's not a true random number generator, so the compiler can know what's going to happen". We usually seed the RNG based on the current time.

Imagine we hooked the RNG up to a geiger counter next to a smoke detector or something. It's *random*. How can it know which function to call?

Answer: a special language feature that operates behind the scenes. In Java this is a VTable (virtual table), a table of function pointers. So barker is pointing to a particular set of function pointers that get called for each abstract method.

This is called *dynamic dispatch*. It means *dynamically* (meaning, at runtime) *dispatching* (calling) a function call to the correct definition.

---

# Questions?

<!-- _class: questions invert -->

---

# What about Haskell

Now we get to the main point: Haskell has this feature, too, but it's not called an interface. In fact, it's a bit more powerful and flexible than an interface.

The equivalent feature in Haskell is called a *typeclass*.

**HUGE IMPORTANT NOTE:
`typeclass` in Haskell is similar to `interface` in Java**
**it is NOT similar to `class`**

---

# A few repititions

Typeclasses are interfaces, not classes

Typeclasses are interfaces, not classes

Typeclasses are interfaces, not classes

Haskell does not have classes. It doesn't need them. It *does* need interfaces, and they are confusingly called *typeclasses*.

---

# Making one

With that important background out of the way, let's finlaly make a typeclass in Haskell:

```haskell
class Barkable a where
    bark :: a -> String
```

*Please* ignore the fact that the keyword is `class`. It's a trick! Typeclasses are like interfaces!

This code says "an `a` can be a `Barkable` if we can call `bark` on it giving a `String`.

And just like an `interface`, a typeclass is not very useful unless it has some data to apply do. But Haskell doesn't have classes, so what implements typeclasses?

---

# Ordinary data types

Instead, we just have ordinary data types implement type classes:

```haskell
data Labrador = Labrador()
data Chihuahua = Chihuahua ()

instance Barkable Labrador where
    bark _ = "woof!"

instance 
```

---

If we inspect the type of `bark` in GHCI, we can finally see that wide arrow notation that we've probably been seeing a lot in error messages:

```
ghci> :t bark
bark :: Barkable a => a -> String
```

---

# Class's being overly overloaded

I promised earlier that I had an example of how the design of classes needing to enable 3 different language features could cause them to not fulfill each one as well as a purpose-built solution.

Here are some examples...

---

# Soa versus Aos

Classes are used for encapsulation. They act as boundaries 
