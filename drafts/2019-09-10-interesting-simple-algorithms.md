# Boyer-Moore majority voting
> given an array, find out the elements which appear more than ceil(n/k)
1. only `k-1` elements could be majority
2. when you meet distinct k items, eliminate them all. then finally the majority will remain, if they exist. Maybe you run through the array again to check if they are really the majority.
