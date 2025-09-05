---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 2: Lambda Calculus
### The foundation of functional programming

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# Last Module

We learned about how class is going to work

We learned what a formal language is.

We learned some programming language history.

We learned about programming paradigms.

We learned about one important paradigm in particular [which one]?

---

# This module

In this module, we'll be learning the fundamental model of computation behind functional programming: lambda calculus.

This has nothing to do with derivatives and limits, *calculus* is a generic term that refers to systems of calculation, and we are going to learn one.

Why? Let's talk about that

---

# Models of computation

What *is* a computer?

I mean, this is a computer *points to lecture computer.

Many of you have laptops and tablets, I think we all agree that those are computers.

But what is the minimum thing that a device needs to be able to do to be called a computer?

Yes or no:
Is a rock a computer? Is an abacus a computer? Is a human a computer? Is a dog a computer? Is a bacterium a computer? Is a (biological) virus a computer? Is a car a computer? Is a tree a computer? Is an electron a computer? Is RAM by itself a computer? Is a CPU by itself a computer?

---

# My thoughts:

A rock is not a computer.
An abacus is not a computer.
A human *is* a computer (and a human *with* an abacus is certainly a computer).
A dog can be trained to compute.
A single bacterium can be a computer (DNA computer).
A virus probably too for the same reason.
A car is not a computer, but it has computers in it, so I guess if you include those.
A tree can respond to stimuli in different ways, so there's probably a way to make it compute, but I don't know how to do it.
An electron is not a computer.
RAM by itself is not a computer.
A CPU by itself is a computer if it has registers or some feedback mechanism.

---

# Okay, but what *is* a computer?

Broadly speaking, something that can carry out instructions.
Specifically, an [effective method](https://en.wikipedia.org/wiki/Effective_method).

But if we want to be more formal, we need to actually specify a model of computation.

A model of computation is a specification for how to carry out computations.

You already know a model of computation because you have taken automata theory (or theory of computation).

[What is it?]

---

# The turing machine

The good old turing machine.

The instructions for a turing machine are encoded as states and transitions.

We can encode pretty much any practical computer program as a turing machine, as long as we can agree on how to represent things like IO.

For example, if we agree that cells 10000 through 20000 represent the colors of pixels (and the alphabet is, say, 32-bit integers), we can code a game for a 100 * 100 screen and "run" it on a Turing machine.

---

# The turing machine (2)

In fact, the turing machine is the model of computation that underlies every imperative programming language that I am aware of.

Every time you modify a variable (or perform I/O), you are changing state.

Any time you have a conditional or loop, you are making a conditional transition.

Even though you aren't programming a turing machine directly, the design of the language is influenced by one. We think in terms of statements.

Every "statement" modifies state. We run code to move the computer from one state to another. Every program basically involves creating some kind of effect.

---

# Questions?
<!-- _class: invert questions -->

---

# A different option?

What if I told you, that's not the only option?

There is a *completely different* model of computation that works in a fundamentally different way.

And yet, it is isomorphic to turing machines. That is, every turing machine can be converted into an expression in this language and back again without losing any information.

This language is called lambda calculus.

It was invented by [Alonzo Church](https://en.wikipedia.org/wiki/Alonzo_Church) who was Alan Turing's Ph.D. advisor. 

---

# Lambda calculus


Lambda calculus is based around the concept of a lambda function.

You may already be familiar with lambda functions from programming languages like Python, although their use of the term lambda is slightly incorrect.

A lambda function is basically a function that takes one argument and has one return value. That's it.

It turns out defining and applying functions of one argument is all you need to compute every computable function.

That seems impossible. Don't we need like...at least to be able to take as many arguments as we want? It turns out, we can get that for free.

---

# Lambda calculus

Lambda is a formal language for expressing computations (functions). 

It actually can be a programming language, although just like a turing machine, you probably don't want to actually program in it.

It can express any computation a turing machine can express and vice versa. Any program you have ever written can be expressed as an expression in lambda calculus if you can agree on how to represent IO (same deal as turing machines).

It's main use is being a system that is so simple, you can't really remove a rule without breaking its turing equivalence. 

So let's see the rules...

---

# Rules of lambda calculus

A string is a lambda expression, also called a *term* if it matches one of these patterns

1. $x$: A string representing a variable is a term. It doesn't have to be $x$. It could be $y$ or $z$, or, even a string like $\textrm{hello}$.
2. $\lambda x. M$: a lambda abstraction, where x is a parameter variable and M is a term. This is how we define functions, which we call the process of "abstraction". Abstract here really means "simple" or "minimal", so we're saying "I'm going to produce the result value M if you give me one additional piece of information" (which is $x$).
3. $M N$: two terms next to each other means to pass the one on the right to the one on the left as the argument. (I say *the* instead of *a* because there is always exactly one parameter for every function)
4. $(\lambda x. M)$ and $(M N)$ are terms, too. You can use parentheses to control order.

---

# Lambda examples

$\lambda x. x + 1$ is a function that takes a number and adds 1 to it.

The one parameter is named $x$, and the one return value is 1 + that.

We can mix lambda expresions and mathematical definitions. For example:
$f = \lambda x. x + 1$
$8 = f \; 7$

Before we move on, let's have some abstraction practice.

---

# Define these functions

1. A function that takes a value and doubles it.
2. A function that takes a value and ignores it and returns 0 instead.
3. A function that computes the square root of a value.

You can use ordinary math notation for the operations like doubling and square roots. You don't have to use numerical techniques or anything to get the square root.

---

# Abstraction answers

1. $\lambda x. x + x$ or alternatively $\lambda x. 2x$
2. $\lambda x. 0$
3. $\mathrm{sqrt}$
(it's already a function that takes one value and returns one value, but you can say $\lambda x. \sqrt{x}$ and that's right, too)

---

# What about application

Rule 2 was abstraction. That just means defining a function.

Rule 3 is application. Suppose we define this function: $\mathrm{add7} = \lambda x. x + 7$

How do we use it? We just put the function next to the argument we want:
$\mathrm{add7} \; 8=15$

Lambda functions are *values*, so we can just substitute the actual definition for the name of the variable we put it in:
$(\lambda x. x + 7) \; 8 = 15$

---

# Abstraction and application practice

1. Define a lambda function to triple a number and apply it to 7 in a single expression.
2. Define a lambda function to concatenate "llo" to the end of a string and apply it to "he" in a single expresion. Use `++` as the concatination operator.
3. Define a lambda function that always returns 0, and a lambda function that always returns 1, apply them to get those values, and add the results together.

---

# Abstraction and application answers

1. $(\lambda x. 3 \cdot x) \; 7$
2. $(\lambda s.s \mathrm{++} \text{"llo"}) \text{"he"}$
3. $((\lambda a. 0) \; 7) + ((\lambda a. 1) \; 200) = 1$

---

# Questions?

<!-- _class: invert questions -->

---

# What does 7 mean?

I ha