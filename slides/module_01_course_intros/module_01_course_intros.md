---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 1: Introduction to Programming Languages

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# What is this class?

This class is about programming languages

Those things we use to write computer programs. 

Specifically, it's about:
- Programming paradigms: the different ways that programming languages see the world.
- Language design: how to actually design and implement a programming language from scratch 
- Functional programming: the programming paradigm that is very popular now in the industry, but which many of you have not ever used before. 
- Haskell: the specific functional programming language we will be learning

---

# About me

I am a teaching professor here

I used to work at Microsoft as a data scientist and software engineer.

Before that, most of my research focused on automated requirements engineering.

However, I have published a workshop paper in which I created a programming language ([Copper](https://scholar.google.com/citations?view_op=view_citation&hl=en&user=uifpy9gAAAAJ&citation_for_view=uifpy9gAAAAJ:LkGwnXOMwfcC)). Programming languages have always been of interest to me. 

---

# What's interesting about programming languages?

At this point, you've learned a few languages:
- You've had computer org, so some assembly experience. Maybe C and assembly, so some C.
- You may have had systems programming: more C.
- Some of you had CS 220, which teaches Typescript.
- Those of you who started here: Python was the language of CS 122.
- You may have also learned Java for algorithms and data structures. 
- Transfer students may have had C++ at clark.

So that's like 5, right off the bat.

---

# Those languages

Now that you've had the experience of learning a bunch of languages, you're probably starting to realize that learning a new language is kind of...I don't want to say "automatic", but...

Like, you know what questions to ask:
- How do I make a variable?
- How do I make a class/struct?
- How do I make a function?
- How do i do a loop?

---

# Those languages (2)

The reason for this isn't that all of computing can be reduced to those concepts, and that all languages are basically the same.

The reason is actually that all the languages you've learned have been imperative paradigm languages, with varying degrees of object-orientation.

They've all been the same *kind* of language. There are radically different ways that programming can work. And as someone who uses them reasonably often, IMO they're just as good.

But before we go too deep, let's talk about how class is going to work...

---

# Questions?

<!-- _class: invert questions -->

---

# The structure of this class

This is a learning mastery class. That means there will be several things that you need to demonstrate (mastery elements) to pass.

Each one is worth a chunk of your final grade.

However, even though they will all be tested in in-class quizzes, I will give you two retakes for each one.

Therefore, there will be 3 opportunities to demonstrate each mastery element. 

I will take the maximum score of all of these to determine what you make on the mastery element.

---

# The quizzes

Each mastery element will have one quiz focused on that element.

There will be six quizzes in total, and each will be roughly 10 minutes long.

If you get a 100% on all the quizzes, congratulations, you don't need to show up at the midterms or the final.

---

# The "exams"

There will be a "midterm" and an "endterm" exam.

Each one will just be three mastery problems stapled together. 

It's not really an exam, it's just a retest opportunity. There will be 3 new problems.

There's a final too. It's going to have problems for all six mastery elements stapled together.

---

# The retakes

If you get a 100% on mastery element 1, a 0% on mastery element 2, and a 50% on mastery element 3, you will want to take the midterm.

When you do, you can skip problem 1. It's just going to be a re-take opportunity for ME 1, which you have already gotten a 100% on so you can't improve. 

However, it will be an opportunity to improve on ME 2 and 3.

If you still need an opportunity, you can take the final. It will have all six.

---

# Skipping tests

If you're happy with your grade, you can skip the final. Therefore, the exams and final are optional. They are just retake options that are made universally available.

You can also skip the quizzes and take the midterm, or even skip everything except the final.

The latter is a bad idea: if you miss everything, I can't give you an incomplete grade for the class, because you haven't attempted 70% of it. You would fail unless you could obtain a withdrawal (or medical withdrawal)


---

# Mastery areas: before midterm

1. Demonstrate an understanding of programming paradigms, and the special role that lambda calculus plays as the foundation of functional programming. Demonstrate a basic understanding of lambda calculus, including anonymous functions, partial application, and curried definitions.
2. Demonstrate a basic level of competence with Haskell. E.g., define types, use functions, write some non-trivial functions.
3. Demonstrate an understanding of functional operators such as map, filter, and fold.

---

# Mastery areas: before second term

4. Demonstrate an understanding of type-classes, especially Functor and Applicative.
5. Demonstrate an understanding of logic programming.
6. Demonstrate an understanding of the dreaded monad, especially the bind operation (either implementing, using, or both).

---

# Let's look at the rest of the syllabus

---

# Questions?

<!-- _class: invert questions -->

---

# Languages

What is a language?

According to Wikipedia: “a structured system of communication”

I like this definition. If a communication system doesn’t have structure, it’s hard to call it a language 

Structure here means grammar: a set of rules for building complex thoughts out of groups simple ones. 

Crows communicate by caw-ing, but I haven’t heard evidence that crow communication supports nested sub-clauses or grammar.

---

# Languages (2)

Nested sub-clauses? That's what makes human languages rich. 

Humans can nest clauses inside of clauses: “I thought that he liked that he wasn’t having to work nights.”

“I thought that ( he liked that ( he wasn’t having to work nights. ) )”

---

# Human languages

In academia, we put languages into 2 categories: human and formal.

A human language is spoken by humans. There are two sub-categories of human language:
1. Natural: the language evolved over time
2. Constructed: someone invented the language intentionally 

Most human languages are natural languages.

However, there are some human languages that are not natural. They are called “constructed languages” or “conlangs” for short. [Can anyone name some?]

---

# Formal languages

In contrast to human languages, formal languages are created by specifying precise grammar rules with no room for ambiguity. 

With formal languages, we can say that a phrase is “valid” or “invalid”.

There’s no “mostly understandable but confusing” like with human languages. 

Instead, we follow rules precisely. 

---

# Example of rules

[This](https://cs.wmich.edu/~gupta/teaching/cs4850/sumII06/The%20syntax%20of%20C%20in%20Backus-Naur%20form.htm) is an exhaustive syntax of the C programming language.

It contains all the information needed to determine whether a given string of letters is a valid C program or not. 

This is called “syntax”

Note: it does not tell you how to execute that language or what it means (that’s called “semantics”), only whether it’s a C program. 

We’ll talk more about how to read this grammar later in the course, but notice how much smaller it is than a book on English grammar. 

---

# Human Languages

- Natural
    - English
    - Japanese
    - Albanian
    - Cherokee
    - ... like 7,000 more ...
- Constructed
    - Esperanto
    - Toki pona
    - Elvish
    - ...

---

# Formal Languages

- Mathematical expressions
- Predicate/first-order/second-order logic
- C
- [Malbolge](https://en.wikipedia.org/wiki/Malbolge)
- Python
- Ruby
- Scratch (yes, visual languages can be formal)
- UML
- The event scripting language for RPG Maker games.

---

