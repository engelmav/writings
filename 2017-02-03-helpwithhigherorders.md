---
title: Higher Order Help
author: Vincent Engelmann
---

When you see the ``applyTwice`` function

```haskell
applyTwice :: (a -> a) -> a -> a
applyTwice f x = f (f x)
```

it gets clearer if you substitute the variables with actual values and "run it" in your mind. Let's run it as such:

``ghci> applyTwice ((+) 1) 2``

(notice I am intentionally setting the plus function to the front so that it looks like a regular old function)

So let's substitute the values we ran it with into the function definition. It will look like:

```haskell
applyTwice ((+) 1) 2 = (+) 1 ((+) 1 2)
                       (+) 1 3
                       4
```

And with list concatenation:

``ghci> applyTwice (++ " secondParam") "firstParam"``

will evaluate as

```
... = (++ "secondParam") ((++ "secondParam") "firstParam")
    = (++ "secondParam") "firstParam secondParam"
    = "firstParam secondParam secondParam"
```

This is me attempting to make clearer the "HAHA" example. The difficulty here is that ``++`` is ordinarily infix, so the parameter orders look weird. When I write ``++ "secondParam"``, I choose the string ``secondParam`` because that string is the *second parameter*, for example in the expression ``"join" ++ " me"``, where ``me`` is the second parameter.

