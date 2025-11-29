---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 10: Typeclasses 

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# Last module

We learned about typeclasses.

We saw some examples

---

# This module

We're going to learn about typeclasses

This is a strange topic, because you already know a lot about them, but don't realize it.

However, you will hear the word "class" and think that it involves classes, and it doesn't.

So what you think you know, you don't know, but you actually know a lot about this subject that you don't know that you know.

---


---

# Now for some math

Okay, it's time for some abstract math.

You see, mathematicians came up with this idea of typeclasses long before programmers ever did.

Consider the idea of a [Semigroup](https://en.wikipedia.org/wiki/Semigroup).

This is a concept from mathematics. A semigroup is a set (i.e., a datatype) that also has an associative binary operation that it is closed under.

For example, integers are a semigroup "under" addition, because the sum of two integers is another integer. (we use the preposition "under" to describe what the unifying operation is because sometimes there's more than one: like multiplication)

---

# Why abstract structures?

Mathematicians invented abstract structures like this because they can prove things about entire families of sets and operations.

For example, [Fields](https://en.wikipedia.org/wiki/Field_(mathematics)) are kind of like semigroups, but they have two operations instead of just 1, and they have two identities (1 for each op.). Also both operations are invertable.

Did you know that if a Field's set is finite, [then its size has to be a power of a prime number](https://en.wikipedia.org/wiki/Finite_field)?

Why? Because it turns out if that weren't the case, the second operation would not always be invertible. This fact comes from number theory. It's not an axiom that the size has to be a power of a prime, it just naturally falls out of the rules. We can state these rules without knowing anything else about the structure. It's just a set, two invertable operations, two identity elements, and a finite number of elements in the set.

---

# Semigroups are in Haskell

It might seem strange, but Semigroups are in Haskell too.

They're actually very useful.

So far we've learned a lot of Haskell and we've tried to make analogies to other programming languages. However, that had to end at some point. This is something most languages do not have.

So what is a Semigroup? Let's look at its typeclass and see if we can figure it out...

---

# The `Semigroup` class

```haskell
class Semigroup a where
    (<>) :: a -> a -> a
    sconcat :: NonEmpty a -> a
    stimes :: Integral b => b -> a -> a
```

A semigroup is any mathematical structure (read: datatype) that has a closed binary operation. That's the point of `<>` above.

This isn't a standard mathematical operator with a well-known meaning. It's a placeholder. You can define it however you want, and different semigroups define it completely differently. 

The other two functions (`sconcat` and `stimes`) are actually optional. There is a *default definition* in terms of `<>`, so you actually only need to define `<>`. Let's see some examples of `<>` first, and then talk about the other two functions.

---

# A really basic semigroup

Remember when I said that integers form a semigroup *under* sum?

That is, if I add two integers together, I get another integer. That's a binary operation.

Therefore, integers and addition are a semigroup.

Unfortunately, integers and multiplication are also a semigroup. There is more than one option for a reasonable semigroup with 

---

# `Semigroup`'s other functions



---

# Default implementations



---

# Finally functors

---