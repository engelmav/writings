Title: Recursion
Date: 2013-12-18 16:00
Tags: haskell
Category: Learning

For me, it is more difficult to understand *how* a program executes recursively, not precisely that it *is* recursive.

I found a way to write out the recursion execution in such a way that it was clearer to me.

For the following code (from [LYAH](http://learnyouahaskell.com/recursion)):

```haskell
maximum' :: (Ord a) => [a] -> a
maximum' []  = error "Cannot find max of empty list."
maximum' [x] = x
maximum' (x:xs) = max x (maximum' xs)
```

We can trace this as follows:

Let's say we pass it the list [ 2, 5, 1 ]. Each step here shows which pattern matched.

| Step | What code looks like |
|-----|----------------|
| 1 | ``maximum' (2:[5,1]) = max 2 (maximum' [5,1]``|
| 2 | ``maximum' (5:[1])  = max 5 (maximum' [1])`` |
| 3 | ``maximum' ([1])     = 1`` |

We've sort of traced this program *down* into its recursion. Now we need to pull the results *back up* to the top. So, we're going from step 3 to 2, and 2 to 1. Let's begin. How does step 3 work with step 2?
The answer (for me) has been to replace ``maximum'`` with the result of the next step.

For example, if we look at step 2:

>> ``maximum' (5:[1]) = max 5 (maximum' [1])``

The result of the next step (step 3) is 1, so we replace the ``maximum [1]`` with ``1``, and it becomes:

>> ``maximum' (5:[1]) = max 5 1``

We continue doing the same thing between steps 1 and 2. Step 2 has been reduced to ``max 5``, which is obviously ``5``. So step 1's right side now becomes:

>> ``max 2 5``

which evaluates to ``5``.
