+++
title = "Loop analysis"
date =  2018-08-26T23:10:41+01:00
weight = 5
+++

### Loop analysis

It's important to remember that the time complexity, and in particular the big O notation is an _aproximation_ and it is supposed to represents only the _growth_ in relation of a variable input.
It's not the sum of the time taken by each algorithm step, it's a measurement of the computation time growth for an arbitrary, variable input.
Anything constant can be ignored

#### O(1)

Represent the time complexity of an algorithms which doesn't perform any loop, recursion, or call to a non time-costant function

```php
function(int $arg): int {
  // any very complex computation
  // still O(1)
  return $arg * 2;
}
```

A loop or recursion that is executed a constant number of times is still considered O(1), no matter how big is the constant

The following loop has still a complexity of O(1)

```php
$c = 1000000; // $c is constant
for($i = 0; $i <= $c; $i++) {
  // O(1) expression
  echo $i;
}
```

#### O(n)

It's a linear _growth_, the time taken by the algorithm grows linearly as the input grows.

```php
// $n is the variable input
for($i = 0; $i < $n; $i++) {
  echo $i;
}
```

In this case, the loop iterator is usually incremented by a constant amount.

```php
$c = 100; // constant increment, O(n) still depends on n only
for($i = 0; $i < $n; $i += $c) {
  echo $i;
}
```

Even if we use an arbitrary amount for incrementing the iterator, the loop is still dependant only on the variable input _$n_
