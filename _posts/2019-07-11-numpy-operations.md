---
title:  "Numpy operations - [:,np.newaxis], [x::y]"
# categories: tech
tags: Python
---

### Problem
I read the code of a function on github but had trouble understanding a few lines using numpy operations like \[np.newaxis,\:\] and \[x\:\:y\].

```python
array1 = np.arange(10)[np.newaxis,:]
array2 = array1[:, 0::2]
```

### \[np.newaxis,\:\]
Increase the dimension of the existing array by one more dimension.
- 1D array will become 2D array
- 2D array will become 3D array
```python
>>> array = np.arange(10)
>>> array.shape
(10,)
>>> array[:,np.newaxis].shape
(10,1)
>>> array[np.newaxis,:].shape
(1,10)
```

### \[x\:\:y\]
Extended Slice. x means 'starting index' and y means 'step'.

```python
>>> array = np.arange(10)
>>> array[1::2]
[1,3,5,7,9]
```
