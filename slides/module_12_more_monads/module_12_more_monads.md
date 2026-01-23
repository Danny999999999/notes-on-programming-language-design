---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 12: More Monads

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# Last Time

We learned about the `IO` monad.

We learned how we could build a basic interpreter. The idea:
- Evaluating a lisp program means turning it into an `IO`
- We return the `IO` from `main`
- The runtime environment evaluates it.

---

# This time

Monad prep! Our quiz is soon, so let's do more prep for it.

We will also learn some more monads. `IO` is not the only one!

In fact, several data types we're already pretty familiar with are also monads, such as lists, `Maybe`, and `Either`. 

First, let's start with a classic problem that is often the first problem of a technical interview series...

---

# FizzBuzz

Write a program that, for every integer from 1 to 100, outputs the following:
- "Fizz" if the number is divisible by 3
- "Buzz" if the number is divisible by 5
- "FizzBuzz" if both conditions are true
- The number itself if it is divisible by neither 3 nor 5.

---

# FizzBuzz (2)

So the output is expected to be:
```
1
2
Fizz
4
Buzz
... 
14
FizzBuzz
16
...
```

Classically, we put newlines after each thing we print. 

Give it a try. Can you do it in Haskell? What about another language?

---

# [Think about it]

<!-- _class: questions invert -->

---

# Let's start with C

If this is a tough problem, [let's do it in C first]

---

# C FizzBuzz

```c
int main() {
    for (int i = 1; i <= 100; i++) {
        if (i % 15 == 0) puts("FizzBuzz");
        else if (i % 3 == 0) puts("Fizz");
        else if (i % 5 == 0) puts("Buzz");
        else printf("%d\n", i); 
    }

    return 0;
}
```

---

# C FizzBuzz remarks

We first check if `i % 15 == 0`, because that's the same as checking if `i` is divisible by 3 and 5 together.

This is kind of a trick, and it's okay if you didn't know it. You could do this just fine:
```c
if ((i % 3 == 0) && (i % 5 == 0)) puts("FizzBuzz");
```

Sometimes, people prefer to print "Fizz" and "Buzz" independently:
```c
if ((i % 3) == 0) printf("Fizz");
if ((i % 5) == 0) printf("Buzz");
if ((i % 3) != 0 && (i % 5) != 0) printf("%d\n", i);
```

This is fine, too, and is a nice design (it would scale well if the interviewer added more factors)

---

# Why is it hard in Haskell?

We need to do these things:

- Loop through all the numbers (from 1 to 100)
- Determine whether a number is a "Fizz", a "Buzz", or just itself
- Print that

The first step is actually the hardest in Haskell. We don't have *loops*. Or do we?

It turns out, with monads, we do. 

But first, let me give *one more* example, in one more language....

---

# Python FizzBuzz

```python
def fizzify(i):
    if i % 15 == 0: return "FizzBuzz"
    elif i % 3 == 0: return "Fizz"
    elif i % 5 == 0: return "Buzz"
    else: return i

for i in range(1, 101):
    print(fizzify(i))
```

Makes sense...okay, can we do that with Haskell? 

The `fizzify` function isn't too bad...

---

# Fizzify in Haskell

```haskell
fizzify :: Int -> String 
fizzify i
    | i `mod` 15 == 0 = "FizzBuzz"
    | i `mod` 3 == 0 = "Fizz"
    | i `mod` 5 == 0 = "Buzz"
    | otherwise = show i
```

Okay, now for the main part. How do we print the `fizzify` of every integer?

---

# Haskell FizzBuzz finished

```haskell
import Control.Monad

fizzify :: Int -> String 
fizzify i
    | i `mod` 15 == 0 = "FizzBuzz"
    | i `mod` 3 == 0 = "Fizz"
    | i `mod` 5 == 0 = "Buzz"
    | otherwise = show i

main :: IO ()
main =
    [1..100] `forM_` (\i ->
        putStrLn $ fizzify i
    )  
```

That's it. Notice how similar it looks to the Python.

---

# Explaining the main

Let's talk more about this part:
```haskell
[1..100] `forM_` (\i ->
    putStrLn $ fizzify i
)  
```

`forM_` is just `mapM_` with the parameters swapped. This is the same thing:

```haskell
(\i -> putStrLn $ fizzify i) `mapM_` [1..100]
```

We're mapping a function to every int in the list. If we just used regular `map`, we'd have: `[putStrLn $ fizzify 1, putStrLn $ fizzify 2, ..., putStrLn $ fizzify 100]`

---

# Explaining FizzBuzz in Haskell (2)

The difference between `map` and `mapM` is that `mapM` will then sequence all the elements of the list with `>>`. So we get this:

`putStrLn (fizzify 1) >> putStrLn (fizzify 2) >> putStrLn (fizzify 3) >> ...`

In fact, I could have just written this: `mapM_ (putStrLn . fizzify) [1..100]` 

If we used `mapM`, we'd have an `IO [...]` that would print everything and return the list of strings. We don't want the list of strings because `main :: IO ()` not `main :: IO [String]`, so we use `mapM_`.

However, let's look at that Haskell again and notice how *similar* it looks to Python, despite seeming really weird...

---

# FizzBuzz in Haskell (3)

```haskell
fizzify :: Int -> String 
fizzify i
    | i `mod` 15 == 0 = "FizzBuzz"
    | i `mod` 3 == 0 = "Fizz"
    | i `mod` 5 == 0 = "Buzz"
    | otherwise = show i

main :: IO ()
main =
    [1..100] `forM_` (\i ->
        putStrLn $ fizzify i
    )  
```

It's really not that different. The thing inside the for loop ends up getting sequenced with itself for every element of the list. So we end up with an `IO` that does the same thing as a "for-each".

---

# But `forM_` is a function

And this is the important thing. Haskell does not have loops as a built-in feature. They are not a part of the langauge. *`forM_` is a function!*

Its type, when applied to a list, is: `forM_ :: Monad m => [a] -> (a -> m b) -> m ()`

That is, it's a function that takes a list and a function that produces monads, and it will produce a single monad that is executed for its effects (i.e., doesn't return anything). 

