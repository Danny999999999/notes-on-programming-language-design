---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 4: Data types

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# Data types

We learned about simple built-in datatypes last lecture. This time, we're going to learn how they work, and how you can make your own.

Obviously, you aren't going to make your own `Int`. Haskell has you covered there. There's also `Int8`, `Int16`, etc.

But, datatypes are a little different in functional languages. There are the equivalent of structs, sure, but also enums. So let's talk about those...

---

# Enums in C

Enums in C let you list every value a type can have.

They are also a way to create constants:
```c
typedef enum {
    FALSE,
    TRUE
} MyBool;
```

Here, a MyBool is supposed to have one of two values:

```c
MyBool a = TRUE; // fine
MyBool b = FALSE; // also fine
MyBool c = 15; // may emit a warning, although technically permitted
```

---

# Enum variants

The items inside the enum are called "variants" in C. 

The compiler creates a constant for each variant.

So now FALSE and TRUE are defined. 
By default, the first variant gets value 0 and the second gets 1

So you can do this:
- `int x = FALSE; // x is 0`
- `x = TRUE; // now it is 1`

But you can also do this:
`int x = 20;`, which you probably don't want...

---

# Enums give some safety

Suppose we add another enum:
`typedef enum { RED, GREEN, BLUE } MyColor;`

The compiler can help you catch errors if you actually use the enum wrongly:
```c
MyBool a = FALSE; // okay
MyBool b = RED; // generates a warning
```
`b` is a `MyBool`, but we’re setting it to a `MyColor`

Behind the scenes `RED == 0` and `FALSE == 0`, so if we were using constants, there would be no warning that we were setting the wrong enumeration value.

