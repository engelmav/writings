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



Unfortunately we seem to be deriving the rules from results. That happens sometimes. It's frustrating but it can be worked through... with enough patience.

