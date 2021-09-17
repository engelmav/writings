Find the intersection of two arrays.

This seems like it would be easy. Let's start out with a small test case*.

```
nums1 = [1, 2, 3]
nums2 = [2, 3]
```

The numbers common to both arrays are 2 and 3. So, for each number in `nums1`, I am going to check in `nums2` to see
if it's present. If it is present, I'm going to add it to another list, let's call that list `common_to_both`.

```
common_to_both = []
for number in nums1:
    if number in nums2:
        common_to_both.append(number)
```

First try.

```
    def intersect(self, nums1: List[int], nums2: List[int]) -> List[int]:
        intersect = []
        for item in nums1:
            if item in nums2:
                intersect.append(item)
        return intersect
```

The above try causes this test case to fail:

[1,2,2,1]
[2]

Got [2,2], Expecting [2]


```
    def intersect(self, nums1: List[int], nums2: List[int]) -> List[int]:
        intersect = {}
        for item in nums1:
            if item in nums2:
                intersect[item] = None
        return list(intersect.keys())
```

The above try causes this test case to fail:

[1,2,2,1]
[2,2]

Got [2], Expecting [2,2]

That's weird. What's the difference between these two test cases?

```
[1,2,2,1]
[2]

Got [2,2], Expecting [2]
```

```
[1,2,2,1]
[2,2]

Got [2], Expecting [2,2]
```

Let's verbalize what we're seeing.

1st test case: When the first list has two 2s, and the second list has one 2, the test expects to see only one 2 (seems to reflect the number of 2s in the second list).

2nd test case: When the first list has two 2s and the second list has two 2s, the test expects to see two 2s.

So far my conclusion is, the result must contain the same number of repeats as are present in the second array.

How can we modify our code to follow this pattern?

I tried flipping the inner and outer arrays and that satisfies these two test cases. 

```
def find_intersection(nums1, nums2):
    common_to_both = []
    for number in nums2:
        if number in nums1:
            common_to_both.append(number)
    return common_to_both
```

Let's run this in leetcode.


22/55 passed.

```
[4,9,5]
[9,4,9,8,4]
```
Got: [9,4,9,4], Expecting [4,9]

What's the difference between what we got and what was expected?

* what we got is duplicated compared to the expected. But we know that duplicate answers are actually OK. There is a certain condition under which it's OK to have dupes, and anther condition under which it's not OK. What are those conditions?
* Why did the program return [9,4,9,4]? It returned the first 9 because it 


```
Each element in the result must appear as many times as it shows in both arrays
```

This seems to contradict the teszt failure above. Or does it mean, appear the minimum between the two arrays? That would make more sense. Let's see [how this guy did it](https://medium.com/@punitkmr/intersection-of-two-arrays-ii-ffb26f04dfd1) (this is the Python version):

```
def find_intersection(nums1, nums2):
    i = 0
    j = 0

    nums1.sort()
    nums2.sort()

    result = []

    while i < len(nums1) and j < len(nums2):
        if nums1[i] == nums2[j]:
            result.append(nums1[i]);
            # go to next index
            i += 1
            j += 1
        elif nums1[i] < nums2[j]:
            # if the number at this index of the first array is smaller than
            # the number at the index of the second array, only increment
            # the index of the first array (i).
            # In other words, walk forward the first array but not the second.
            # Why?
            i += 1
        else:
            # if the number at this index of the first array is NOT smaller than
            # the number at the index of the second array, only increment
            # the index of the SECOND array.
            # Why?
            j += 1
    return result
```

Put the currently failing test case through this and see if it reveals anything.

```
[4,9,5]
[9,4,9,8,4]
```
Got: [9,4,9,4], Expecting [4,9]

It works! But how or why??

Let's trace. If you're doing find_intersection([4, 5, 9], [4, 4, 8, 9, 9])


1. nums1[0] = 4 with nums2[0] = 4, increment i and j, result.append(nums1[i]) => [4]
2. nums1[1] = 5 with nums2[1] = 4, increment j
3. nums1[1] = 5 with nums2[2] = 8, increment i
4. nums1[2] = 9 with nums2[2] = 8, increment j
5. nums1[2] = 9 with nums2[3] = 9, increment i and j, result.append(nums1[i]) => [4, 9]
6. nums1[3] = ... the while condition is not satisfied, iteration stops.


Unfortunately we seem to be deriving the rules from results. That happens sometimes. It's frustrating but it can be worked through... with enough patience.