[Important: does it make sense how we need enums here to detect the error, and how regular constants aren't enough?]

---

# Setting the values of variants

One last thing, just to make sure you’ve seen it:

You can set the initial value or all the values if you want:
```c
typedef enum {
    FALSE = 0,   //set the initial value 
    TRUE 	 //this value is now 1 automatically.
} MyBool;

typedef enum { RED = 20, GREEN = 21, BLUE = 22 } MyColor;
```

It’s like setting a bunch of constants but you’ll be warned if you use one in the wrong place (like assigning a bool to a color).

---

# Questions?

<!-- _class: invert questions -->

---

# Why do Enums exist?

They are actually extremely common in actual professional code, but you may not have seen them much in your studies so far.

Enums are very common as a way to describe states or commands.

For example, a webserver has certain `modes` for connections, where for each one it might be parsing requests, authenticating, or sending data.

And games have states, too...

---

# Gamestate enum

Suppose we’re making a basic video game in C.

It’s common to implement states as enums:
```c
typedef enum {
    STATE_SPLASH_SCREEN = 1, //skip state 0, maybe use it to debug
    STATE_MAIN_MENU,
    STATE_CHAR_SELECT,
    STATE_IN_GAME,
    STATE_LOADING
} GameState;
```
How do we use them?

---

# States in old-school C

Typically there’s some kind of if-statement or switch where we do something depending on which state we’re on:

```c
if( current_state == STATE_SPLASH_SCREEN )
    update_splash_screen( time_remaining );
else if( current_state == STATE_MAIN_MENU )
    update_main_menu();
else if( current_state == STATE_CHAR_SELECT )	
    update_char_select( secret_chars_unlocked );
else if …
```

(protip, if you start your states at 0 or close to 0, you can then use the state variable as an index into an array of function pointers.

`GameState state_impls[] = { &DoDebug, &DoSplash, &DoMenu, … }; )`

---

# Explanation

Here, we're using an enum to describe all the states we could be in.

When we're in a state, we do some unique action. I'm glossing over that part.

The only issue is, the data the state needs to access is probably global with this approach. Not the end of the world, but not always good. For example, in the webserver example, there might be thousands of connection objects.

It's possible to attach each state enum to a void* data. This is called a "tagged union". But it's not 100% typesafe: we have to make sure to always interpret the `void*`...

```c
typedef Connection {
    ConnectionState state; // CONNECTING, SENDING, PARSING, etc.
    void* data; // unique data for each state.
}
```

---

# Questions?

<!-- _class: invert questions -->

---

# Back to Haskell

So, what about our game we're writing in Haskell?

Haskell doesn't have `void*`, (technically, it's possible but not recommended). 

Instead, it has typesafe ways of doing this.

We want to associate data for each state:
- For the splashscreen, maybe how much time has passed so we can fade in.
- For the main menu, no extra data.
- For character select, which extra characters have been unlocked.
- For the in-game state, probably a lot of data. Which characters were selected, their health and meters, the time left, the stage, etc.
- For the loading state, what data are we loading?

---

# Back to Haskell (2)

At first glance, Haskell seems like it has an exact substitute for enums.

We use `data` to define a new datatype...

```haskell
data GameState = 
        Splash	
    |  	MainMenu
    |   CharSelect	
    |   Loading	
    |   InGame
```

Use the vertical bar `|` to separate variants.

We can then define a variable of this type:
```haskell
startingState :: GameState
startingState = Splash
```

----

# Differences between C and Haskell enums/datas

You don’t define the actual number that each variant takes. (that’s hidden by the compiler)

You separate with `|` instead of `,`

We use initial camel case instead of all-caps snake case. 

In Haskell you *may not* mix variants from other types.

Suppose `f :: GameState -> Bool`
This is allowed: `f InGame`
This is forbidden: `f Red`

Haskell is very strictly-typed here. The variants are not just constants. They are fully unique values that can't be mixed with incompatible types.

---

# Constructors

In fact, Haskell does not call these *variants*...

It calls them **constructors**!

Yes, the same term as OO, but used differently.

Consider this datatype: `data Rgb = Red | Green | Blue`

Here, `Red`, `Green`, and `Blue` are all constructors in Haskell.

They are ways of constructing data. If you write `Red`, you have constructed an `Rgb`. Likewise for `Green` or `Blue`.

---

# Knowledge check 1

1. Define a type with two constructors.
2. Define a different type with four constructors.
3. Define a function and its type that takes the first type and returns the second.
4. Call the function. What is the result?

---

# Knowledge check 1 answers
1. `data Blip = blip | blop`
2. `data Foo = foo | bar | baz | quux`
3.
```haskell
blorp :: Blip -> Foo
blorp blip = foo
blorp blop = baz
```
4. `blorp blip`. The result is `foo`.

---

# Questions?

<!-- _class: invert questions -->

---

# Real data types

Now we get to something that is clasically confusing. We're going to learn about true tagged unions in a functional language.

This feature isn't just in functional languages anymore. Rust has it for example. It's a nice way of implementing datatypes.

You see, these data types don't just have to be distinct constructors. Each constructor can also carry different data.

For example, what if I want a data type to represent single data values, *or* pairs of values?

We can still do that, and this is where Haskell's data types can do something that a basic C enum cannot do...

---

# Single or Double example

```haskell
data SingleOrPair =
        Single Int
    |   Pair Int Int
```

Here we've created a new datatype with two constructors: one for single ints, and one for pairs of ints.

`SingleOrPair` is a type. `Single` and `Pair` are constructors, not types.

These two constructors require either one `Int` or two in order to be called.

This is weird, so let me show you how this is used.

---

# Single or Double example

Let's say I want to define a Single or pair: 
```haskell
s :: SingleOrPair
s = Single 20 -- this is fine

t :: SingleOrPair
t = Pair 30 40 -- also fine
```

A variable of type `SingleOrPair` can be filled with either a `Single` or `Pair`.

---

# Constructors are not types

We can think of `Single` and `Pair` as different families of values. A `SingleOrPair` can either be a `Single` or a `Pair`. However `Single` is not a type, and neither is `Pair`. 

You *cannot* do this: `s :: Pair`
because the thing after the `::` is expected to be a type, but `Pair` is a constructor.

Think of `Single` and `Pair` as functions, rather than types, and this will make sense.

But with the `data` keyword, we are expected to exhaustively define all the ways of creating the value.

---

# Datatypes are exhaustive

This is an important point. When you create a data type in Haskell, you must list *all* of its constructors. 

This means, Haskell knows at that point, all of the possible values it could take.

But how do we get the values out? Suppose I have a `Pair 20 30`? How do I get that `20` and `30` back out of it?

---

# Pattern matching

Pattern matching is the main, basic way we get data out of data structures.

Not only in Haskell, but it's fairly common in many functional languages.

One simple way to do it is using case expressions...

---

# Case expressions

This Haskell code will take a `SingleOrPair` and either print the value if it's a single, or print the sum of both values if it's a pair:

```haskell
main = do
    let x = Pair 20 30
    case x of
        Single y -> print y
        Pair y z -> print (y + z)
```

(remember that `let` expressions don't have an `in` clause inside of `do` blocks. We'll talk more about `do` blocks when we get to monads.)

Here, we're using `case` to consider all the possible values that `x` could have. Crucially, *Haskell will warn us if we miss one!* Something that C enums can't always do.

---

# Pattern-matched definitions

Alternatively, we can define functions on particular constructors:

```haskell
singleOrSum :: SingleOrPair -> Int
singleOrSum (Single x) = x
singleOrSum (Pair x y) = x + y
```

This is saying: `singleOrSum` is a function that, when it receives an `Int` named `x`, it just returns `x`, but when it receives a `Pair` of `x` and `y`, it returns `x + y`.

This works exactly the same as pattern matching on basic values like `Int`:
```haskell
fibo 0 = 0
fibo 1 = 1
fibo n = fibo (n - 1) + fibo (n - 2)
```

Here, `Pair 20 30` is just as much a value as `1`.

---

# Knowledge check 2

1. Rewrite that case expression so that there is only one `print`. We're repeating code unecessarily.
2. Define a data type called `NilOrTriple` which has a constructor `Nil` which takes no arguments, or a triplet of Ints.
3. Write a function that returns the second value if it is a `Triple`, or `0` if it is a `Nul`. Include its type.

---

# Knowledge Check 2 answers

1. `print (case x of Single y -> y ; Pair y z -> y + z)`
   (note: we can use semicolons instead of newlines)
   Make sure this one makes sense: `case` is an expression, not a statement. It has a value that can be substituted for the argument of `print`.
2. `data NilOrTriple = Nul | Triple Int Int Int`
3. 
```haskell
secondOrZero :: NothingOrTriple -> Int
secondOrZero x = 
    case x of Nil -> 0 ; Triple _ y _ -> y

-- or, this is nicer imo vvv
secondOrZero Nil = 0
secondOrZero (Triple _ y _) = y 
```


---

# Questions?

<!-- _class: invert questions -->

---

# Behind the scenes

Data types in Haskell work in roughly the same way as they do in C behind the scenes.

There is a number stored in the value for each constructor.

This number is hidden to you, but Haskell keeps track of it.

That means there's a small bit of overhead. And if there is only one constructor, that overhead is optional...

---

# Newtype

Sometimes we know that the data type will have exactly one constructor that takes exactly one piece of data.

This happens when we use types as *wrappers*. That is, we have a basic type, like a String, but it represents something specific, like an Id.

We could do this:
`data Id = Id String`
(you're allowed to have constructors with the same name as the datatype)

Here, we're saying an `Id` is just a string. But the user has to explicitly use the constructor so that it's clear they know the string is being used as an `Id`.

---

# Newtype (2)

However, this might involve overhead. What if the Haskell implementation is storing data about the variant, but there's only one variant so it's pointless?

There's a keyword you can use to suggest to Haskell that it should not do this:
```haskell
newtype Id = Id String
```

Here, `newtype` is just like `data`, but it can *only* be used when there is a single constructor that has exactly one field.

Not zero fields: `newtype Blark = Blorp -- wrong!`
Not two fields: `newtype Bloink = Blip Int Float`
Exactly one field `newtype Blomp = Bloop Int`

---

# Newtype (3)

Newtype is kind of a janky feature. You don't have to use it, it was added as a tiny optimization.

In programming language design, we often don't like features like this. We generally like features that are *orthogonal*. 

*Orthogonal* features are those that behave the same way consistently, or that interact well with a large number of other features.

---

# Orthogonality

For example, functions in Haskell are a very orthogonal feature. Their basic mechanics are responsible for almost all the features of the language.

Case expressions are also orthogonal: case can be used to match any expression.

`newtype` is not very orthogonal. It only works with a very specific kind of datatype. It's the kind of feature that makes programming language designers say "ugh".

So does that mean we shouldn't have it?

---

# Unfortunately we need it

`newtype` actually does fill a tiny but important role.

You might think "just check to see if there is only one constructor with one field and then omit the extra information that `data` stores."

The problem is that the extra information here also allows the constructor to store erroneous values without crashing the program.

This is getting esoteric, but there is a certain value, called "bottom" (written $\bot$) that basically represents an "unreachable" value.

---

# Unfortunately we need it (2)

With lazy evaluation, a constructor that holds a field with bottom in it will not be considered an error by itself. In rare circumstances this can be desierable. For example, deliberately leaving and undefined value when I know I won't need it but I want to crash if I'm wrong.

I don't expect this all to make sense right now. The main point I'm making is that programming languages aren't things that emerge naturally, or that are necessarily perfect.

Instead, they are things that are engineered. And engineering involves tradeoffs. Every language is going to have something janky in it, and Haskell is no exception.

---

# One more option

There's one more, slightly less janky way of defining new datatypes.

The `type` keyword is useful. It lets you define a *type alias*.

A type alias is literally just another name for a type. Like this:

```haskell
type Blip = String
```

Now, everytime I write `Blip`, it will be interpreted as `String`.

```haskell
f :: Blip -> Blip
f x =
    | x == "Hi" = "Hi back at you!"
    | otherwise = "You said: " ++ x
```
(reminder that `++` is string concatination)

---

# Knowledge check 3

1. Use newtype. Just use it to define some kind of type that takes a String as a field.
2. Now use `type` to define an alias of that type.
3. Now redefine the function `f` on the previous slide to take your alias. Be sure to use the constructor when defining the function. You can't create an instance of a custom type defined with `data` or `newtype` without calling the constructor, and you can't get data out without some kind of pattern matching (either directly or in a function).

---

# Knowledge check 3 answers

1. `newtype Blop = Blop String`
2. `type Bloop = Blop`
3. 
```haskell
f :: Bloop -> Bloop
f (Blop "Hi") = "Hi back at you!"
f (Blop whatever) = "You said" ++ whatever
```

---

# Questions?

<!-- _class: invert questions -->

---



---

# Lists

Now that we understand datatypes, let's build a linked list!

Ignore the fact that we already have them. We're trying to understand them better.




---

start with datatypes and lists, end with recursion

---

# Lists and conses

One of the most important datatypes in functional programming is the linked list.

Why? Because all the useful operations on them don't have to delete anything.

For example, if we don't want the head, we can return a new list without the head. The old value will be cleaned up later by the garbage collector.

---


the function call operator