But notice: this is a function that works *on any monad*. We've only really been using one kind of monad: `IO`. It turns out there are many. Let's learn about some!

But first...

---

# Questions?

<!-- _class: invert questions -->

---

# Maybe is a monad

Remember `Maybe`? A `Maybe a` *might* be `Nothing`, but it also could be `Just` an `a`.

`Maybe` is a functor. If we `fmap` a function with a `Maybe`, it applies the function to the value inside the `Just`, and just returns `Nothing` if the value is a `Nothing`.

`Maybe` is also an `Applicative`. a `Maybe f` applied to a `Maybe x` is `Just $ f x` if both maybes are `Just`, and `Nothing` if either is `Nothing`.

And `Maybe` is a `Monad`, too. 

What does that mean it can do?

---

# Monad reminders

Monads are things that can *bind*. Bind is written `>>=`.

Bind means "based on the result of the previous program, use this function to create a new program."

So the question we ask about unfamiliar monads: what do they do as programs? What happens when we bind them?

Think of bind as if, in C, you could redefine what a semicolon means. You could make it, for example, check the result of the previous operation to change based on what it returned. This is why monads are such a powerful pattern: they are programmable semicolons.

So what does `Maybe`'s bind do?

---

# The Maybe Monad

Let's set the stage. Consider a bunch of computations that can fail:

```haskell
fallableComputation1 :: Int -> Int -> Int -> Maybe Int
fallableComputation2 :: Int -> Maybe String
fallableComptuation3 :: String -> Maybe Double
```

All of these functions return `Maybe`s, because some of their inputs might make them fail. Suppose we want to use all of them to calculate something, but we want to stop if any of them fails (like an early return).

---

# The Maybe Monad (2)

We saw with applicative functors that we could sequence these, so that if any of them failed, we would get `Nothing`:

```haskell
fallableResult :: Maybe Double
fallableResult = 
    fallableComputation1 1 2 3 *>
    fallableComputation2 4 *>
    fallableComputation3 "hi"
```

And since `*>` is equivalent to `>>`, we can use `do` notation. Is this right?

```haskell
fallableResult = do
    fallableComputation1 1 2 3
    fallableComputation2 4
    fallableComputation3 "hi"
```

---

# Not quite

That isn't quite what we want! It will compile, but it probably isn't what we want.

[Why? What's wrong?]

---

# The problem

The problem is that we had to know the arguments of the functions in advance.

There's really no point, then. We could replace the `Maybe` with an `and`. 

But what we (probably) want is to have the second computation depend on the result of the first, and the result of the third computation depend on the result of the second.

Currently, we're not using that kind of dependency. In fact, `*>` *can't* do that. Only `>>=` (or the `<-` in do notation) can.

Let's see...

---

# Example of solution

```haskell
fallableResult :: Maybe Double 
fallableResult = do 
    x <- fallableComputation1 1 2 3
    y <- fallableComputation2 x
    fallableComputation3 y
```

or, equivalently:
```haskell
fallableResult =
    fallableComputation1 1 2 3 >>= 
    fallableComputation2 >>=
    fallableComputation3
```

Here, we're calling `fallableComputation1` with the initial arguments, but `fallableComputation2` and `fallableComputation3` depend on results from the previous computations.

---

# Why is that helpful?

Because it gives us early returns. 

In C, we often like this feature. It lets us return early so we can focus on the "happy path" (i.e., the code which we want to run rather than the boring error-handling code) without indenting it.

Consider this simple problem:
Parse an IPv4 address such as 123.123.123.123 into a big-endian integer (`Word32`).

In C, we might return an integer so that we could report failure, or maybe a boolean status code along with writing to a char pointer if successfull...

---

# Happy-path programming: one approach

```c
// split_on_dots("123.12.1.2") == {"123", "12", "1", "2"}
// writes the number of individual strings to number_of_elements
char** split_on_dots(char* str, int* number_of_elements);
bool parse_int(char* str, int* i); // returns false if it fails
_Bool parse_ipv4(char* ip_str, int* result) {
    int size;
    char** splitted = split_on_dots(ip_str, &size);
    if (size != 4) return false; // the first error
    // now, if *any* of the 4 integers fail to parse, we also bail
    for (int i = 0; i < 4; i++) {
        int* val; 
        if(!parse_int(splitted[i], &val)) return false;
        if(val < 0 || val > 255) return false;
        result |= val << ((3 - i) * 8)       
    }
    return true;
}
```

# Why happy path?

