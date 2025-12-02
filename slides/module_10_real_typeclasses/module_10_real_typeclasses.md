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

We saw some examples, espeially `Num`.

[What defines a typeclass? That is, what things separate one typeclass from another?]


---

# This module

We're going to learn some *very important typeclasses*

Specifically:
1. `Monoid`
2. `Functor`
3. `Applicative` 

At first, these will seem like very abstract things. 

It turns out, you already know the first two pretty well, but you don't realize it yet.

The third one will take some understanding.

---

# When Haskell gets weird

Haskell has a reputation. You're probably starting to understand that now.

What we're going to cover is precisely the point where Haskell gets *weird*. That is, where it starts using weird terms from category theory for everything.

Rest assured: the weird names like `Monoid` refer to perfectly ordinary things that occur naturally in math and in other programming languages. 

You don't need to be a category theorist to understand Haskell. (which is good or I wouldn't be qualified to teach this class)

---

# Now for some math

Okay, it's time for some abstract math.

You see, mathematicians came up with this idea of typeclasses long before programmers ever did.

Consider the idea of a [Semigroup](https://en.wikipedia.org/wiki/Semigroup).

This is a concept from mathematics. A semigroup is a set (i.e., a datatype) that also has an associative binary operation that it is closed under.

For example, integers are a semigroup "under" addition, because the sum of two integers is another integer. (we use the preposition "under" to describe what the unifying operation is because sometimes there's more than one: like multiplication)

---

# Can you come up with more?

What about integers under multiplication? Yes, that is also a semigroup!

To have a semigroup, you just need a set (or a datatype) and a binary function with type `ElementOfThatSet -> ElementOfThatSet -> ElementOfThatSet`

[Can you come up with some more examples?]

---

# Why abstract structures?

Mathematicians invented abstract structures like this because they can prove things about entire families of sets and operations.

For example, [Fields](https://en.wikipedia.org/wiki/Field_(mathematics)) are kind of like semigroups, but they have two operations instead of just 1, and they have two identities (1 for each op.). Also both operations are invertable.

Did you know that if a Field's set is finite, [then its size has to be a power of a prime number](https://en.wikipedia.org/wiki/Finite_field)?

Why? Because it turns out if that weren't the case, the second operation would not always be invertible. This fact comes from number theory. It's not an axiom that the size has to be a power of a prime, it just naturally falls out of the rules.

---

# Semigroups are in Haskell

It might seem strange, but Semigroups are in Haskell too.

They're actually very useful.

So far we've learned a lot of Haskell and we've tried to make analogies to other programming languages. However, that had to end at some point. This is something most languages do not have (but maybe should).

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

This isn't a standard mathematical operator with a well-known meaning. It's a placeholder. You can define it however you want, and different semigroups define it completely differently. Sometimes it's `+`, sometimes it's `++`, etc.

The other two functions (`sconcat` and `stimes`) are actually optional. There is a *default definition* in terms of `<>`, so you actually only need to define `<>`. Let's see some examples of `<>` first, and then talk about the other two functions.

---

# A really basic semigroup

Remember when I said that integers form a semigroup *under* sum?

That is, if I add two integers together, I get another integer. That's a binary operation.

Therefore, integers and addition are a semigroup.

Unfortunately, integers and multiplication are also a semigroup. There is more than one option for a reasonable semigroup with integers.

So, how do we distinguish them?

---

# Special data types

There is a `Sum a` data type. So `Sum Integer`, `Sum Float`, etc.

The only requirement is that `a` be a number.

This data type implements `Monoid`. 

So, now, we can, uh, add numbers together...

```haskell
import Data.Semigroup
Sum 10 <> Sum 20 == Sum {getSum = 30}
```
(the `Sum` data type stores the actual integer inside of a field named `getSum`, I'm guessing because it means they didn't need to add a pattern matcher)

Isn't that handy? We can add numbers everyone! That's super useful right!

---

# I know what you're thinking

Okay! Give me a second! I know what you're thinking. "We could already do that! And it was easier! And we didn't need to import `Data.Semigroup` or use `getSum` to get the result back out!"

Okay, true. The real point behind `Semigroup` isn't that it lets you add numbers. It's the other two functions. What were they again?

```haskell
class Semigroup a where
    (<>) :: a -> a -> a
    sconcat :: NonEmpty a -> a
    stimes :: Integral b => b -> a -> a
```

Right, `sconcat` and `stimes`. 

---

# The `sconcat` function

`sconcat` will take any `NonEmpty` list of a monoid, and convert it into a single value.

That's the real power of `Monoid`s. They represent a universal "foldable" type in which we don't have to specify associativity.

Remember with folding we needed to specify `foldl`, `foldr`, `foldl'`, etc.? We had to specify the associativity and strictness, because it was a meaningful distinction.

Sometimes it's not meaningful, and if something is a Semigroup, we can just flatten any non-empty list into a single thing. 

And what's more, we don't have to ask how to do it. We know how to do it. `Sum` is flattened with `+`. So `sconcat` is less work to call than `fold` et al.

So instead of `Semigroup`, think "naturally foldable without extra info"

---

# What is `NonEmpty`?

Which brings us to `NonEmpty`. What is the point?

`NonEmpty a` is basically a `[a]`, except that it has no empty constructor.

A `List a` has two constructors: `[]` and `a : [a]`. 

That is, we can either create an empty list or add the value to an existing list.

`NonEmpty` only has one constructor: `a :| [a]`
The `:|` is just like `:` but for `NonEmpty`. It means we must have an existing list (which can be empty) *and* a value to construct a `NonEmpty`.

Therefore, it is impossible for a `NonEmpty` to ever be empty, because we needed at least one value to construct it.

And when we destructure it, the resulting right side list might be empty, which is right.

---

# Using `sconcat`

We can use `sconcat` to sum some values:
`sconcat $ Sum 10 :| [Sum 20] == Sum {getSum = 30}`

We can use the same function to multiply too:
`sconcat $ Product 10 :| [Product 20, Product 30] == Product {getProduct = 6000}`

And if we get tired of writing `Product`, we can use `map`, right? Not quite, we haven't learned `Functor` yet, and `map` only works for regular lists, not `NonEmpty` (we can do this though: `sconcat $ fmap Product $ 10 :| [20, 30]`)

We can also use `sconcat` to concatenate strings, without any extra work:
`sconcat $ "hey" :| ["there", "hi", "there"] == "heytherehithere"`


---

# Why `NonEmpty`?

So why do we have to use this janky weird list instead of a regular list? Why can't `sconcat` just take a normal list? Who cares if the list is empty?

Because `Semigroup` doesn't have a "default" value. What should the sum of an empty list be? You might assume it would be `0`, and that makes sense, but what about the product? In that case, it probably makes more sense for it to be `1` (because `1` means the same thing as "do nothing" when multiplying)

Because it's not obvious, `Semigroup` doesn't make a distinction. Not in Haskell and not in mathematics: semigroups in math are not required to have an identity element.

And that's annoying, because if we had an identity element, we'd have a useful value to use if the list were empty, and then we wouldn't have to use `NonEmpty`.

---

# Monoid

A semigroup which also has an identity element is called a *Monoid*, both in Haskell and in general mathematics.

There is a `Monoid` typeclass just like you'd expect:
```haskell
class Semigroup a => Monoid a where
    mempty :: a
    mappend :: a -> a -> a
    mappend = (<>) -- by default, `mappend` is just the Semigroup operator
                   -- don't override it: that makes sense.
    mconcat :: [a] -> a
```

---

# Typeclasses can require a global value

First, this illustrates a cool thing about typeclasses in Haskell.

Notice that `mempty` is a constant. The typeclass can say "there needs to be a value named `mempty` somewhere". 

This is a rare and powerful feature. In Java, we can't use inheritance to require the existence of a value (only of a function that produces a value). 

Also, now that we have an `mempty`, we don't need the list to be nonem

---

# What about times?

---

# Monoid laws


---

# Monoid practice



---

# Questions?

<!-- _class: invert questions -->


---

# Finally functors

---