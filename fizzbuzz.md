Title: Fizz Buzz
Date: 2014-07-02 19:00
Category: Solutions
Tags: python 

Here it goes.

We run through the FizzBuzz requirements:

1. Create a list from 1 to 100
2. For each item in the list that is divisible by 3, print "fizz"
3. For each item in the list that is divisible by 5, print "buzz"
4. For each item in the list that is divisible by both 5 and three, print "FizzBuzz"

And potentially mock it out someway through (not always necessary, but for illustration):

![sketching](../images/fizzbuzz.png)

(There is a mistake in the image -- the 6th row, column B should be blank.)

I suspect that, were it necessary, one could inductively prove this...

We implement the pattern seen above in Python:

```python
def main():
    xs = range(1,101)
    map(fizz_buzz,xs) # or for...

def fizz_buzz(x):
    if x % 15 == 0: # Requires commentary...
        result = "%d\t%s" % (x, "FizzBuzz") # do not use tabs like me
    elif x % 3 == 0:
        result = "%d\t%s" % (x, "fizz")
    elif x % 5 == 0:
        result = "%d\t%s" % (x, "buzz")
    else:
        result = x
    print result

if __name__ == '__main__':
    main()
```

I saw this least common multiple approach at a [Haskell meetup downtown](http://www.meetup.com/Haskell_For_Cats/events/188432402/). Granted it's cool, and I think personally math does well to be in a programmer's toolkit, but it may elude some folks reading the code (or even the you two weeks in the future!) And so, I would probably really use `if x % 3 == 0 and x % 5 == 0:`. The syntax makes this code easier to grok at a glance.

At any rate, it's important to note that this condition must come at the top of the conditions list, since it will "short circuit" the rest of the `if-then-else` structure.

But how you implement this (or anything)  probably hinges on 3 main considerations:

1. readability
2. computational speed
3. testability

At least here I've ignored readability and gone for the secret fourth consideration -- cleverness -- most likely because this is an interview question.

If speed is essential, we can do some profiling. Using a standard library module called `timeit`, we can do a speed comparison between the "Least Common Multiple Approach" and the "Explicit Approach." I ran the script at 100000 repetitions with an input range from 1 to 1001:

The "Explicit Approach":
```
vengelmann@t430:~$ python timedfizz.py 
26.7621219158
```
The "LCM Approach":
```
vengelmann@t430:~$ python timedfizz.py 
24.4981269836
```
...for a gain of ~2.264 seconds.

This is also an example where optimization can lead to decreased readability (OK, I'm stretching it), and so we see optimization has its own 'cost'.

Now of course we can import `unittest`, wrap the fizzbuzz function as a method in a class, make that class inherit from `unittest.TestCase`, and write in some assertion methods, etc.
