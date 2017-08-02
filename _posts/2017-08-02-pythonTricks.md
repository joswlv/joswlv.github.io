---
layout: post
title: Python Tricks
date: 2017-08-02 10:04
categories: python
---

## 9 Neat Python Tricks Beginners Should Know

### Trick #1

#### String Reversing

```python
>>> a = "comdementor"
>>> print "Reverse is",a[::-1]
Reverse is rotnemedoc
```

### Trick #2

#### Transposing a Matrix

```python
>>> mat = [[1,2,3], [4,5,6]]
>>> zip(*mat)
[(1,4), (2,5), (3,6)]
```

### Trick #3

#### Store all three values of the list in 3 new variables

```python
>>> a = [1,2,3]
>>> x, y, z = a
>>> x
1
>>> y
2
>>> z
3
```

### Trick #4

#### Create a single from all the elements in list above

```python
>>> a = ["a", "b", "c", "d"]
>>> print " ".join(a)
a b c d
```

### Trick #5

```python
>>> list1 = ['a','b','c','d']
>>> list2 = ['p','q','r','s']
>>> for x, y in zip(list1,list2):
... 	print x, y
...
a p
b q
c r
d s
```

### Trick #6

#### Swap two numbers with on line of code

```python
>>> a=7
>>> b=5
>>> b, a = a, b
>>> a
5
>>> b
7
```

### Trick #7

```python
>>> print "code"*4+' '+"good"*5
codecodecodecode goodgoodgoodgoodgood
```

### Trick #8

#### Convert it to a single list without using any loops.

```python
>>> list1 = [ [1 , 2] , [ 3, 4] ]
>>> answer = sum( list1 , [ ] )
>>> print answer
1,2,3,4
```
### Trick #9

#### Taking a string input

```python
>>> wantToInt = "1 2 3 4"
>>> map(int, wantToInt.split())
[1, 2, 3, 4]
```

### 출처

- [https://www.codementor.io/sumit12dec/python-tricks-for-beginners-du107t193](https://www.codementor.io/sumit12dec/python-tricks-for-beginners-du107t193)