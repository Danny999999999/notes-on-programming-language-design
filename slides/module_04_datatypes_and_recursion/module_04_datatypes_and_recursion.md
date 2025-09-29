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

----

# Differences between C and Haskell enums/datas

You don’t define the actual number that each variant takes. (that’s hidden by the compiler)

You separate with `|` instead of `,`

We use initial camel case instead of all-caps snake case. 

In Haskell you *may not* mix variants from other types.

Suppose `f :: GameState -> Bool`
This is allowed: `f InGame`
This is forbidden: `f Red`

Haskell is very strictly-typed here. The variants are not just constants. They are real values that can't be mixed with incompatible types.

---



---

start with datatypes and lists, end with recursion

---

# Lists and conses

One of the most important datatypes in functional programming is the linked list.

Why? Because all the useful operations on them don't have to delete anything.

For example, if we don't want the head, we can return a new list without the head. The old value will be cleaned up later by the garbage collector.

---

# Pattern Matching

---

# Guards
