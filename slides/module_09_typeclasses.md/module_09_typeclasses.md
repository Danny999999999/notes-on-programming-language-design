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

In several early OO-textbook, there were three key ideas that defined OO programming:
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

However, there is a feature we won't cover: [modules](https://en.wikibooks.org/wiki/Haskell/Modules).

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




---

# Class's being overly overloaded

I promised earlier that I had an example of how the design of classes needing to enable 3 different language features could cause them to not fulfill each one as well as a purpose-built solution.

Here are some examples...

---

# Soa versus Aos

Classes are used for encapsulation. They act as boundaries 
