---
marp: true
theme: slides
paginate: true
---

# Programming Language Design

## Module 6: More functional operators

<br>
<br>

Slides © Grant Williams, [CC BY-SA 4.0](https://creativecommons.org/licenses/by-sa/4.0/).  

<br>

This is an open educational resource.
Feel free to submit fixes, improvements, and new material [here](https://github.com/grantwill74/notes-on-programming-language-design).

---

# Last time

We learned about lexing. [what is it?]

We learned about functional operators. [what are they?]

---

# This time

Way more functional operators! (I know you think they're awesome)

Tail recursion (an optimizable way to write recursive functions)

---

# Tail recursion

Is recursion really good enough?

We know about for-loops. We also know they're pretty fast.

Function calls have overhead. 

So doesn't using recursion everywhere slow us down?

Well, most of the time there really is some overhead to it. But not *always*...

---

# Tail recursion (2)

There's a special time in which recursion doesn't necessarily require much more overhead than a loop does.

It's when the last thing we do (the "return") is the recursive call. 

We refer to such a function call as a "tail call".

For example, this is *not* a tail call:
`sum (x : xs) = x + sum xs`

Because, after we call `sum xs` recursively, we then add x to the result.

However, what if we wrote it like this...

---

# Tail recursion (3)

```haskell
sum acc [] = acc
sum acc (x : xs) = sum (acc + x) xs 
```

Here, we're still computing the sum. `sum 0 [1,2,3]` still returns `6`. However, the recursive call of sum is the last (and only) thing the function does. Adding `x` happens before.

Here, we call `acc` an *accumulator* (hence its name). We add each `x` value to the accumulator, and when we're out of elements, we return it.

---

# Tail recursion (4)

Why is this faster?

Because the function's stack frame can be re-used.

Consider what would happen in the classic C calling convention:
`int result = sum(0, p, n);`

In old-school 32-bit `ccall`, we first push `n`, then `p`, then `0`.

Inside the function, we do the same thing for the recursive call.

And again...

So every time there's a recursive call, we push the function's data.

When it's done, we free the data by moving the stack pointer back up (adding to it).

---

# Tail recursion (5)

There are two issues:
1. Pushing/moving the stack pointer takes a little time
2. We don't actually free that memory until the very end. The stack stays allocated.

Are these things bad? Well, number 1 is a small slowdown, but number 2 is a bigger problem. In C, there is a limited amount of stack memory.

In Haskell, there's quite a lot more stack space by default, but you can still have stack overruns if you're not careful.

Given the importance of recursion in functional programming, we like to use recursion heavily, and we don't want to run out of stack space.

---

# Tail recursion (6)

So, if the *last thing* a function does is to return a recursive call, we can just re-use the same stack space:

```c
int sum(int acc, int* vals, int n) {
    if (n <= 0) return acc;

    return sum(acc + *vals, vals + 1, n - 1);
}
```

Here, for the recursive call to `sum`, we can overwrite `acc` with `acc + *vals`, we can overwrite `vals` with `vals + 1` (which makes it point to the next value), and we can overwrite `n` with `n - 1`. 

---

# Tail recursion (7)

This function is *not* tail recursive, because the addition happens after the recursive call:

```c
int sum(int* vals, int n) {
    if (n <= 0) return 0;
    return sum(vals + 1, n - 1) 
}
```

So we have to keep pushing the pointer and `n` on the stack until finally, we reach `n == 0`, and we can return from each call.

---

# Tail recursion (8)

In haskell, not tail recursive:
```haskell
sumNoTail [] = 0
sumNoTail (x : xs) = x + sumNoTail xs
```

Tail recursive:
```haskell
sumTail acc [] = acc
sumTail acc (x : xs) = sumTail (acc + x) xs
```

Is it worth it? Well, if we expect the list to be long, *maybe* it's worth optimizing. Sometimes, though, it's vastly faster to use tail recursion. We'll see an example soon.

But, even if you use tail recursion in Haskell, you can still crash from stack overflow. That's because of lazy evaluation.

---

# Lazy eval and tail recursion

What happens in Haskell when we do this? 
`y = sumTail 0 [1,2,3]`

Normally: nothing, because of lazy evaluation. However, suppose we actually use `y` later: `main = print y`. Now we need to compute `y` so we can print it. 

We apply the definition of `sumTail`:
```haskell
sumTail 0 [1,2,3] == 
sumTail (0 + 1) [2,3] == 
sumTail (0 + 1 + 2) [3] ==
sumTail (0 + 1 + 2 + 3) [] ==
    (0 + 1 + 2 + 3) -- <- actual answer
```

We don't actually combine `0 + 1 + 2 + 3` into one value until printing.

---

# Lazy eval and tail recursion (2)

Why do we keep collecting values? Why does the accumulator end up being `0 + 1 + 2 + 3` instead of `6`? Because of lazy evaluation.

When we compute something like `x + 1`, Haskell doesn't immediately reduce that into a single value. Instead, it creates a *thunk* which is like an eager lambda function that, when run, will eventually return a value.

So here, the thunk would be a function that calls the thunk for `x` and then adds `1`.

This means that we still end up using extra space until the function returns, even when we're using tail recursion.

This is the real cost of lazy evaluation: it makes memory usage unpredictable. This is one reason why Haskell wouldn't be my first choice as a system or gamedev language.

---

# Can we fix that?

We actually can just tell Haskell not to lazy-evaluate stuff using the `$!` operator.

This operator is almost identical to the `$` operator, but it forces its right argument to be eagerly evaluated by one stage, so you basically do one addition.

For example: 
```haskell
sumTail acc [] = acc
sumTail acc (x : xs) = (sumTail $! (acc + x)) xs
```

So now, because of the `$!`, we actually reduce the `acc + x` into a single value each time we recursive call, preventing the thunk from growing.

---

# Foldl?

So, what about `foldl` and `foldr`? Remember when we defined those?

I don't, because we only defined `fold`, so here it is:
```haskell
foldl'' :: (b -> a -> b) -> b -> [a] -> b
foldl'' _ init [] = init
foldl'' f acc (x : xs) = foldl'' f (acc `f` x) xs

foldr'' :: (a -> b -> b) -> b -> [a] -> b
foldr'' _ init [] = init
foldr'' f acc (x : xs) = f x (foldr'' f acc xs)
```

Notice that `foldl` is tail-recursive and `foldr` is not.

So does that mean we should use `foldl` when we can? Because it's tail recursive? Well, not really... I mean, first, we should notice that it's lazy, not eager!

---

# Foldl (2)

There's actually another version of `foldl` that forces strict evaluation.

You have to `import Data.List` to use it...

It's called...[drumroll]

`foldl'`

It's the same as `foldl`, but it's strict. So `sum = foldl' (+) 0` won't stack overflow.

There's also `foldr'`, but it seems less useful to me, because it's still not tail recursive, so the stack keeps growing.

---

# Should I use it?

So if `foldl'` is tail-recursive and eager, that must be the fastest one, right?

It makes sense, but we should test it. 

I'm going to make a simple microbenchmark, but first, **big warning**: microbenchmarks are highly specific. They show you how a language performs on a very specific task on a specific platform at a specific point in time and phase of the moon. 

I'm going to show you what happens when we use `foldl'` instead of `foldl` specifically to sum a giant list of `Int` (not `Integer`) *and* the list has already been fully constructed (not lazily) *and* it's running on Windows *and* probably a bunch of other stuff I haven't controlled for.

---

# A microbenchmark

```haskell
main :: IO ()
main = do
    let count = 100000000
    let xs = [1..count] :: [Int]
    deepseq xs (return ()) -- force construct list first: I'll explain this
    
    start <- getCurrentTime 
    print $ foldl (+) 0 xs
    end <- getCurrentTime
    putStrLn $ "foldl time: " ++ show (diffUTCTime end start)

    start <- getCurrentTime
    print $ foldl' (+) 0 xs
    end <- getCurrentTime 
    putStrLn $ "foldl' time: " ++ show (diffUTCTime end start) 
```

Note: if you want to do this, import `Data.List`, `Data.Time`, and  `Control.DeepSeq`

---

# Results

On my specific computer I get this:
```
5000000050000000
foldl time: 15.7061889s
5000000050000000
foldl' time: 0.5516658s
```

So it actually made a huge difference.

But note: *big* list. I've seen another person online run this test in different circumstances online and they said that it actually slowed it down.

Also, in other news, `foldr` is actually faster for me than `foldl`. Probably because it doesn't end up needing to build thunks for each addition.

Okay, now let me answer your other question...

---

# What is `deepseq`

`deepseq` is related to `$!`. 

In fact, there is a function called `seq` that is closely related to `$!`. 

`seq a b` means "slightly simplify `a`, then return `b`

This technically means it has a side effect. It is one of the few haskell "functions" with side effects. If they are observable, you probably don't want to use it.

We can define `$!` in terms of `seq`: `f $! x = f (seq x x)`

What is "slightly simplify"? In the case of a list, it will remove the outer thunk and expose the *cons*: `seq (1 : 2 : []) (1 : 2 : [])` will create a single list node with a value of `1`, whose next pointer is a thunk that will construct `2 : []`


---

# What is `deepseq`? (2)

That's all `seq` does. Calling it again won't help. It just consumes the outer thunk.

For arithmetic operations, it actually does simplify the whole thing, but those are a special case. 

Since seq isn't that helpful, if we want to thoroughly construct the entire list, we use `deepseq`. 

This consumes the thunk and replaces it with a fully initialized list, so that we can compare `foldl` and `foldl'` directly without also including the time it takes to allocate list nodes.

`deepseq x (return ())` means "first compute `x`, and then do nothing". `return` in Haskell does not mean what it means in C. It's a constructor for monads. In this case, an IO object that does nothing and returns `()`. We'll talk about this later.

---

# $!!

In the same way that `$!` does `seq` before calling a function, `$!!` does `deepseq`.

We don't need to do these things often, but it can occassionally be important, especially when doing performance optimization.

---

# Questions?

<!-- _class: invert questions -->

---

# Knowledge check 1

1. Define a function `reverse'`, which should take a list an reverse it. Include its type. Do not use tail recursion.
2. What is the big-O runtime of this function?
3. Define a tail-recursive version. Include its type.
4. What is the big-O runtime of this function? This is the one I warned you about earlier when I said tail recursion would have a huge impact on performance. 
5. Now use one of the folds to define `reverse`.
6. Suppose we had a giant list we wanted to reverse. How could we force evaluation to reverse it before we needed to use it later. 

---

# KC 1 answers (1)

1.
```haskell
reverse' :: [a] -> [a]
reverse' [] = []
reverse' (x : xs) = reverse' xs ++ [x]
```

2. It's quadratic! Because `++` is linear and we're doing it `n` times, where `n` is the length of the list. It's much slower than you'd think.
3. 
```haskell
reverse'' :: [a] -> [a] -> [a]
reverse'' acc [] = acc
reverse'' acc (x : xs) = reverse'' (x : acc) xs
```

---

# KC 1 answers (2)

4. This one is actually linear. We're prepending each value in the list to the front of the accumulator, which is fast (constant time). We do this for each value in the list.

5. 
```haskell
reverse''' :: [a] -> [a]
reverse''' = foldl (flip (:)) [] 
```

More explanation about `flip` on the next slide.

---

# `flip`

`flip` is a function that takes a function and flips its arguments. It's defined like this: `flip f = \y x -> f x y`. So instead of `:` taking a value and a list and prepending the left argument to the right, `flip (:)` returns a function that takes a list and then a value to prepend.

We run this on each value of the list. It's like using a stack to reverse a list. We push each value from left to right onto the front of the return list.

[can you do it without flip given the definition above?]

---

# KC 1 answers (3)

6. `let reversed = deepseq (reverse l) (reverse l) in ...`

Here, `deepseq` makes it so that the list is fully allocated and not just a bunch of thunks that perform the `cons` operation. 

It's really not required to do this. Remember that premature optimization is the root of all evil.

---

# Questions?

<!-- _class: invert questions -->

---

# More operators: zip

I know what you're thinking.

"We love functional programming, but there aren't enough higher order functions. Please teach us another functional operator!"

Okay, let's learn about `zip`. This is a surprisingly useful operator that doesn't show up very often in imperative languages (although Python has it).

---

# `zip`

Sometimes we have two lists that we want to "pair up" somehow.

For example, maybe there are a lists of names, and their employee ids and we want to print them out together.

First, let's see how we would do this in C...

---

# Pairing up lists in C

The most common way I see imperative programmers solve this problem is by using the same index in both lists.

```c
char* names[] = { "Alice", "Bob", "Camille", "Dan", ... };
char* eids[] = { "1234", "5678", "9123", "4567", ... }

void print_names_and_eids(char** names, char** ids, size_t n) {
    for (size_t i = 0; i < n; i++) {
        printf("%s: %s\n", names[i], ids[i]);
    }
}
```

Here, `i` is an index (a `size_t`, which is usually an `unsigned long`).

We iterate through both lists with the same index, so we "pair up" names and eids which are in the same order.

---

# Pairing up lists in Haskell

There is no reason we can't do this in Haskell, too:

```haskell
printNamesAndEids :: [String] -> [String] -> IO ()
printNamesAndEids _ [] = return ()
printNamesAndEids [] _ = return ()
printNamesAndEids (name : names) (eid : eids) = do
    putStrLn $ name ++ ": " ++ eid 
    printNamesAndEids names eids
```

(Note: I don't expect you to understand what `return ()` does yet, or what an `IO ()` really is. I'm just showing you that Haskell can do the same thing as C.)

However, as you might have started noticing, even though recursion is important to functional programming, we usually end up finding elegant ways of solving problems that handle the recursion for us.

---

# Using `zip` to pair up lists

That is what we will do here. We will use `zip`.

`zip` is a function that takes a pair of lists, and returns a list of pairs.

That is, `zip ["Alice", "Bob", "Camille"] ["123", "456", "789"] ==`
`[("Alice", "123"), ("Bob", "456"), ("Camille", "789")]`

Let's do a bit of clean up.

---

# Using `zip` to pair up lists

```haskell
import Control.Monad -- we will learn more about monads later. Just a taste!
names = ["Alice", "Bob", "Camille"]
ids = ["123", "456", "789"]

namesAndEids :: [String]
namesAndEids = map (\(n, e) -> n ++ ": " ++ e) $ zip names eids

printNamesAndEids' :: IO ()
printNamesAndEids' = mapM_ putStrLn $ namesAndEids names ids  
```

Don't worry to much about the `mapM_` function. It's kind of like Haskell's version of "foreach" in that it applies a function to a list. 

The important thing is that `map ... zip` above...

---

# Using `zip` (2)

Here it is again

```haskell
namesAndEids :: [String]
namesAndEids = map (\(n, e) -> n ++ ": " ++ e) $ zip names eids
```

`zip names eidss` is returning a list of pairs.

Then, `map (\(n, e) -> n ++ ": " ++ e) ...` is applying that lambda function to each pair. It is taking the name and eid and pasting them together with `": "`.

The result is just a list of strings like "Alice: 123", "Bob: 456", "Camille: 789"

---

# `zip` and `map` together

Using map with zip is so common, there's a special combination operator: `zipWith`:

```haskell
namesAndEids' = zipWith (\n e -> n ++ ": " ++ e) names ids
```

It also automatically uncurries the function, so we can write `\n e` instead of `\(n, e)`.

[what is currying and uncurrying again?]

---

# Knowledge check 2

1. Use zip to create a list of `(n, n^2)` for each natural number `n`. This should be an infinite list.
2. Print the first 10 elements of that infinite list.
3. Now construct a new infinite list that is the sum of each element of the first list. So `(0 + 0^2), (1 + 1^2), (2 + 2^2), (3 + 3^2), ...`
4. Print the first 10 elements of this infinite list.
5. Now, use `zipWith` to construct the same list as 3 without needing `uncurry`.

---

# KC 2 answers

1. 
```haskell
nAndN2 :: [(Integer, Integer)]
nAndN2 = zip [0..] $ map (^2) [0..]
```

2. `print $ take 10 nAndN2`

3. `sumNAndN2 = map (uncurry (+)) nAndN2`

4. `print $ take 10 sumNAndN2`

5. `sumNAndN2' = zipWith (+) [0..] $ map (^2) [0..]`

---

# Questions?

<!-- _class: invert questions -->

---

# Making `zip`

Can we define `zip` ourselves?

First, [what should its type be?]

---

# Making `zip`

```haskell
zip :: [a] -> [b] -> [(a, b)]
```

"Give me a list of `a`s and a list of `b`s and I will give you a list of `(a, b)` pairs.

Now, [code it up]. First, let's use recursion...

---

# Making `zip`

```haskell
zip :: [a] -> [b] -> [(a, b)]
zip _ [] = []   
zip [] _ = [] 
zip (x : xs) (y : ys) = (x, y) : (zip xs ys)
```

If either list is empty, we're done.

Otherwise, we pair up the head of both remaining lists, and append it to the end of the result.

This is the most straightforward way to do it. It's hard to use one of the `fold`s to make `zip`, because it takes 2 list arguments instead of 1. 

It's technically possible, but it involves some goofy coding, like having the accumulator be a tuple of the 2nd list and an empty list of pairs and such. It's easier recursively.

---

# What about the opposite?

If there is a zip, which takes two lists and makes a list of pairs, is there also an unzip?

[What do you think?]

---

# Of course there is!

And if there weren't, we could make it ourselves.

`unzip [(1, 'a'), (2, 'b'), (3, 'c')]` should be
`([1,2,3], ['a', 'b', 'c'])`

Right?

What would its type be?

---

# `unzip`'s type

```haskell
unzip :: [(a, b)] -> ([a], [b])
```

"Give me a list of pairs, I will give you a pair of lists."

How do we code it?

Start recursively...

---

# `unzip`

```haskell
unzip' :: [(a, b)] -> ([a], [b])
unzip' [] = ([], [])
unzip' ((x, y) : rest) =
    let (xs, ys) = unzip' rest
    in  (x : xs, y : ys)
```

The wrinkle here is that we need to bind the `xs` and `ys` from the recursive call so we can push new values onto them.

You might have preferred the recursive solutions to these built-in operators so far, but I think you'll like the one using fold for this one.

First, which fold do we use? `foldl` or `foldr`?

---

# `unzip` (2)

`foldl` would end up reversing the order (trace through it to see why!). We use `r`.

```haskell
unzip'' :: [(a, b)] -> ([a], [b])
unzip'' = foldr (\(x, y) (accx, accy) -> (x : accx, y : accy)) ([], [])
```

Here, we start with an empty pair of lists. The binary function appends each of the pair in the original list to the appropriate value in the accumulator.

When we're done, the accumulator holds our result.

It ends up being in the right order because we push from right to left. `foldl` would push from left to right and flip the order!

---

# Is `unzip` the inverse of `zip`?

We say that `g` is the inverse of `f` if `g . f` is equivalent to `id`. 

`id` is the identity function. `id x = x`. It's just the lambda that returns its argument without doing anything.

If applying `g` after `f` is the same as doing nothing at all, `g` is an inverse of `f`.

It seems like `unzip` "undoes" `zip`. So are they inverses?

---

# Not quite



---

# Questions?

<!-- _class: questions invert -->

---

