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

# Tail recursion

Is recursion really good enough?

We know about for-loops. We also know they're pretty fast.

Function calls have overhead. 

So doesn't using recursion everywhere slow us down?

Well, most of the time there really is some overhead to it. But not *always*...

---

# Tail recursion (2)

There's a special time in which recursion doesn't really involve overhead.

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

So, what about `foldl` and `foldr`? Remember when we defined 

---

# Questions?

<!-- _class: invert questions -->

---

# Knowledge check 5

1. Define a function `reverse'`, which should take a list an reverse it. Include its type. Do not use tail recursion.
2. What is the big-O runtime of this function?
3. Define a tail-recursive version. Include its type.
4. What is the big-O runtime of this function? This is the one I warned you about earlier when I said tail recursion would have a huge impact on performance. 
5. 

---

# KC 5 answers

---