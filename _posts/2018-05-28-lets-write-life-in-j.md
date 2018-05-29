---
layout: post
title: Let's Write Life in J
---

TL;DR — one J program for Conway’s Game of Life is:

```j
  ((3&=) +. (y&*.)@(4&=)) +/ (> , {;~ 1 0 _1) |. y 
```

## Create the Input Array

First, we need an input for the [Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) program to run on.

Create a 5x5 table with an embedded [glider](https://en.wikipedia.org/wiki/Glider_(Conway%27s_Life)):

```j
  ]R =: 5 5 $ 0 0 0 0 0 0 0 1 0 0 0 0 0 1 0 0 1 1 1 0 0 0 0 0 0
    
0 0 0 0 0
0 0 1 0 0
0 0 0 1 0
0 1 1 1 0
0 0 0 0 0
```

## Rotate the Array

For each cell, get a count of how many neighbor cells are alive (`1`) or dead (`0`).

Let’s take a simplified example. Take a 1x5 table

```j
  0 1 0 1 0
```

If we want to count how many neighbors of the each cell are alive, we can generate and then add two new tables: the original table rotated to the right and the original table rotated to the left. _Note: we’re assuming that this table wraps around._

```j
  rotated_right =: 0 0 1 0 1
  rotated_left  =: 1 0 1 0 0
  rotated_right + rotated_left
    
1 0 2 0 1
```

The result is the number of alive cells horizontally adjacent to each cell.

For our 2-dimensional table, we can do the same thing — except we’ll generate new tables by rotating left, right, up, down, and diagonally up and down. Then we’ll add up all the tables to get the sum of adjacent cells that are alive.

To do this, we’ll use the `|.` ([rotate](http://code.jsoftware.com/wiki/Vocabulary/bardot#dyadic)) verb. We’ll pass two numbers, the first rotating vertically and the second rotating horizontally. Positive numbers rotate up/left, negative numbers rotate down/right.

```j
  1 1 |. R
    
0 1 0 0 0
0 0 1 0 0
1 1 1 0 0
0 0 0 0 0
0 0 0 0 0
    
  _1 _1 |. R
    
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 0 0 1
0 0 1 1 1

  1 _1 |. R

0 0 0 1 0
0 0 0 0 1
0 0 1 1 1
0 0 0 0 0
0 0 0 0 0
```

Let’s do more than one rotation at a time. We’ll do this by passing an array of multiple rotation values.

Use the `,:` ([laminate](http://code.jsoftware.com/wiki/Vocabulary/commaco#dyadic)) verb to build a table from two lists. Then we can use `,` ([append](http://code.jsoftware.com/wiki/Vocabulary/comma#dyadic)) to add additional lists.

To generate the same three tables as above, we’d create a table with 3 lists.

```j
  1 1 , _1 _1 ,: 1 _1

 1  1
_1 _1
 1 _1

  (1 1 , _1 _1 ,: 1 _1) |. R

0 1 0 0 0
0 0 1 0 0
1 1 1 0 0
0 0 0 0 0
0 0 0 0 0

0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 0 0 1
0 0 1 1 1

0 0 0 1 0
0 0 0 0 1
0 0 1 1 1
0 0 0 0 0
0 0 0 0 0
```

We can apply all nine rotations this way, explicitly listing each rotation combination:

```j
  (0 1 , 1 0 , 1 _1 , 0 1 , 0 0 , 0 _1 , _1 1 , _1 0 ,: _1 _1) |. R

0 1 0 0 0
0 0 1 0 0
1 1 1 0 0
0 0 0 0 0
0 0 0 0 0

0 0 1 0 0
0 0 0 1 0
0 1 1 1 0
0 0 0 0 0
0 0 0 0 0

0 0 0 1 0
0 0 0 0 1
0 0 1 1 1
0 0 0 0 0
0 0 0 0 0

0 0 0 0 0
0 1 0 0 0
0 0 1 0 0
1 1 1 0 0
0 0 0 0 0

0 0 0 0 0
0 0 1 0 0
0 0 0 1 0
0 1 1 1 0
0 0 0 0 0

0 0 0 0 0
0 0 0 1 0
0 0 0 0 1
0 0 1 1 1
0 0 0 0 0

0 0 0 0 0
0 0 0 0 0
0 1 0 0 0
0 0 1 0 0
1 1 1 0 0

0 0 0 0 0
0 0 0 0 0
0 0 1 0 0
0 0 0 1 0
0 1 1 1 0

0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 0 0 1
0 0 1 1 1
```

It’d be cleaner if we could generate all the rotation combinations. We can do this using `{` ([catalogue](http://code.jsoftware.com/wiki/Vocabulary/curlylf)), which allows us to create a Cartesian product of two lists.

Catalogue takes a `boxed list`, a particular kind of datatype in J. We can create a boxed list by using `;` ([link](http://code.jsoftware.com/wiki/Vocabulary/semi#dyadic)) to create a list of boxed items.

```j
  1 0 _1 ; 1 0 _1

+--------+--------+
| 1 0 _1 | 1 0 _1 |
+--------+--------+
```
    
Then we pass this boxed list to `{` to get the Cartesian product:

```j
  { 1 0 _1 ; 1 0 _1
    
+-------+-------+-------+
|  1  1 |  1  0 |  1 _1 |
+-------+-------+-------+
|  0  1 |  0  0 |  0 _1 |
+-------+-------+-------+
| _1  1 | _1  0 | _1 _1 |
+-------+-------+-------+
```

From here, we convert the 2-dimensional boxed list into a 1-dimensional boxed list by using `,` ([ravel](http://code.jsoftware.com/wiki/Vocabulary/comma)):

```j
  , { 1 0 _1 ; 1 0 _1

+-----+-----+------+-----+-----+------+------+------+-------+
| 1 1 | 1 0 | 1 _1 | 0 1 | 0 0 | 0 _1 | _1 1 | _1 0 | _1 _1 |
+-----+-----+------+-----+-----+------+------+------+-------+
```

Finally, we unbox the entire thing, which converts it into the 2-dimensional array we wanted:

```j
  > , { 1 0 _1 ; 1 0 _1
    
 1  1
 1  0
 1 _1
 0  1
 0  0
 0 _1
_1  1
_1  0
_1 _1
```

We can reduce this further. As we’ve used it, `;` takes two arguments, the left `1 0 _1` and the right `1 0 _1`. In J, we represent this as `x ; y`. In our case, as both arguments are identical, we can use `~` ([reflex](http://code.jsoftware.com/wiki/Vocabulary/tilde)) in combination with `;` — this use of `~` takes a single argument and copies it on both sides of the verb, so that `;~ y` is the same as `y ; y`:

```j
  ;~ 1 0 _1
    
+--------+--------+
| 1 0 _1 | 1 0 _1 |
+--------+--------+
```

So we can say:

```j
  > , {;~ 1 0 _1
    
 1  1
 1  0
 1 _1
 0  1
 0  0
 0 _1
_1  1
_1  0
_1 _1
```

And now we can use this to build the rotated tables:

```j
  (> , {;~ 1 0 _1) |. R
    
0 1 0 0 0
0 0 1 0 0
1 1 1 0 0
0 0 0 0 0
0 0 0 0 0

0 0 1 0 0
0 0 0 1 0
0 1 1 1 0
0 0 0 0 0
0 0 0 0 0

0 0 0 1 0
0 0 0 0 1
0 0 1 1 1
0 0 0 0 0
0 0 0 0 0

0 0 0 0 0
0 1 0 0 0
0 0 1 0 0
1 1 1 0 0
0 0 0 0 0

0 0 0 0 0
0 0 1 0 0
0 0 0 1 0
0 1 1 1 0
0 0 0 0 0

0 0 0 0 0
0 0 0 1 0
0 0 0 0 1
0 0 1 1 1
0 0 0 0 0

0 0 0 0 0
0 0 0 0 0
0 1 0 0 0
0 0 1 0 0
1 1 1 0 0

0 0 0 0 0
0 0 0 0 0
0 0 1 0 0
0 0 0 1 0
0 1 1 1 0

0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 0 0 1
0 0 1 1 1
```

## Add the Rotated Arrays

Now, let’s reduce all these tables into a single table by adding up all the corresponding elements. To do this, we’ll use `+/` (sum)

```j
  +/ (> , {;~ 1 0 _1) |. R
    
0 1 1 1 0
0 1 2 2 1
1 3 5 4 2
1 2 4 3 2
1 2 3 2 1
```

## Living Cells in the Next Generation

The rules of [Conway’s Game of Life](https://en.wikipedia.org/wiki/Conway%27s_Game_of_Life) stipulate that a cell is alive in the next generation if:

1.  The cell is alive in the current generation and has `2` or `3` living neighbors
2. The cell is dead in the current generation and has `3` living neighbors

### Cells with Value of 3

Any cell with `3` will be alive in the next generation, regardless of whether it is alive in the current generation — either it is currently alive (`1`) and has `2` living neighbors, or it is currently dead (`0`) and has `3` living neighbors.

We can find all cells with a `3` by using `=` ([equal](http://code.jsoftware.com/wiki/Vocabulary/eq#dyadic)):

```j
  3 = +/ (> , {;~ 1 0 _1) |. R
    
0 0 0 0 0
0 0 0 0 0
0 1 0 0 0
0 0 0 1 0
0 0 1 0 0
```

We can turn the phrase `3 =` into a standalone verb by [currying](https://en.wikipedia.org/wiki/Currying) it. To curry a verb in J, we use `&` ([bond](http://code.jsoftware.com/wiki/Vocabulary/ampm)):

```j
  (3&=) +/ (> , {;~ 1 0 _1) |. R
    
0 0 0 0 0
0 0 0 0 0
0 1 0 0 0
0 0 0 1 0
0 0 1 0 0
```

We’ll use this bonded verb soon.

### Cells with Value of 4

A cell with `4` will be alive in the next generation if it is alive (`1`) in the current generation and has `3` living neighbors. On the other hand, a cell with `4` that is dead (`0`) in the current generation and has `4` living neighbors will be dead in the next generation.

Let’s first find the cells that have `4`

```j
  4 = +/ (> , {;~ 1 0 _1) |. R
    
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 1 0 0
0 0 0 0 0
```

Let’s bond `4 =`:

```j
  (4&=) +/ (> , {;~ 1 0 _1) |. R
    
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 1 0 0
0 0 0 0 0
```

Then we’ll use `*.` ([AND](http://code.jsoftware.com/wiki/Vocabulary/stardot#dyadic)) to select only those cells with `4` that are alive (`1`) in the current generation:

```j
  R *. (4&=) +/ (> , {;~ 1 0 _1) |. R
    
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 1 0 0
0 0 0 0 0
```

Let’s bond `R *.`:

```j
  (R&*.) (4&=) +/ (> , {;~ 1 0 _1) |. R
    
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 1 0 0
0 0 0 0 0
```

Let’s compose the two bonded verbs — `R&*.` and `4&=` into a single verb. In J, we do this with `@` ([atop](http://code.jsoftware.com/wiki/Vocabulary/at)):

```j
  (R&*.)@(4&=) +/ (> , {;~ 1 0 _1) |. R 
    
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 1 0 0
0 0 0 0 0
```

### Get the Cells with 3s OR 4s

The reason for the bonding and composing is that we can now combine both tests and apply them together using `+.` ([OR](http://code.jsoftware.com/wiki/Vocabulary/plusdot#dyadic)):

```j
  ((3&=) +. (R&*.)@(4&=)) +/ (> , {;~ 1 0 _1) |. R 
```

This returns the logical OR of the two tables we generated that checked for cells with `3` and the cells with `4` (that are currently alive).

The phrase `((3&=) +. (R&*.)@(4&=))` contains three verbs in a row `(3&=)`, `+.`, and `(R&*.)@(4&=)`.  In J, this is called a **[train](http://www.jsoftware.com/help/dictionary/dictf.htm)** and it has some rather deep and elegant interpretations. But that’s a story for a different time.

## Convert to Verb

Finally, we’ll convert to a standalone verb, with `y` representing the input array:

```j
  life =: verb define
  ((3&=) +. (y&*.)@(4&=)) +/ (> , {;~ 1 0 _1) |. y 
  )
```

## Results

Now we can apply our verb to the original input:

```j
  ]R =: 5 5 $ 0 0 0 0 0 0 0 1 0 0 0 0 0 1 0 0 1 1 1 0 0 0 0 0 0
    
0 0 0 0 0
0 0 1 0 0
0 0 0 1 0
0 1 1 1 0
0 0 0 0 0
    
  life R
    
0 0 0 0 0
0 0 0 0 0
0 1 0 1 0
0 0 1 1 0
0 0 1 0 0
```

We can apply `life` to the output of `life` as well:

```j
  life life R
    
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 1 0 1 0
0 0 1 1 0
```

Using `^:` ([fixed power](http://code.jsoftware.com/wiki/Vocabulary/hatco)), we can apply the same function to its own output as many times as we like. To get the same output as `life life R`, we can raise `life` to the second power.

```j
  life^:2 R
    
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 1 0 1 0
0 0 1 1 0
```

If we give `^:` an array, it will run once for each element and output the complete set of results. For `i.2` (the array `0 1`), it produces 2 generations (the original generation for `0` and the next generation for `1`:

```j
  life^:(i.2) R
    
0 0 0 0 0
0 0 1 0 0
0 0 0 1 0
0 1 1 1 0
0 0 0 0 0
    
0 0 0 0 0
0 0 0 0 0
0 1 0 1 0
0 0 1 1 0
0 0 1 0 0
```

Finally, if we run `life` 20 times, we can see that the glider will return to it’s original position in our 5x5 table:

```j
  life^:(i.20) R
    
0 0 0 0 0
0 0 1 0 0
0 0 0 1 0
0 1 1 1 0
0 0 0 0 0

0 0 0 0 0
0 0 0 0 0
0 1 0 1 0
0 0 1 1 0
0 0 1 0 0

0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 1 0 1 0
0 0 1 1 0

0 0 0 0 0
0 0 0 0 0
0 0 1 0 0
0 0 0 1 1
0 0 1 1 0

0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
0 0 0 0 1
0 0 1 1 1

0 0 0 1 0
0 0 0 0 0
0 0 0 0 0
0 0 1 0 1
0 0 0 1 1

0 0 0 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 1
0 0 1 0 1

0 0 0 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 1 0
1 0 0 0 1

1 0 0 1 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 1
1 0 0 0 0

1 0 0 0 1
0 0 0 0 1
0 0 0 0 0
0 0 0 0 0
1 0 0 1 0

1 0 0 1 0
1 0 0 0 1
0 0 0 0 0
0 0 0 0 0
1 0 0 0 0

1 1 0 0 0
1 0 0 0 1
0 0 0 0 0
0 0 0 0 0
0 0 0 0 1

0 1 0 0 0
1 1 0 0 1
0 0 0 0 0
0 0 0 0 0
1 0 0 0 0

0 1 0 0 1
1 1 0 0 0
1 0 0 0 0
0 0 0 0 0
0 0 0 0 0

0 1 0 0 0
0 1 0 0 1
1 1 0 0 0
0 0 0 0 0
0 0 0 0 0

1 0 0 0 0
0 1 1 0 0
1 1 0 0 0
0 0 0 0 0
0 0 0 0 0

0 1 0 0 0
0 0 1 0 0
1 1 1 0 0
0 0 0 0 0
0 0 0 0 0

0 0 0 0 0
1 0 1 0 0
0 1 1 0 0
0 1 0 0 0
0 0 0 0 0

0 0 0 0 0
0 0 1 0 0
1 0 1 0 0
0 1 1 0 0
0 0 0 0 0

0 0 0 0 0
0 1 0 0 0
0 0 1 1 0
0 1 1 0 0
0 0 0 0 0
```

## Coda

Thanks to [Jordan Scales](https://twitter.com/jdan), who provided the [code](https://github.com/jdan/j/blob/master/gameoflife.ijs) for this exploration. And thanks to [Words and Buttons](http://wordsandbuttons.online/apl_deserves_its_renaissance_too.html) for their APL Game of Life post that provided the inspiration for this J version.

_EOF_

