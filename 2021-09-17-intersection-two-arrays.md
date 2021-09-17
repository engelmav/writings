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


