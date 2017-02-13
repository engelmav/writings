Title: If-Then
Date: 2014-02-14
Tags: logic, boolean, sentential 
Category: Learning

I had a hard time with the ``if .. then`` truth table. An "if this, then that" scenario *seems* "intuitive," but look at it:

```
      p | q | (p -> q)
     ________________   
1)    T | T |    T
2)    T | F |    F
3)    F | T |    T
4)    F | F |    T
```

Lines ``3`` and ``4`` just made no sense to me.

If ``p`` is FALSE and ``q`` is TRUE, how does that make the statement true?

If ``p`` is FALSE and ``q`` is FALSE, how does that make the statement true?

Luckily, as is usually the case, there are a wealth of explanations about this on the StackExchange site (thank you, [Mr Atwood](http://blog.codinghorror.com/about-me)).

Two concepts have proved most helpful:

1. P and Q are linked to each other. You can't affect one without affecting another (it's a sort of interdependence)
2. You need *context* for your ``P``s and ``Q``s to really see why this is the case 

How do you provide context? Well, you can make those lone ``P``s and ``Q``s actual mathematical functions instead of just symbols. It'll help give more "meaning" to the concept of ``True`` and ``False``.

So let's make a couple of function examples (it appears we actually need *two* in order to complete our context). We'll start by giving an example of the fourth line of the truth table.

Example 1:

```
p =  x > 0

q = 2x > 0
```

Example 2:

```
p =     x > 0

q = x + 1 > 0
```

Remember, the results of these functions is a simple ``True`` or ``False`` (we are "assigning" this boolean to the ``p`` or ``q`` at the beginning of these example functions).

Now in human-speak, example 1 becomes:

>> ``2x`` is positive when ``x`` is positive.

Example 2 becomes:

>> If ``x`` is positive, then so is ``x+1``.

Let's run example 1 into reality. Meaning, we'll choose a number and plug it into ``x``. We'll make ``x = -1``.

In doing so, our first example becomes:

```
p = (-1) > 0
```
``p`` here will return ``False``!

and
```
q = 2(-1) > 0
```
``q`` here will *also* return ``False``!

So let's put *that* situation into plain English.

>> If you make ``p`` false, you're making q false, given what ``p`` and ``q`` stand for.

Here's the important part. What you are checking here is *the truth of that very statement!*

Rephrased as a question:

>> Is it true that if you make ``p`` false, that ``q`` is false... *given what ``p`` and ``q`` stand for*?

...whereupon you should find it in your mind to say "Yes!" (e.g., ``True``)

It is *true* that if you make ``p`` false, you will make ``q`` false.   

This is a slight re-write of Jyrki's response [here](http://math.stackexchange.com/a/48181/96367). 

Anyway, that was line 4 of the truth table. Now for line 3.

For line 3 we go to example 2. Let's put example 2 through a reality check. We'll use ``x = -1/2``.

```
p = (-1/2) > 0
```
``p`` returns ``False``!

and
```
q = (-1/2) + 1 > 0
```
``q`` returns ``True``!

Again we put this into plain English.

>> *Under the conditions given, if you make ``p`` false, you'll make ``q`` true.

And again - you are *verifying the truth of this above statement*, not the individual ``p``s and ``qs``.

I find the question-form helpful, so here it goes:

>> Under the conditions given, if you make ``p`` false, will this make ``q`` true?

To which you'd answer, "Yup, *if you use those functions*, that's what happens." (e.g., the result is ``True``.).

There are two plain English phrases I have used that should show *context*:

* "...given what ``p`` and ``q`` stand for``" and
* "Under the conditions given..."

Both of these mean, we have provided an environment for our ``if-then`` -- it cannot stand alone!
