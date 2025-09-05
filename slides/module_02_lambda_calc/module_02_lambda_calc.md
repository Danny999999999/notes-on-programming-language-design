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

It can express any computation a turing machine can express and vice versa. Any program you have ever written can be expressed as an expression in lambda calculus if you can agree on how to represent IO (same deal as turing machines). That is why it is called a model of computation.

Its main use is being a system that is so simple, you can't really remove a rule without breaking its turing equivalence. 

So let's see the rules...

---

# Rules of lambda calculus

An expression is a valid *lambda term* if it matches one of these patterns

1. $x$: a variable by itself is a term. It doesn't have to be $x$. It could be $y$ or $z$, or, even a string like $\textrm{hello}$.
2. $\lambda x. M$: a lambda abstraction, where x is a parameter variable and M is a term. This is how we define functions, which we call the process of "abstraction". Abstract in general math means "simple" or "minimal", so we're saying "I'm going to produce the result value M if you give me one additional piece of information" (which is $x$).
3. $M N$: two terms next to each other means to pass the one on the right to the one on the left as the argument. (I say *the* instead of *a* because there is always exactly one parameter for every function)
4. $(\lambda x. M)$ and $(M N)$ are terms, too. You can use parentheses to control order.

---

# Lambda examples

$\lambda x. x + 1$ is a function that takes a number and adds 1 to it.

The one parameter is named $x$, and the one return value is 1 + that.

We can mix lambda expresions and mathematical definitions. For example:
$f = \lambda x. x + 1$
$8 = f \; 7$

We have to agree on what operators like "+" mean to do this, though, which we'll talk about later (they aren't actually part of lambda calculus).

Before we move on, let's have some abstraction practice.

---

# Define these functions

1. A function that takes a value and doubles it.
2. A function that takes a value and ignores it and returns 0 instead.
3. A function that computes the square root of a value. This one doesn't have to be a lambda abstraction.

You can use ordinary math notation for the operations like doubling and square roots. You don't have to use numerical techniques or anything to get the square root.

---

# Abstraction answers

1. $\lambda x. x + x$ or alternatively $\lambda x. 2x$
2. $\lambda x. 0$
3. $\mathrm{sqrt}$
(it's already a function that takes one value and returns one value, but you can write it as an abstraction by saying $\lambda x. \sqrt{x}$ and that's right, too)

---

# $\alpha$ conversion

Old-fashioned lambda calculus didn't have a notion of "scope". Variables just weren't supposed to conflict with each other.

There is a *reduction* we can perform to rename variables and ensure this. A reduction is just the process of taking a lambda term and simplifying or changing it. 

In this case, $\alpha$ conversion (read: "alpha conversion") means changing the name of the lambda parameter consistently throughout the whole term, as long as the new name does not conflict with another parameter.

It's like using the "rename" refactoring in your favorite IDE.

---

# $\alpha$ conversion (2)

For a simple lambda function with one parameter, we just change the name.
$\lambda x. x + x$ can become $\lambda y. y + y$, and
$\lambda x. x + x + x$ can become $\lambda y. y + y +y$

