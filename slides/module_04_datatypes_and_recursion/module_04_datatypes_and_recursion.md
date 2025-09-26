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

For example, a webserver has certain `modes` for connections, where it might be parsing, authenticating, 



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
