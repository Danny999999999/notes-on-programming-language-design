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

# How to parse

Assume you accept my premise that code is easier to work with (meaning execute or compile) when it is structured as a tree.

How do we actually build one of these trees?

There are a ton of ways to do it, but there is a particularly elegant way that works perfectly in Haskell called *Recursive Descent*.

Let's learn how it works...

---

# Parsing arithmetic expressions

Let's say I want to parse an arithmetic expression like `2 + 3 * x - 1`

There are a couple of things that make it harder:
1. We need to handle in-order operations. It's not in a convenient form for turning into a tree.
2. We need to handle precedence. We want the expression to parse the same as this: `2 + (3 * x) - 1`.

First, let's look at what our input will actually look like...

---

# It will be tokenized

We already talked about tokenization, so let's assume that we have tokenized this expression. It will look something like this:

`[IntLit 2, Plus, IntLit 3, Times, Var "x", Minus, IntLit 1]`

Assuming we have a token datatype that looks like this:
```haskell
data Token = 
        IntLit Integer
    |   Plus
    |   Minus
    |   Times
    |   Var String
    |   etc.
```

---

# The goal

The goal (here) is to parse a single expression. So some string of operators and operands that will end up being evaluated.

In our example, the type is simple: it's either addition, subtraction, or multiplication:
```haskell
data Expr = 
        Add Expr Expr -- a sum (e.g., a + b)
    |   Sub Expr Expr -- a difference (e.g., a - b)
    |   Mul Expr Expr -- a product (e.g., a * b)
    |   Val Integer -- a value by itself (e.g., 7)
```

This datatype is actually a tree. `Val` is a constructor for leaves. Otherwise, the other constructors take two expressions. This will be the result of our parsing.

`Add Expr Expr` means "this expression is the sum of two smaller expressions.", etc.

---

# The goal (2)

We will start with some string like `2 + 3 * x - 1`, and we want to turn it into an `Expr`.

Why do we need to do that? Why not just turn it into a single integer like `7`?

Because of the `x`. We don't actually know what value this expression is going to have until we decide what `x` is.

Therefore, we have to store the whole expression, we can't just reduce it down to a single value (although if you could, feel free to try; it's a good optimization when it's available!) 

---

# The goal (3)

Therefore, we are writing this function:
```haskell
parseExpr :: [Token] -> Expr
```

That's it. We take a string and return an expression.

The technique we use will be useful for parsing most things, including programs, not just expressions, but let's start there.

---

# Questions before we get started?

<!-- _class: questions invert -->

---

# The starting point

Okay, to start with, what *is* an expression?

Well, it could be a sum, a difference, a product, a variable, or a value.

Let's consider this as a list ordered by *precedence*:

```
expr ::= expr + term
expr ::= expr - term
expr ::= term
expr ::= term * factor
term ::= factor
factor ::= Variable
factor ::= Value
```

---

# Backus-Naur form

That is a language that is used to define grammars called "Backus-Naur form" or (BNF).

You've probably seen it in your automata class, but here's a refresher:

BNF works like this: `production ::= rules`

This says that we can replace a list of `rules` with a production named `production`.

So if we have an expression, followed by a `+` and then a term, we can invoke this rule to combine them all into an expression:
`expr ::= expr + term`

---

# Backus-Naur warnings

We have to be careful when we write our grammar.

This is not the right grammar:
```
expr ::= expr + expr
expr ::= expr - expr
expr ::= expr * expr
expr ::= Variable
expr ::= Value
```

