---
layout: post
title: Let's Write Quicksort in J
---

This is [quicksort](https://en.wikipedia.org/wiki/Quicksort) in J:

```j
  qs =: (($:@(<#[), (=#[), $:@(>#[)) {.) ^: (1<#)
```

Quicksort is a sorting algorithm. You give it a list of items, it sorts them. Quickly.

If you want to know the detailed version of how it works, consult an algorithms text. Here is the basic version, which I’m lifting from [Wikipedia](https://en.wikipedia.org/wiki/Quicksort#Algorithm):

1. Pick an element, called a pivot, from the array.
2. Reorder the array so that all elements with values less than the pivot come before the pivot, while all elements with values greater than the pivot come after it (equal values can go either way). After this partitioning, the pivot is in its final position. This is called the partition operation.
3. Recursively apply the above steps to the sub-array of elements with smaller values and separately to the sub-array of elements with greater values.

## Create an Unsorted Input Array

For testing purposes, we’ll create an unsorted array of 10 elements, each one a random integer from 0 to 99. To do this, we’ll use `?` ([roll](http://code.jsoftware.com/wiki/Vocabulary/query)) and pass it a list of 10 `100`s.

	  unsorted =: ? 10 # 100
	44 35 49 67 95 56 50 35 70 83

## Pick a Pivot

Take `unsorted` and use `{.` ([head](http://code.jsoftware.com/wiki/Vocabulary/curlylfdot)) to take the first element. That’ll be our pivot:

	  pivot =: {. unsorted
	44

## Find All the Values Greater Than Pivot

Use `>` ([larger than](http://code.jsoftware.com/wiki/Vocabulary/gt#dyadic)) to compare each element of `unsorted` with the pivot. Those that are larger return true (`1`); those that are smaller return false (`0`).

	  unsorted > pivot
	0 0 1 1 1 1 1 0 1 1

Use `#` ([copy](http://code.jsoftware.com/wiki/Vocabulary/number#dyadic)) to pull a copy of the corresponding elements from `unsorted` where `>` returned true.

      NB. unsorted list       -> 44 35 49 67 95 56 50 35 70 83
      NB. larger than pivot   ->  0  0  1  1  1  1  1  0  1  1
      NB. return these values ->       49 67 95 56 50    70 83

      (unsorted > pivot) # unsorted

    49 67 95 56 50 70 83

## Bring in the Forks

To make this more concise, we’ll take advantage of a concept in J called a `fork`. A fork is a sequence of three verbs in a row (call them `f`, `g`, and `h`) that operate on an input (call it `y`):

	  (f g h) y

In J, this is evaluated as:

	  (f y) g (h y)

This is equivalent to operations on functions in mathematics:

	  (f + h)(x) = f(x) + h(x)
	  (f - h)(x) = f(x) - h(x)
	  (f * h)(x) = f(x) * h(x)

In forks, we just generalize to any operation.

As some J verbs (`dyadic` verbs) can take 2 arguments, there is a corresponding version of forks for dyadic verbs:

	  x (f g h) y

This is evaluated as:

	  (x f y) g (x h y)

Which is equivalent to operations on functions like:

	  (f + h)(x, y) = f(x,y) + h(x,y)
	  (f - h)(x, y) = f(x,y) - h(x,y)
	  (f * h)(x, y) = f(x,y) * h(x,y)

## Greater Than Pivot, Redux

So, using forks, this:

	  (unsorted > pivot) # unsorted

Can be rewritten as:

	  unsorted (>#[) pivot

Which will be evaluated as:

	  (unsorted > pivot) # (unsorted [ pivot)

We’ve introduced a new verb here, `[` ([left](http://code.jsoftware.com/wiki/Vocabulary/squarelf#dyadic)), which simply returns the value of the argument on the left, regardless of the argument on the right. So this simplifies to:

	  (unsorted > pivot) # unsorted

Which is what we started with.

So call the new verb `gtp` (greater than pivot):

	  ]gtp =: unsorted (>#[) pivot
	
	49 67 95 56 50 70 83

## Find All Values Less Than Pivot

Following our logic from `gtp`, we can define `ltp` (less than pivot), this time using `<` ([less than](http://code.jsoftware.com/wiki/Vocabulary/lt#dyadic)):

	  ]ltp =: unsorted (<#[) pivot
	
	35 35

## Find All Values Equal to Pivot

This time using `=` ([equal](http://code.jsoftware.com/wiki/Vocabulary/eq#dyadic)), we define `etp` (equal to pivot):

	  ]etp =: unsorted (=#[) pivot
	
	44

## Combine and Simplify

We can organize the results of each of these phrases using `,` ([append](http://code.jsoftware.com/wiki/Vocabulary/comma#dyadic)) to create a new list from the original lists:

	  ltp , etp , gtp
	
	35 35 44 49 67 95 56 50 70 83

This is equivalent to:

	  (unsorted (<#[) pivot) , (unsorted (=#[) pivot) , (unsorted (>#[) pivot)

There’s a lot of duplication here. We can simplify this quite a bit.

We have 5 verbs in a row:

	  ltp , etp , gtp

Remember forks? We can redefine the right three verbs as a fork:

	  (unsorted (=#[) pivot) , (unsorted (>#[) pivot)
	
	44 49 67 95 56 50 70 83
	
	  unsorted ( (=#[) , (>#[) ) pivot
	
	44 49 67 95 56 50 70 83

This leaves us with:

	  (unsorted (<#[) pivot) , (unsorted ( (=#[) , (>#[) ) pivot)

Which is another string of three verbs. So we can redefine this as a fork:

	  (unsorted (<#[) pivot) , (unsorted ( (=#[) , (>#[) ) pivot)
	
	35 35 44 49 67 95 56 50 70 83
	
	  unsorted ( (<#[) , (=#[) , (>#[) ) pivot
	
	35 35 44 49 67 95 56 50 70 83

Remember back to how `pivot` was defined:

	  pivot =: {. unsorted

So we can rewrite our fork as:

	  unsorted ( (<#[) , (=#[) , (>#[) ) {. unsorted
	
	35 35 44 49 67 95 56 50 70 83

Notice the structure of the sentence now, treating the entire verb train as a single verb

	  f =: ( (<#[) , (=#[) , (>#[) )
	  g =: {.
	  y =: unsorted
	
	  y f g y
	
	35 35 44 49 67 95 56 50 70 83

In J, the `y f g y` pattern is known as a `hook`, and it can be written more simply as:

	  (f g) y
	
	35 35 44 49 67 95 56 50 70 83

In combinatory logic, this is known as the [S combinator](https://en.wikipedia.org/wiki/SKI_combinator_calculus).

Rewriting the entire sentence using the `hook` structure, we get:

	  (( (<#[) , (=#[) , (>#[) ) {.) unsorted
	
	35 35 44 49 67 95 56 50 70 83

## Recurse

Now that we’ve simplified the structure, let’s continue with the sorting.

We’ve made a single pass, creating three sub-arrays — numbers less that the pivot, numbers equal to the pivot, and numbers greater than the pivot. However, the numbers in each of those groups are not themselves sorted.

Now we’ll proceed to step 3 of the algorithm:

> Recursively apply the above steps to the sub-array of elements with smaller values and separately to the sub-array of elements with greater values.

To do this, we’ll recursively invoke our quicksort code on the sub-arrays by combining the `ltp` and `gtp` phrases with the name of the sentence itself. We do that using `@` ([atop](http://code.jsoftware.com/wiki/Vocabulary/at)), which produces a single verb from two verbs.

If we represent that as `f@g`, then `f` will be applied to the results of `g` applied to a input value. In this case, we want to apply `qs` to the result of the `ltp` and `gtp` phrases. We don’t need to recurse on the `etp` phrase, because there’s nothing to sort there — everything in that sub-array is equal to the pivot.

	  qs =: (( qs@(<#[) , (=#[) , qs@(>#[) ) {.)
	  qs unsorted
	
	|stack error: qs
	|       qs unsorted

That’s not good. We recursed so many times that we exceeded the recursion limit. We need a way of identifying when to stop recursing.

J will stop recursing when the code returns a constant function.

In our case, we’ll use `^:` ([fixed power](http://code.jsoftware.com/wiki/Vocabulary/hatco)), which will apply the entire sentence to the input a number of times. It’s critical to know that any verb raised to the 0 power (`^:0`) will always return the input — a constant function. Raising a verb to the 0 power is just another way of saying that we're going to apply the verb to the input 0 times (not at all).

For instance, we can define a verb that will double the input:

	  double =: 2&*
	  double 2
	
	4

We can apply that verb to the input a number of times. For example, let’s apply it 5 times:

	  (double^:5) 2
	
	64

But if we apply it zero times, we get the input as the output:

	  (double^:0) 2
	
	2

Using this insight, we’ll use `^:` with our quicksort definition.

First, we’ll check whether the length of the input is greater than 1. If the comparison is true (`1`), then there are 2 or more items to sort, so apply the function 1 time and return the calculated output. If the comparison is false (`0`), then there is only 0 or 1 elements to sort (which is to say, there is no need to sort), so apply the function 0 times, and return the input as the output.

	  qs =: (( qs@(<#[) , (=#[) , qs@(>#[) ) {.) ^: (1<#)
	  qs unsorted
	
	35 35 44 49 50 56 67 70 83 95

This time, we didn’t exhaust the recursion limit, and we got the fully sorted list back. Success!

## Anonymous Recursion

Our recursion code explicitly refers to the name of the function. What happens if we change the name of the function?

	  qs_new =: (( qs@(<#[) , (=#[) , qs@(>#[) ) {.) ^: (1<#)
	  qs_new unsorted
	
	|value error: qs
	|       qs_new unsorted

To avoid this, we can use `$:` ([self-reference](http://code.jsoftware.com/wiki/Vocabulary/dollarco)) to refer to the recursive verb without specifying the name:

	  qs =: (( $:@(<#[) , (=#[) , $:@(>#[) ) {.) ^: (1<#)
	  qs unsorted
	
	35 35 44 49 50 56 67 70 83 95

We have now constructed the quicksort code from the beginning of the article. [QEF](http://mathworld.wolfram.com/QEF.html).

## Taciturn

If you’ve been reading closely, you’ll notice that we stopped putting the name of the unsorted array on the same line as the verb definition, and we didn’t replace it with a variable to hold the input:

	  qs =: (( $:@(<#[) , (=#[) , $:@(>#[) ) {.) ^: (1<#)

J allows verbs to be defined [tacitly](http://www.jsoftware.com/help/primer/tacit_definition.htm):

> In a tacit definition the arguments are not named and do not appear explicitly in the definition. The arguments are referred to implicitly by the syntactic requirements of the definition. You have already used several tacit definitions.

J makes extensive use of [tacit programming](https://en.wikipedia.org/wiki/Tacit_programming), allowing the structure of the code to emerge more clearly. That’s a story for another time.

_EOF_

