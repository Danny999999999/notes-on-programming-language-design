---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 7: Parsing 

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# Last time

We got more comfortable with practical coding in Haskell using functional operators.

A lot of operators... (but we *love* that, right?)

---

# This time

We're going to learn to parse.

This is incredibly important, and once you understand it, you will be most of the way to making your own toy language (i.e., our lisp dialect)

We've seen the term before, but what does it mean? [What is parsing in computer science?]

---

# To parse

In computer science, "to parse" means "to convert a string into a value, tree, or other data structure" (my definition)

So `parseInt` in javascript converts a string to an int. 

But we don't want to just stop with simple values, we want to parse *programs*.

That means we need to parse a more complex datastructure. How can we represent a *program*?

---

# Programs are trees

The most common datastructure we use to represent a program is a tree.

Why? Consider some C code:

```c
#include <stdio.h>
#include <stdlib.h>
int blort(int a, int b) {
    return 3 * a + 4 * b;
}
int main(int argc, char** argv) {
    if (argc != 3) { 
        puts ("usage: blort <a> <b>");
        return 1;
    }
    printf("blort is: %d\n", blort(atoi(argv[1]), atoi(argv[2])));
    return 0;
}
```

---

# Is that a tree?

It doesn't look like it at first glance, but a parser is turning that code into a tree.

The parser is part of the compiler. The compiler runs in several phases:
- Pre-processing (expanding `#include` and friends)
- Lexing (i.e., *lexical analysis*)
- Parsing (what we're going to cover today)
- Semantic analysis (i.e., checking types and other rules and finding errors)
- Optimization (happens in several places)
- Codegen (i.e., actually generating the lower-level code, such as assembly or llvm or machine code. We don't do this in this class: take the course in Compilers!)

---

# Compilation units

In C, the basic unit of compilation is the...uh...*compilation unit*.

When we run the compiler, it interprets its input as a giant text file (after all the preprocessor stuff has been included) that describes a bunch of functions and variables that need to be compiled together.

The (normal) *output* of a C compiler is an object file. That is, a `.o` file.

[Whats in an `.o` file?]

---

# Object files

You can see for yourself with the [`objdump`](https://linux.die.net/man/1/objdump) comand on *unixey* platforms. Or check the [file format](https://en.wikipedia.org/wiki/Executable_and_Linkable_Format) (it's similar on Windows but ).

But basically, it's broken into "sections" of binary data:
- All the data known at compilation time goes in the `.data` section.
- All the code goes in the `.text` section. (why not call it `.code`? I don't know)
- All the data that has space reserved at compilation time but whose size we don't know goes in the `.bss` section. (another strange name)
- A string table `.strtab`, which holds a list of exported function names (like "main")
- A symbol table `.symtab` which maps the string table strings to virtual addresses (i.e., the virtual address of the function `main`)

---

# Where is the tree?

Buried between the input (C code) and the output (an `.o` file) is the parsing step.

We actually represent the compilation unit as some kind of tree (or forest).

Why? Because even though a C program is just a string of code, it is conceptually "structured". Specifically, structured like a tree.

Here's how our C program above could be represented as a forest...

---

# Our program from earlier (as a prefix ordered tree)

Leaving out types:

```
compilation-unit [
    <stuff from header files>;
    function blort [
        params: a, b
        body: [
            return [ 
                + [ * [3; a]; * [4; b] ]
            ]
        ]
    ];
    function main [ ... ]; etc.
]
```

---

# Why?

Every function is itself a tree.

In fact, individual expressions are trees. Notice how we represent `3 * a + 4 * b`:

```
+ [
    * [ 3, a ];
    * [ 4, b ]
]
```

The root is `+`, the two inner nodes are `*`, and the leaves are 3, `a`, 4, and `b`.

By using trees, we can quickly answer questions like "I want to add here, but what am I adding? Oh, it's 3 and `a`"

By turning expressions into trees, we can either execute them (i.e., interpret them), or turn them into machine code easier.

---

# Example

Suppose we want to execute `+ [ * [3; a]; * ][4; b] ]`

We can recursively execute its children and add them.

We execute `* [3, a]` and get a result.
We execute `* [4, b]` and get another result.
We add those results together.

What about generating code? It's more complicated, but it's similar. We emit code to compute `3 * a` and, e.g., push the result. We emit code to compute `4 * b` and push the result. Then we emit code to add the top two stack elements together.
(in practice we would probably not use the stack here, but this is just conceptual)

---

# Knowledge check 1

1. Generate a tree using our goofy notation for the expression `2 + 2`
2. Now generate a tree for `2 + 2 + 2`
3. Now generate a tree for `2 + 2 * 3` but use order of operations (so the `*` would need to happen first)

---

# KC 1 answers

1. `+ [2; 2]`
2. Several answers: `+ [2; 2; 2]`,  `+[ +[ 2; 2]; 2]`, and `+[2; +[2; 2]]. `+` is associative, so we don't need to consider `2 + 2 + 2` distinct from `(2 + 2) + 2` or `2 + (2 + 2)`. All are the same expression.
3. This one only has one answer: `+[2; *[2; 3]]`

---

# Questions?

<!-- _class: invert questions -->

---



---

# Required reading

[Classes and Types](https://en.wikibooks.org/wiki/Haskell/Classes_and_types) (Very important!)

[The Functor Class](https://en.wikibooks.org/wiki/Haskell/The_Functor_class) (Also very important!)