[What's wrong with it?]

---

# It's ambiguous

The first issue with that grammar is that it's ambiguous. 

We don't just care *whether* a string is an expression, but *how*.

For example, which tree is correct for `1 + 2 * 3`:
1. `1 + (2 * 3)` which results in `Add 1 (Mul 2 3)`
2. `(1 + 2) * 3` which results in `Mul (Add 1 2) 3`

We know that we want to follow normal order of operations, which means #1. But the grammar doesn't require that, it's possible to apply rules in either order...

---

# It's ambiguous (2)

When we see `1 + 2 * 3`,

We can apply the `expr ::= value` rule to get this: `Value 1 + Value 2 * 3`.
Then we apply `expr ::= expr + expr`  to get this: `(Add (Value 1) (Value 2)) * 3`
Then we apply the `expr :: value` rule again... `(Add (Value 1) (Value 2)) (Value 3)`
Finally, `Mul (Add (Value 1) (Value 2)) (Value 3)`

But, this is what we *want*:
Apply `expr ::= value` to `2` and `3`: `1 + Value 2 * Value 3`
Then `Value 2 + (Mul (Value 2) (Value 3))`
Then `Add 2 (Mul (Value 2) (Value 3))`

So we need to make sure there's only one way to parse it.

---

# Fixing it

We need to create intermediate categories like *term* and *factor*:
```
expr ::= expr + term
expr ::= term
term ::= term * factor
term ::= factor
factor ::= variable
factor :: value
```

Now we only have one way to parse: `1 + 2 * 3`. What rules can we apply?

Previously we went bottom-up, replacing words as quickly as possible.

Instead, let's start from `expr`...

---

# Top-down parsing

`1 + 2 * 3`

There are two ways to build an expression. Either from a `+` expression, or from a term.
We see a `+`, so let's try that: `expr 1 + term (2 * 3)`

We need to parse 1 as an expression and `2 * 3` as a term.

The only way to parse 1 as an expression, is to treat it like a term, which means to treat it as a factor, and then to make it a value.

To parse `2 * 3`, we invoke `term * factor`, and then parse `2` as a factor and then a value. `3` is already a factor, so we treat it as a value.

So we end up with `Add (Value 1) (Mul (Value 2) (Value 3))`

And importantly: we can't get anything else. Any other parse fails (end up with a non expression)

---

# Associativity is fixed, too

Notice how we have this in our language:
```
expr ::= expr + term
expr ::= expr - term 
...
```

These are left-recursive rules. What would change if we did this?

```
expr ::= term + expr
expr ::= term - expr
...
```

[?]

---

# The associativity would change

If we did that, we would end up parsing `1 + 2 + 3` as `1 + (2 + 3)` instead of `(1 + 2) + 3`. Is that a problem?

Not for `+`, but it is for `-`: `1 - 2 - 3` should be 4, not `1 - (2 - 3) == 1 - (-1) == 2`

So we can't do `expr ::= expr + expr` because we get ambiguous parses
We can't do `expr ::= term + expr` because the parentheses go around the recursive parse, and we want `+` (and `-`) to be left recursive.

Instead, we do `expr ::= expr + term` to get the right associativity, and to also ensure that multiplication happens before addition, even if it's on the right of a `+`.

---

# Knowledge check 2

1. Parse 1 + 2 * x * 3 - 9 - 2 by hand. What Haskell `Expr` do you end up with?
2. Extend the grammar by adding `^` to it (exponentiation). Make it be right associative.

If you're wondering how we extend our intuition about how to do this to Haskell, don't worry. That's coming up.

---

# KC 2 answers

1. I like to start by putting in parentheses:
   `((1 + ((2 * x) * 3)) - 9) - 2`
   Now start putting in constructors:
   `((Add 1 ((2 * 4) * 3)) - 9) - 2`
   `((Add 1 ((Mul 2 4) * 3)) - 9) - 2`
   `((Add 1 (Mul (Mul 2 4) 3)) - 9) - 2`
   `(Sub (Add 1 (Mul (Mul 2 4) 3)) 9) - 2`
   `Sub (Sub (Add 1 (Mul (Mul 2 4) 3)) 9) 2`
   Lastly, fill in the values and variables:
   `Sub (Sub (Add (Value 1) (Mul (Mul (Value 2) (Var "x")) (Value 3))) (Value 9)) (Value 2)`


This is what `parseExpr` would return. 

---

# KC 2 answers (2)

```
expr ::= expr + term
expr ::= expr - term
expr ::= term
term ::= term * factor
term ::= factor
factor ::= factor ^ exp
factor ::= exp
exp ::= Variable
exp ::= Value
```

---

# Questions?

<!-- _class: questions invert -->

---

# Putting this into Haskell

Now that we have refreshed on how to use grammars, let's turn this into Haskell.

Our goal is to write this function
```haskell
parseExpr :: [Token] -> Expr
```

However, that function is going to need to parse terms, and terms will need factors. So let's show the whole family:

```haskell
parseExpr :: [Token] -> Expr
parseTerm :: [Token] -> ([Token], Expr) -- the tuple has the left-over string
parseFactor :: [Token] -> ([Token], Expr)
parseVariable :: [Token] -> ([Token], Expr)
parseValue :: [Token] -> ([Token], Expr) 
```

---

# Variables and values

Let's start with the easiest ones, `parseVariable` and `parseValue`.

We just take a Value token and wrap it in a `Val` `Expr`:
```haskell
parseVariable (Sym name) : remainder = (remainder, Variable name)
parseVariable _ = error "expected a symbol"
```

This is why so many error messages say "expected 'blah'". It's because once we've decided we need a variable, if there isn't one there, that's an informative thing to say (it tells the user what state the parser was in).

[do `parseValue`]

---

# Parsing factors

Parsing factor is a little more complex, but not really.

Remember the rules look like this:
```
factor ::= variable
factor ::= value 
```

So the function just has two definitions, one for a symbol and one for a literal:
```haskell
parseFactor :: [Token] -> ([Token], Expr) 
parseFactor (Sym s) : rem = parseVariable $ (Sym s) : rem
parseFactor (IntLit i) : rem = parseValue $ (IntLit i) : rem
parseFactor _ = error "parsing factor: expected symbol or literal."
```

---

# What to notice so far

So far, notice that for each rule, we have a function definition.

`factor ::= Variable` becomes `parseFactor (Sym s) : rem = ...`
`factor ::= Value` becomes `parseFactor (IntLit i) : rem = ...`

We're effectively treating our grammar rules as functions.

It's worked so far, let's keep going...

---

# Parsing terms

The first tricky one is when parsing a term. We have two possibilities:
1. The term is just a factor (i.e., there is no `*`)
2. The term is an actual multiplication of two operands.

Let's apply the grammar and see what happens:
```haskell
parseTerm tokens =
    let (remaining, first) = parseTerm tokens
    in  if null remaining then ([], first) -- no operator
        else let (remaining', second) = parseTerm' remaining
             in Mul first second
    where
        parseTerm' (Times : remaining) = parseFactor remaining 
        parseTerm' _ = error "expected '*'"  
```

Let's understand this first, and then try to see a problem...

---

# Parsing terms

We are attempting to apply the parsing rule `term ::= term * factor` directly

That means, we first parse a term recursively. Then we check for a `*`. If we find one, we parse the remaining factor.

The result is either a `Term` or a `Mul (Term ...) (Factor ...)`
Either way, we return it and the rest of the tokens.

But there's a problem here. This code won't work.

[What's wrong?]

---

# Left-recursive grammar

The problem is that the grammar is left-recursive. 

So our first recursive call is unguarded. 

It will therefore run forever.

This part...
```haskell
parseTerm tokens =
    let (remaining, first) = parseTerm tokens
    in ... 
```

...is the issue. Specifically that parseTerm call.

---

# What's the problem?

Suppose it were right-recursive: `term ::= factor * term`

This would no longer have the associativity we want, and it would pose a problem if we added division to the language, but let's just pretend that was the rule. 

Then, our function would look like this:
```haskell
parseTerm tokens =
    let (remaining, first) = parseFactor tokens
        h : remaining' = remaining
    in if h == '*' then 
            let (remaining'', second) = parseTerm tokens
            in (remaining'', Mul first second)
        else (remaining', first)
```

This version does not have the problem.

---

# How can we fix it?

We have a few options:
1. Change the grammar to eliminate the recursion
2. Modify the behavior (the semantics) so that the math works out with right recursion.

Let's consider both options.

---

# Changing the grammar

If we remove the recursive term by expanding it in the grammar (meaning, replace it with equivalent things that aren't recursive), we can get around this.

So instead of this:
```
term ::= term * factor
term ::= factor
```

We replace the "some kind of term and a star" with this regular expression style term:

```
term ::= (factor '*')* factor
```

---

# What's that?

That first star '*' is just the star symbol for multiplication.

That second star is a Kleene star. Just like a regular expression, it means "zero or more of the thing before me".

This modification of Backus-Naur form is called [Extended Backus-Naur form](https://en.wikipedia.org/wiki/Extended_Backus%E2%80%93Naur_form).

Why is it better? Well, let's revisit what it looks like to parse a term...

---

# Parsing a term

---

# Required reading

[Classes and Types](https://en.wikibooks.org/wiki/Haskell/Classes_and_types) (Very important!)

[The Functor Class](https://en.wikibooks.org/wiki/Haskell/The_Functor_class) (Also very important!)