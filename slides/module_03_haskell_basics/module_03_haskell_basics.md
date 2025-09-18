---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 3: Intro to Haskell

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# Last Module

We learned about Lambda Calculus.

Lambda Calculus is a model for computation that treats every operation as either creating a function or calling it. 

Lambda calculus seems weird, but hopefully it became clear that you could do anything in it. 

Now it's time to put it to use...

---

# This module

We're going to start learning Haskell.

Haskell is a pure functional programming language.

It has several weird features that you have never seen before. 

So, without further ado...

---

# History of Haskell

Haskell was designed by a committe to specifically be a Lazy functional programming language. (We'll explain what this means later).

This has a specific technical meaning. It doesn't mean that it's chill. It's actually way weirder than that.

There are several Haskell compilers, but the main one is [GHC](https://www.haskell.org/ghc/), the Glasgow Haskell Compiler, named after the University of Glasgow where it was developed.

---

# Basic programming language facts

Haskell is (usulaly) a compiled language, meaning that we run the compiler on Haskell source code to generate an executable that can be run later.

Haskell is statically typed, meaning that the types of variables and expressions is known at compile time ("statically" means at compile time, "dynamically" means at run time).

Unlike C or other languages you may be familiar with, Haskell is implicitely typed. It is good at figuring out the types of expressions without you having to write them. Writing them is usually optional (but still a good idea for public functions).

Haskell is *not* object-oriented. The class keyword is unrelated to Classes in an OO language. Instead, we organize our programs around functions, instead of classes.

---

# Functional programming

What is functional programming?

Fundamentally, functional programming means that your computer program is a function. The functionall programming paradigm treats all programs as functions.

But wait, does that mean C is a functional language? After all, the program starts with the `main` function!

No, because `main` (usually) is not actually a function. At least, a mathematician would not consider it to be a function.

So what is a *real* function, then?

---

# Real functions are *pure*

Functions that we care about in this class are *pure* functions.

A pure function has two properties:
1. Referential transparency. That is, every time the function is called with a particular argument, it should give the same result. The functions output should depend only on its input (and maybe constants).
2. Lack of side effects. That is, calling the function should not change anything elsewhere. This may sound extreme: it is! Variables aren't allowed to change.

Let's examine these two properties.

---

# Referential transparency

In math, suppose I define a function: $f(x) = 2x$

So $f(2) = 4$, and $f(7) = 14$

Those are not temporary values, they are definitional. $f(7)$ is $14$ because $14$ is $2 \times 7$. 

This would make no sense:
$f(7) = 14; f(7) = 15; f(7) = 16$

A mathemetician looking at that would not accept it. "Which is it? Is $f(7)$ 14, 15, or 16?"

The term "referential transparency" can be hard to remember, so let's see where it comes from...

---

# Referential transparency (2)

Discussion from an [interesting stackoverflow post](https://stackoverflow.com/questions/210835/what-is-referential-transparency#9859966) (top response)

The term comes from analytic philosophy. The idea was to determine which kind of verbs could simply have their "referrents" (the things they were about) replaced transparently.

For example, in the phrase "Football is watched by millions of Americans every year", "Football" can be replaced by "America's most popular sport" without altering the meaning of the sentence.

However, in the phrase "I watched the football game", we cannot say "I watched the America's most popular sport game."

So "is watched by" is referentially trasparent here, but "I watched the" is not.

---

# Referential transparency (3)

This idea was adapted to functional programming languages by treating arguments to functions as "referrents".

A referentially transparent function is one in which all the references to its arguments can be replaced by the argument.

So that function from earlier, $f(x) = 2x$ is referentially transparent, becuase $f(7)$ can be rewritten as $f(7) = 2(7)$ without getting something wrong.

Can anyone think of a function that is *not* referentially transparent.

---

# Referential transparency (4)

`input()` in Python is one example of a non-referentially transparent function.

Sometimes, `input()` returns "Hello". Other times, it returns "Bob". It returns whatever string is in standard input at that time.

But in Haskell, this would not be possible. It has to always return the same thing given the same arguments.

And since `input` has no arguments, that means it has to always return the same thing. It would have to be a constant.

Another example: in C, `scanf` returns the number of characters it read. That's going to depend on what's in standard input.

---

# No side effects

The other aspect of pure functions is that they have no *side effects*.

A side effect is any kind of IO or variable change.

So this means, a pure function never modifies a variable. It also never writes to a file, inserts into a database, or sends an internet packet.

Haskell *only* has pure functions.

So how does it do any of those things? Can you just not write database programs in Haskell?

No, you can, you just have to change how you think about it. Don't worry, we'll see some examples of writing useful software in Haskell. There are [games](https://wiki.haskell.org/Applications_and_libraries/Games), [office software](https://pandoc.org/MANUAL.html), and even a [Linux window manager](https://xmonad.org/) written in Haskell.

---


# Questions?
<!-- _class: invert questions -->


---

# Lazy Evaluation

Almost every programming language you have ever used, is *eagerly* or *strictly* (both mean the same thing) evaluated.

What does that mean?

It means that expressions are evaluated in the order they are arrived at.

That seems to straightforward that it's impossible to imagine anything different, so here's a concrete example.

---

# Eager evaluation in C

C is a (mostly) eagerly evaluated language.

Consider this C code:
```c
int main() {}
    int x = 8 + 8;
    int y = 4 + 4;
    printf("x is %d\n", x);
    return 0;
}
```

What does it do?

It first computes 8 + 8, and stores the result in a variable called `x`. It then computes 4 + 4 and stores the result in an unused variable `y`. It then prints out x. Finally, it exits with a code of 0. 

---

# How else could it possibly work?

Here is a similar looking bit of Haskell code:

```haskell
main = do
    let x = 8 + 8
    let y = 4 + 4
    print x
```

What does it do?

It computes 8 + 8 and binds the result under the name `x`. Then it prints `x`.

It *does not* compute `y`.

Why? Because it doesn't matter. You never print it, so it is never used.

---

# How does it know?

Fundamentally, Haskell evaluates expressions from the "other direction". 

Whatever actions are present in main need to happen to matter what. So it sees `print x` and realize that it needs to compute x at that point.

Then it works backward and actually computes it, which means 8 + 8.

Nothing ever forces it to compute `y`, so it never does.

There was an old Haskell benchmark that was supposed to compute digits of pi or something. The benchmark creator forgot to print, so briefly Haskell was the fastest language in the world, because the program ran in 0 seconds.

---

# Why?

Lazy evaluation isn't actually common, even in functional programming languages.

Apparently, it was chosen as a way to *force* Haskell to be functionally pure.

If you can't rely on a particular order of computation, you can't have side effects.

It can be faster under certain circumstances. If there's some variable that is expensive to compute and only occasionally necessary, you can save the cost if it isn't used.

It also means you can easily have infinite data structures without iterators. Nothing stops you from creating a list of all integers, because each element is created as it is read in the list, and not all at once.

---

# Why not?

If lazy evaluation is so great, why isn't it more common?

Unfortunately, it makes memory usage very unpredictable.

When haskell sees an expression, it creates a special value that kind of encodes the computation without running it (called a "thunk"). 

If the expression is a large or infinite list, as you pull data from it, you end up allocating a lot of memory all at once.

It's possible to disable this behavior, but the fact that it's the default means that software that needs to be easily speed/memory audited tends to not be written in Haskell.

---

# Questions?

<!-- _class: invert questions -->

---

# So what about "print"?

But now, let me address a question you may have: if functions are all pure and have no side-effects, what does this mean?

```haskell
main = putStrLn "Hello, World"
```

It's clearly printing! It's doing something! How can you say that all functions in Haskell are pure?

---

# I didn't lie!

That function has no side effects.

"But it prints!" No it doesn't!

That function actually does not print anything.

Consider the strange syntax. Notice that we write: `main = ...` and not `main { ... }` or `main: ...`. Why is it an equals sign?

Because it is a mathematical definition. We are saying that main is a constant that contains the value `putStrLn "Hello, World"`.

But `putStrLn` is a function, not a value, right?

---

# It's a value

Let's write the type above the program:
```haskell
main :: IO ()
main = putStrLn "Hello, World"
```

You've already seen types in your readings so far. We write them with `::`. This is saying that `main` is something called an `IO`, and then there's this `()` next to it.

`IO` is a kind of *monad*, a concept we will learn in detail much later in the course. 

However, we can think of an `IO` as a kind of program. An `IO` is a program that performs input and output when it is run.

---

# When does it run?

When is it run? Not by us! We are returning it from `main`. The Haskell runtime is going to actually run it.

So *in Haskell*, every function is pure. But the function can return a program, and a different system will run that program. It can have side effects.

In fact, it probably *should* have side effects. Otherwise you could never print anything!

The `()` after `IO` is the return type of the program. In this case, the program doesn't return anything. `()` is pronounced "unit", and it represents a type of value that is irrelevant or for which there is only one possibility (like "nothing"). It's kind of like null, but it's not only a value, but also a type.

`IO Int` would be an `IO` program that returns an `Int`. But main must be `IO ()`.

---

# Re-iterating that

In Haskell, *main is not a function*.

It is a constant. It is a program that never changes. Every time the runtime runs your program, it will reach inside and pull the program out of the `main` variable, and then run it.

The program is allowed to have side effects. It can print and read and open files. 

However, `main` will always return the same program. Even if the program has side-effects, `main` is not running the program, only returning it.

This is the mindset shift that we need to do in order to really *get* functional programming. Nothing changes; everything is a map from input to output.

---

# The lazy evaluation strategy

Inside your program, Haskell doesn't actually do anything.

When you return a value as `main`, the runtime starts evaluating the program you returned. This is when it starts evaluating expressions, and lazy evaluation happens.

If `main` doesn't use a value as part of IO it never gets evaluated.

I'll prove it:
```haskell
main :: IO ()
main = do
    let x = putStrLn "hi"
    return ()
```

This doesn't print anything. `putStrLn` is not evaluated.

---

# How does outside code get called?

So how can someone call a C library from Haskell?

There is a way to do it. There are ways to call native programs.

There's also a function called `unsafePerformIO` that allows you to use a value computed from an IO action inside a pure function.

But it still doesn't actually run until the runtime's lazy evaluator makes it.

---

# Questions?
<!-- _class: invert questions -->

---

# Big changes from imperative programming

Okay, so functional programming is weird. But if I kind of look at that earlier Haskell code side by side with the C code, it kind of looks the same! What's the big difference?

Well, for one thing, you can't do this:
```haskell
x = 20
x = 10
```

That's not valid. `x` can only be one thing.

---

# Big changes (2)

You *can* do this:
```haskell
do
x <- return 20
x <- return 10
print x
```

But it doesn't actually change `x`, it creates a different value in a different scope. (we'll talk about the weird arrow later).

So that means we can't have loops!

---

# No loops

In C, a loop is a structure that executes a series of statements as long as a condition is true. Then it breaks when the condition fails.

In Haskell, this would never happen, because the condition would either be true or false, and that would never change. Haskell does not have C-style loops.

(There are alternatives, including things like for-each, but they involve those monad things, so let's not worry about it for now)

So what do we do if we can't loop? Surely there has to be some kind of way? [What's the alternative?]

---

# Recursion instead

Instead we use recursion.

Let's write a simple loop over a list to compute the sum. 
This already exists, but it's a simple exercise.

Here's how we'd do it in C
```c
void sum(int* arr, size_t n) {
    int sum = 0;
    for(size_t i = 0; i < n; i++)
        sum += arr[i];
    return sum;
}
```

But in Haskell...

---

# Lists

```haskell
sum'' :: [Int] -> Int
sum'' list =
    if list == [] then 0
    else head list + sum'' (tail list)
```

This is a recursive function. It takes a list of ints and returns an int.

It first checks if its argument is empty. If it is, the sum is zero.

Otherwise, it returns the head of the list plus the sum of the tail.

Take a second to convince yourself it works, and consider that we just "looped" without modifying anything.

---

