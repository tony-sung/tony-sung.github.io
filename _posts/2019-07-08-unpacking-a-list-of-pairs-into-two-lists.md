---
title:  "Unpacking a list of pairs into two lists"
# categories: tech
tags: Python
---

### Problem
Let's assume there's a list of pairs.
```
>>> source_list = [('1','a'),('2','b'),('3','c'),('4','d')]
```

If you want to seprate the list into two lists like the following, what should you do?
```
>>> list1
['1','2','3','4']
>>> list2
['a','b','c','d']
```

### Solution
One of the easiest ways for this is using **zip(\*list)**.
```
>>> list1, list2 = zip(*source_list)
>>> list1
['1','2','3','4']
>>> list2
['a','b','c','d']
```
The single star '\*' refers to unpacking the collection into positional arguments.

### Reference
https://stackoverflow.com/questions/7558908/unpacking-a-list-tuple-of-pairs-into-two-lists-tuples