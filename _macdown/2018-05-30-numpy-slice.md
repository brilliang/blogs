
# Slicing in Numpy
It is a reading note of one piece of `numpy` [doc](https://docs.scipy.org/doc/numpy-1.13.0/reference/arrays.indexing.html).

Pure python can only index or slice on the 1st dimension of a list. When the list has many dimensions, we have to use cascaded square brackets. 

```python
>>> a = [[1,2], [3,4]]
>>> a[0][1]
2
```

But cascaded square brackets will invoke `__getitem__` several times, we need a method to do the slicing more efficiently In `numpy`.

Then `numpy` uses a `selection tuple` to indicate how to slice, and provides a syntactic sugar for `x[(exp1, exp2, ..., expN)]`.

So that we can write as `x[exp1, exp2, ..., expN]`. Every element in the selection tuple associates with one dimension of the narray.

By now, we can try to learn how many methods can be used to slice one dimension of the narray.


### Basic Slicing: use the [slice](https://docs.python.org/3/library/functions.html#slice)
slice is a build-in class in python, which has read-only data attributes `start`, `stop` and `step` and no other explicit functionality. Some of the 3 attributes have default values when omitted and can be negative.

```python
>>> x = np.array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9])
>>> x[1:7:2]
array([1, 3, 5])
```


#### Ellipsis
Ellipsis can be used when you only want to specify slice for a few dimension but not all, which means you will load all data of that dimension.

```python
>>> x
array([[[1],
        [2],
        [3]],

       [[4],
        [5],
        [6]]])
>>> x[...,0]
array([[1, 2, 3],
       [4, 5, 6]])
>>> x[0, ...]
array([[1],
       [2],
       [3]])

```

#### newaxis

newaxis expands the dimensions of the resulting selection by one unit-length dimension. 


### Advanced Indexing
Please notice its name is *indexing* rather than slicing. Because it will indicate every element in the dimension if it's selected or not.

#### Integer array indexing
You can use a integer array to specify what index will be selected from each dimension:

```python
>>> x = np.array([[1, 2], [3, 4], [5, 6]])
>>> x[[0, 1, 2], [0, 1, 0]]
array([1, 4, 5])
```

at the mean time you can organise the structure of index to format your output:

```python
>>> x = array([[ 0,  1,  2],
...            [ 3,  4,  5],
...            [ 6,  7,  8],
...            [ 9, 10, 11]])
>>> rows = np.array([[0, 0],
...                  [3, 3]], dtype=np.intp)
>>> columns = np.array([[0, 2],
...                     [0, 2]], dtype=np.intp)
>>> x[rows, columns]
array([[ 0,  2],
       [ 9, 11]])
```

#### Combining advanced and basic indexing

``` python

>>> x = array([[ 0,  1,  2],
...            [ 3,  4,  5]])
>>> x[1:2, [1, 2]]
array([[4, 5]])

```

#### Boolean array indexing
A common use case for this is filtering for desired element values:

``` python
>>> x = np.array([[1., 2.], [np.nan, 3.], [np.nan, np.nan]])
>>> x[~np.isnan(x)]
array([ 1.,  2.,  3.])
```

or, only apply to one dimension:

```python
>>> x = np.array([[0, 1], [1, 1], [2, 2]])
>>> rowsum = x.sum(-1)
>>> x[rowsum <= 2, :]
array([[0, 1],
       [1, 1]])
```