But we can not use alpha conversion to turn $\lambda x. x + y$ into $\lambda y. y + y$, because we were already using the variable y (from an external definition or outer parameter that I'm not showing)

And this is good, because $\lambda x. x + y$ is a function that adds some external fixed value $y$ to the given value $x$, whereas $\lambda y. y + y$ just doubles the given value. They are completely different functions.

---

# $\alpha$ equivalence

If two lambda terms would be the same after an $\alpha$ conversion, we call them alpha equivalent.

This is one way two lambda functions can be "equal". If they are literally the same function except their parameter is named differently, then we know they will meet the looser requirements for extrinsic functional equivalence (meaning, they will give the same output values for the same input values)

Functions can be equivalent without being $\alpha$ equivalent, but if they are $\alpha$ equivalent then you know they are equivalent.

---

# $\alpha$ practice

1. Perform an $\alpha$ conversion to this term: $\lambda x. x / 2$
2. List an invalid $\alpha$ conversion for this term: $\lambda x. x / b$ and explain why it is invalid.
3. List a lambda term that is alpha equivalent to $\lambda x. x + b$
4. Are $\lambda x. x * y$ and $\lambda z. z * y$ alpha equivalent? Why or why not?

---

# $\alpha$ answers

1. $\lambda x. x / 2$ can become $\lambda q. q / 2$
2. $\lambda b. b / b$ would be an invalid $\alpha$ conversion. It would change the function from dividing by b to the constant value 1 (or not a number if applied to 0).
3. $\lambda \mathrm{bloop}. \mathrm{bloop} + b$ is $\alpha$ equivalent to $\lambda x. x + b$
4. Yes, because we can apply an alpha conversion from $x$ to $z$ without conflicting with any other variable.

---

# What about application

Rule 2 was abstraction. That just means defining a function.

Rule 3 is application. Suppose we define this function: $\mathrm{add7} = \lambda x. x + 7$

How do we use it? We just put the function next to the argument we want:
$\mathrm{add7} \; 8=15$

Lambda functions are *values*, so we can just substitute the actual definition for the name of the variable we put it in:
$(\lambda x. x + 7) \; 8 = 15$

This whole system is built around functions, so the easiest thing to do is call a function. You just write the function next to the value. There is no special syntax like $f(7)$, you just write $f \; 7$. You *can* write $f(7)$ if you, want, but the parentheses aren't needed.

---

# Application in detail

So how do we actually *do* the application? We apply a new reduction: $\beta$ reduction.

$\beta$ reduction means replacing an abstraction's parameter with a given value. This means applying a single term to another term.

So for example:
$(\lambda x. x + 1) \; 7$  after beta reduction is $(7 + 1)$

Notice that we delete the header of the term, (the $\lambda x.$ part) and we replace every $x$ with the argument $7$.

So, to perform a single application, just do one round of $\beta$ reduction.

---

# Abstraction and application practice

1. Define a lambda function to triple a number and apply it to 7 in a single expression.
2. Define a lambda function to concatenate "llo" to the end of a string and apply it to "he" in a single expresion. Use `++` as the concatenation operator.
3. Define a lambda function that always returns 0, and a lambda function that always returns 1, apply them to get those values, and add the results together.

---

# Abstraction and application answers

1. $(\lambda x. 3 \cdot x) \; 7$
2. $(\lambda s.s \mathrm{++} \text{"llo"}) \text{"he"}$
3. $((\lambda a. 0) \; 7) + ((\lambda a. 1) \; 200) = 1$

---

# Function call?

Because imperative programming is based on changing state, when we *call* a function, we store the return address, jump to the function's address, and begin executing its instructions.

When we *apply* a function in lambda calculus, we just replace all instances of a term with a variable. Nothing actually *changes* outside the function. 

This is not a coincidence. Functional programming doesn't *allow* you to change things outside of a function.

---

# Questions?

<!-- _class: invert questions -->

---

# There is no way you only need one argument

I mean come on.

*Surely* we can't just only have functions of one argument? I mean, you used concatenation earlier! That takes two arguments!

You're right. We often need more than one argument. But it turns out, you can build functions of more than one argument out of functions with exactly one argument and this does not require bending the rules of reality.

It does require bending our brains a little, though...

---

# Currying

Suppose addition is defined, but we want to create a lambda term that does it.

We can write this:
$\lambda x. \lambda y. x + y$

This is a function that takes one argument, and returns a new function that takes another argument, that then returns the result of adding the two together.

This is confusing at first, so let's spend a moment to ponder it...

---

# Currying (2)

Let's try to add $7$ to $8$ using this lambda term. We write it like this:
$(\lambda x. \lambda y. x + y) \; 7 \; 8$

Here, there are two arguments with a space between them. Is there a rule for that? No, it actually follows from $\beta$ reduction. 

Apply $\beta$ reduction once. Replace every $x$ with a $7$:
$(\lambda x. \lambda y. x + y) \; 7 \; 8 = (\cancel {\lambda x.} \lambda y. 7 + y) \; 8$
$(\lambda y. 7 + y) \; 8 = \cancel {\lambda y.} 7 + 8=15$

---

# Currying (3)

The process of representing a multi-parameter function as a series of single-parameter function definitions is called "currying", after American mathematician, [Haskell Curry](https://en.wikipedia.org/wiki/Haskell_Curry).

Curry used it extensively in his work, but the original idea was originated by Soviet mathematician [Moses Schönfinkel](https://en.wikipedia.org/wiki/Moses_Sch%C3%B6nfinkel) 6 years earlier, so sometimes people call it "Curry-Schönfinkeling".

Curry is so famous and influential, all three of his names were also used to name programming languages, including the one we're using in this class.

---

# Currying (4)

The big idea behind currying is that the outer function takes a parameter, which becomes *bound*, and returns a family of functions that use that parameter.

So if we stop at $(\lambda x. \lambda y. x + y) \; 7$, we get $\lambda y. 7 + y$, which is a function that adds 7 to things.

$(\lambda x. \lambda y. x + y)$ is a function that creates a function that adds $x$ to whatever its argument is.

---

# Currying Practice

This can be a little challenging to understand, so let's do some drills:

1. Define a term that has 3 parameters and adds them together.
2. What is the result of partially applying that term on the values 7 and 8? (we say partially applying because we haven't supplied all 3 arguments).
3. Is it meaningful to only give that function 2 arguments? Why?

---

# Currying answers

1. $\lambda a. (\lambda b. (\lambda c. a + b + c))$ or equivalently, $\lambda a. \lambda b. \lambda c. a + b + c$
2. $(\lambda a. \lambda b. \lambda c. a + b + c) \; 7 \; 8 =\lambda c. 7 + 8 + c =\lambda c. 15 + c$
3. Yes, it is meaningful because the result is a function that adds 15 to things.

Note: it doesn't matter with the addition example, but lambda application is *left associative*. That means we apply the arguments from left to right.

---

# Currying notation

Currying is so common, that there's special notation for it.

Instead of writing:
$\lambda a. \lambda b. \lambda c. a + b + c$
You can write:
$\lambda a b c. a + b + c$

Those are equivalent. That's the main reason we have a $.$ in the definition. 

If you need long variable names, you can separate them with spaces:

$\lambda \mathrm{blip} \; \mathrm{blop} \; \mathrm{blorp}. \mathrm{blip} + \mathrm{blop} + \mathrm{blorp}$

**This is just shorthand**. $\lambda a b c. a + b + c$ is still just a 1 argument function that returns one result. It just so happens that that result is another function, etc.

---

# Questions?

<!-- _class: invert questions -->


---

# Why did I have to learn that?

Because in many functional languages, including Haskell, every function takes one argument. 

Any multi-argument function is typically written curried. 

This allows some really cool tricks to be performed, although we'll have to wait to see them. 

But just to give you an idea...

---

# Why did I have to learn that? (2)

If I want an increment function, I can write it like this:
```haskell
inc = (1 +)
```

Which means "incrementing is the same as adding 1 to something". 

Addition isn't special in Haskell. But it takes two arguments, and we're supplying the first one, so the result of doing that is a function that takes one argument and adds 1 to it. This is partial application in action, and currying makes it easy.

---

# The proof is in the Python

You might think "okay, this is some fancy functional stuff", but really it has more to do with the basic operation of anonymous functions / lambdas / closures.

You can do it in python:
```python
f = lambda x: lambda y: lambda z: x + y + z
add15 = f(7)(8)
print(add15(9)) # prints 24
print(f(7)(8)(9)) # also prints 24
```

Really, you can do this in any language that supports anonymous closures. A closure is just a function that is allowed to use variables from its containing scope (environment).

---

# Why haven't I seen that before?

The reason that snippet looks strange is that currying is not part of Python's culture. Python is an imperative language, and it prefers multi-parameter functions. Its "lambda" keyword can actually declare anonymous functions of many variables, so its name is somewhat misleading.

The reason we need several groups of parentheses is that we aren't calling one function on `7, 8, 9`, we're calling a series of functions:

```python
a = f(7)
b = a(8)
c = b(9)
# c is now 24. a and b are lambda functions.
```

But in functional languages, currying is the default way to have multiple parameters.

---

# Questions?

<!-- _class: invert questions -->

---

# What does 7 mean?

I have been using expressions from general math that aren't actually lambda terms according to our definition.