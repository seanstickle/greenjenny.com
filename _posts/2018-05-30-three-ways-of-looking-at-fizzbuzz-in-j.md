---
layout: post
title: Three Ways of Looking at FizzBuzz in J
---

With apologies to Wallace Stevens' [Thirteen Ways of Looking at a Blackbird](https://www.poetryfoundation.org/poems/45236/thirteen-ways-of-looking-at-a-blackbird).

## 1. The **wycdd** Way

From [Fizzbuzz in J, Explained](https://wycd.net/posts/2017-01-19-fizz-buzz-and-triangles-in-j.html) by [Wayne Chang](https://twitter.com/wycdd):

	  fb =: ('FizzBuzz';'Fizz';'Buzz';":){::0 i.15 3 5|]

### Create the Lookup List

Create a list of boxed items that contain all the possible return values:

- “Fizzbuzz” when divided by both 3 and 5
- “Fizz” when divided by 3
- “Buzz” when divided by 5
-  The input value when divided by none of the above

Use `;` ([link](http://code.jsoftware.com/wiki/Vocabulary/semi#dyadic)) to build the list of boxed items. To include the input value in the list of boxed items, use `":` ([default format](http://code.jsoftware.com/wiki/Vocabulary/quoteco)) to convert the number to a string:

	  fb =: 'FizzBuzz' ; 'Fizz' ; 'Buzz' ; ":
	  input =: 10
	  fb input
	
	+--------+----+----+--+
	|FizzBuzz|Fizz|Buzz|10|
	+--------+----+----+--+

### Get the Residues

Take the input number and find out if it’s divisible by 3, 5, or 15. To do this, use `|` ([residue](http://code.jsoftware.com/wiki/Vocabulary/bar#dyadic)), which returns the remainder when dividing by a given number:

	  n =: 9
	  15 3 5 | n
	
	9 0 4
	
	  n =: 10
	  15 3 5 | n
	
	10 1 0
	
	  n =: 30
	  15 3 5 | n
	
	0 0 0

For `3` and `5`, there’s only a single `0` returned. For `15`, since all three numbers divide evenly, there are three `0`s returned. If we find the index of the **first** `0`, we’ll know which number was the greatest divisor with no residue.

### Get the Index

Use `i.` ([index of](http://code.jsoftware.com/wiki/Vocabulary/idot#dyadic)) to find the index of the first `0`. Normally, `i.` operates like `x i. y`, returning the index of the first occurrence of `y` in `x`.

	  n =: 9
	
	  ]residues =: 15 3 5 | n
	
	9 0 4
	
	  residues i. 0
	
	1

We can also use `~` ([passive](http://code.jsoftware.com/wiki/Vocabulary/tilde#dyadic)) to reverse the order, so that it operates like `y. i. x`, allowing us to be more concise:

	  n =: 9
	
	  ]residues =: 15 3 5 | n
	
	9 0 4
	
	  0 i.~ residues
	
	1
	
	  0 i.~ 15 3 5 | n
	
	1

Important note. If the value we’re looking for in the list doesn’t appear, `i.` returns an index that exceeds the size of the list we’re examining. For instance, try finding the index of the first occurrence of either `0` or `19` in the list `5 6 7`:

	  0 i.~ 5 6 7
	
	3
	
	  19 i.~ 5 6 7
	
	3

In a list with only 3 items, index 3 does not exist. This will be useful in the next step.

### Use the Index to Fetch From the Lookup List

We now have a lookup list:

	  fb =: 'FizzBuzz' ; 'Fizz' ; 'Buzz' ; ":
	  n =: 10
	  fb n
	
	+--------+----+----+--+
	|FizzBuzz|Fizz|Buzz|10|
	+--------+----+----+--+

And an index based on the residues of dividing by 15, 3, and 5:

	  n =: 10
	  0 i.~ 15 3 5 | n
	
	2

Use the index to fetch the appropriate value from the lookup list. To do this, use `{::` ([fetch](http://code.jsoftware.com/wiki/Vocabulary/curlylfcoco#dyadic)):

	  n =: 3
	  NB. First occurence of `0` would be at index `0`
	  (0 i.~ 15 3 5 | input) {:: 'FizzBuzz' ; 'Fizz' ; 'Buzz' ; ": n
	
	Fizz
	
	  n =: 5
	  NB. First occurence of `0` would be at index `1`
	  (0 i.~ 15 3 5 | input) {:: 'FizzBuzz' ; 'Fizz' ; 'Buzz' ; ": n
	
	Buzz
	
	  n =: 15
	  NB. First occurence of `0` would be at index `2`
	  (0 i.~ 15 3 5 | input) {:: 'FizzBuzz' ; 'Fizz' ; 'Buzz' ; ": n
	
	FizzBuzz
	
	  n =: 7
	  NB. No residues of `0`, so i. returns `3`
	  (0 i.~ 15 3 5 | input) {:: 'FizzBuzz' ; 'Fizz' ; 'Buzz' ; ": n
	
	7

### Simplify with a Fork

In J, there is a pattern called a **fork**, which looks like this:

	  f g h y

Where `f` is a verb, `g` is a verb, `h` is a verb, and `y` is a noun. _Note: For those new to J, read “verb” as function and “noun” as “variable”._

Forks are evaluated like this:

	 (f y) g (h y)

The canonical example in J is calculating the mean of a set of numbers:

	 (+/ % #) y

Which is evaluated as:

	  (+/ y) % (# y)

Which is: the sum of y divided by the count of y.

In our example, we have this code:

	  (0 i.~ 15 3 5 | n) {:: 'FizzBuzz' ; 'Fizz' ; 'Buzz' ; ": n

Using a fork, we can extract the variable and rewrite this as:

	  fb =: (0 i.~ 15 3 5 | ]) {:: 'FizzBuzz' ; 'Fizz' ; 'Buzz' ; ":

Notice that we’ve introduced a new verb: `]` ([right](http://code.jsoftware.com/wiki/Vocabulary/squarert#dyadic)). We extracted the variable `n`, and `|` ([residue](http://code.jsoftware.com/wiki/Vocabulary/bar#dyadic)) expects two arguments (one on either side), so we had to put in `]` as a placeholder for whatever value we pass to `fb`. There’s more nuance to this that I’ll explain in another post.

### Reverse the Order

We could stop here, as the new verb works great:

	  fb 1
	
	1
	
	  fb 3
	
	Fizz
	
	  fb 5
	
	Buzz
	
	  fb 15
	
	FizzBuzz

To get to **wycdd**’s original version, we just need to use `~` ([passive](http://code.jsoftware.com/wiki/Vocabulary/tilde#dyadic)) again to reverse the order that `{::` is applied:

	  fb =: ('FizzBuzz' ; 'Fizz' ; 'Buzz' ; ":) {::~ 0 i.~ 15 3 5 | ]

### Run the Code

Our new verb `fb` only runs on single integers. If we want to apply it to the numbers `1` to `100`, we’ll need to apply it to a whole list. To do that, use `"0`, which tells J to apply the verb to individual atoms of a list. And we’ll use `i.` ([integers](http://code.jsoftware.com/wiki/Vocabulary/idot)) to generate a list from `0` to `99` (and add `1` to get a list of `1` to `100`):

	  fb"0 (1+i.100)
	
	1
	2
	Fizz
	4
	Buzz
	Fizz
	7
	8
	Fizz
	Buzz
	11
	Fizz
	13
	14
	FizzBuzz
	16
	...


## 2. Rosetta Code Solution 0

From [Solution 0](https://rosettacode.org/wiki/FizzBuzz#J) on Rosetta Code’s [FizzBuzz page](https://rosettacode.org/wiki/FizzBuzz):

	  > }. (<'FizzBuzz') (I.0=15|n)} (<'Buzz') (I.0=5|n)} (<'Fizz') (I.0=3|n)} ":&.> n=: i.101

### Start with the Input List

Create a list from `0` to `100` using `i.` ([integers](http://code.jsoftware.com/wiki/Vocabulary/idot)):

	  ]n =: i.101
	
	0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23 24 25 26 27 28 29 30 31 32 33 34 35 36 37 38 39 40 41 42 43 44 45 46 47 48 49 50 51 52 53 54 55 56 57 58 59 60 61 62 63 64 65 66 67 68 69 70 71 72 73 74 75 76 77 78 79 80 81 82 83 84 85 86 87 88...

### Box It Up

Turn the numeric list into a boxed list. To do this, use `<` ([box](http://code.jsoftware.com/wiki/Vocabulary/lt)). By default, `<` will act on the whole list:

	  < n =: i.101
	
	+-----------------------+
	|0 1 2 3 4 5 6 7 8 9 ...|
	+-----------------------+

So we’ll use `"` ([rank](http://code.jsoftware.com/wiki/Vocabulary/quote)) to apply `<` to each atom (rank `0`) of the list:

	  <"0 n =: i.101
	
	+-+-+-+-+-+-+-+-+-+-+---+
	|0|1|2|3|4|5|6|7|8|9|...|
	+-+-+-+-+-+-+-+-+-+-+---+

### Find Numbers Divisible by 3

Find the indexes of all the elements of `n` that are evenly divisible by `3`. 

To do this, use `|` ([residue](http://code.jsoftware.com/wiki/Vocabulary/bar#dyadic)) to return the remainders of each number when divided by `3`.

	  3 | n
	
	0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1 2 0 1

Then, check whether each residues are equal to `0` using `=` ([equal](http://code.jsoftware.com/wiki/Vocabulary/eq#dyadic)):

	  0 = 3 | n
	
	1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0 0 1 0

Finally, find the index of every `1` in that boolean list:

	  I. 0 = 3 | n
	
	0 3 6 9 12 15 18 21 24 27 30 33 36 39 42 45 48 51 54 57 60 63 66 69 72 75 78 81 84 87 90 93 96 99

### Amend the Boxed List

With the indexes of all the values that are evenly divisible by `3`, we can update the values the corresponding boxes in our boxed list.

To do this, use `}` ([amend](http://code.jsoftware.com/wiki/Vocabulary/curlyrt#dyadic)). This takes three arguments: `x m } y`, where `y` is the list to update, `m` is an array of cell indexes to update, and `x` is the new value to use in those cells:

	  y =: <"0 n =: i.101
	  m =: I. 0 = 3 | n
	  x =: <'Fizz'
	  x m } y
	
	+----+-+-+----+-+-+----+-+-+----+---+
	|Fizz|1|2|Fizz|4|5|Fizz|7|8|Fizz|...|
	+----+-+-+----+-+-+----+-+-+----+---+

Notice that, since we’re updating a boxed list, the value `Fizz` has to be boxed (`<`) as well.

We’ll put this all in one line like this:

	  (<'Fizz') (I. 0 = 3 | n) } <"0 n =: i.101
	
	+----+-+-+----+-+-+----+-+-+----+---+
	|Fizz|1|2|Fizz|4|5|Fizz|7|8|Fizz|...|
	+----+-+-+----+-+-+----+-+-+----+---+

### Repeat for 5 and 15

We can do the same “check and amend” process for both `5` and `15`:

	  (<'Buzz') (I. 0 = 5 | n) } <"0 n =: i.101
	
	+----+-+-+-+-+----+-+-+-+-+----+--+--+---+
	|Buzz|1|2|3|4|Buzz|6|7|8|9|Buzz|11|12|...|
	+----+-+-+-+-+----+-+-+-+-+----+--+--+---+

	  (<'FizzBuzz') (I. 0 = 15 | n) } <"0 n =: i.101
	
	+--------+-+-+-+-+-+-+-+-+-+--+--+--+--+--+--------+--+--+---+
	|FizzBuzz|1|2|3|4|5|6|7|8|9|10|11|12|13|14|FizzBuzz|16|17|...|
	+--------+-+-+-+-+-+-+-+-+-+--+--+--+--+--+--------+--+--+---+

### Combine the Amendments

J evaluates from right to left, the output of one phrase becoming the input of the next. A simple arithmetic example:

	  6 + 5 * 4 + 3
	
	41

Which evaluates as:

	  6 + (5 * (4 + 3))
	
	41

In the same way, we can chain our “check and amends” so that the output of one is the input of the next:

	  (<'FizzBuzz') (I. 0 = 15 | n) } (<'Buzz') (I. 0 = 5 | n) } (<'Fizz') (I. 0 = 3 | n) } <"0 n =: i.101
	
	+--------+-+-+----+-+----+----+-+-+----+----+--+----+--+--+--------+---+
	|FizzBuzz|1|2|Fizz|4|Buzz|Fizz|7|8|Fizz|Buzz|11|Fizz|13|14|FizzBuzz|...|
	+--------+-+-+----+-+----+----+-+-+----+----+--+----+--+--+--------+---+

### Cut Off the Head

Since we want the results for `1` to `100`, we don’t want the first cell in the boxed list. To remove this, use `}.` ([behead](http://code.jsoftware.com/wiki/Vocabulary/curlyrtdot)):

	}. (<'FizzBuzz') (I. 0 = 15 | n) } (<'Buzz') (I. 0 = 5 | n) } (<'Fizz') (I. 0 = 3 | n) } <"0 n =: i.101
	
	+-+-+----+-+----+----+-+-+----+----+--+----+--+--+--------+---+
	|1|2|Fizz|4|Buzz|Fizz|7|8|Fizz|Buzz|11|Fizz|13|14|FizzBuzz|...|
	+-+-+----+-+----+----+-+-+----+----+--+----+--+--+--------+---+

### Unboxing Day  

Now we’ll unbox everything so that we have a simple list. To do this, use `>` ([open](http://code.jsoftware.com/wiki/Vocabulary/gt)):

	> }. (<'FizzBuzz') (I. 0 = 15 | n) } (<'Buzz') (I. 0 = 5 | n) } (<'Fizz') (I. 0 = 3 | n) } <"0 n =: i.101
	
	|domain error
	|       >}.(<'FizzBuzz')(I.0=15|n)}(<'Buzz')(I.0=5|n)}(<'Fizz')(I.0=3|n)}<"0 n=:i.101

Uh oh. This problem is caused because the contents of the boxed list are a combination of `literals` (characters) and `integers`. We can’t create a simple list that contains both.

We’ll need to convert all the numbers to characters.

### Convert to Characters

The easiest place to do that is when we boxed them up in the first place.

So, instead of doing this:

	  <"0 n =: i.101

We’ll do this:

	  ":&.> n =: i.101

This uses `&.` ([under](http://code.jsoftware.com/wiki/Vocabulary/ampdot)), which takes three arguments: `u &. v y`, where `u` is a verb, `v` is a verb, and `y` is a variable. 

For `u`, we’ll use `":` ([default format](http://code.jsoftware.com/wiki/Vocabulary/quoteco)), which converts numbers to characters. For `v`, we’ll use `>` ([open](http://code.jsoftware.com/wiki/Vocabulary/gt)).

The order of operation is a little complicated:

1. Executes `v` on `y` cell-by-cell
2. For each cell, `u` is applied to the result of `v`
3. Then `v^:_1` (i.e. the obverse of v) is applied to the result of `u` — giving the result for that cell.

So, for us, this means:

1. Execute `>` ([open](http://code.jsoftware.com/wiki/Vocabulary/gt)) on each cell of `y` (`i.101`). These are already unboxed, so this doesn’t do anything.
2. Execute `":` to the result, which converts the numbers to characters.
3. Execute `>^:1`, that is, the inverse of `>` ([open](http://code.jsoftware.com/wiki/Vocabulary/gt)), which is the same as `<` ([box](http://code.jsoftware.com/wiki/Vocabulary/lt)).

```
  ":&.> n =: i.101
	
+-+-+-+-+-+-+-+-+-+-+---+
|0|1|2|3|4|5|6|7|8|9|...|
+-+-+-+-+-+-+-+-+-+-+---+
```

This _looks_ the same as our original, but the values inside those boxes are different.

Here’s the original and the updated boxed lists:

	  original =: <"0 n =: i.101
	  updated  =: ":&.> n =: i.101

If we use `{` ([from](http://code.jsoftware.com/wiki/Vocabulary/curlylf#dyadic)) to take the first element of each boxed list, then use `>` ([open](http://code.jsoftware.com/wiki/Vocabulary/gt)) to unbox it, we can then use `datatype` to identify the contents:

	  > 1 { original
	
	1
	
	  datatype > 1 { original
	
	integer
	
	  > 1 { updated
	
	1
	
	  datatype > 1 { original
	
	literal

### Unboxing Day, Take 2

Updating the code to ensure the contents of the boxed list are all converted to `literals`, we can now try to unbox the results again:

	  > }. (<'FizzBuzz') (I. 0 = 15 | n) } (<'Buzz') (I. 0 = 5 | n) } (<'Fizz') (I. 0 = 3 | n) } ":&.> n =: i.101
	
	1
	2
	Fizz
	4
	Buzz
	Fizz
	7
	8
	Fizz
	Buzz
	11
	Fizz
	13
	14
	FizzBuzz
	16
	...


## 3. Rosetta Code Solution 1

From [Solution 0](https://rosettacode.org/wiki/FizzBuzz#J) on Rosetta Code’s [FizzBuzz page](https://rosettacode.org/wiki/FizzBuzz):

	  Fizz=: 'Fizz' #~ 0 = 3&|
	  Buzz=: 'Buzz' #~ 0 = 5&|
	  FizzBuzz=: ": [^:('' -: ]) Fizz,Buzz

### Fizz for 3

For a number, see if it’s evenly divisible by `3`. We can do this in the same way we’ve done previously, using `|` ([residue](http://code.jsoftware.com/wiki/Vocabulary/bar#dyadic)) and then comparing the result to `0` using `=` ([equal](http://code.jsoftware.com/wiki/Vocabulary/eq#dyadic)):

	  n =: 9
	  0 = 3 | n
	
	1

If the result is `1`, we’ll return `Fizz`; if the result is `0`, we’ll return nothing.

We’ll do this by using `#` ([copy](http://code.jsoftware.com/wiki/Vocabulary/number#dyadic)), which takes two arguments: `x # y`, where `y` is the thing you want copied, and `x` is the number of copies you want. For example, we can create 10 copies of `1`:

	  10 # 1
	
	1 1 1 1 1 1 1 1 1 1

In our case, we’ll make either `0` or `1` copies of `Fizz`, depending on whether the residue is equal to `0` or not:

	  n =: 9
	  (0 = 3 | n) # 'Fizz'
	
	Fizz
	
	  n =: 10
	  (0 = 3 | n) # 'Fizz'
	

To make this read a little easier, use `~` ([passive](http://code.jsoftware.com/wiki/Vocabulary/tilde#dyadic)) with `#` ([copy](http://code.jsoftware.com/wiki/Vocabulary/number#dyadic)) to flip the order or the arguments:

	  n =: 9
	  'Fizz' #~ 0 = 3 | n
	
	Fizz

We can extract the variable and turn this into a standalone verb:

	  Fizz =: 'Fizz' #~ 0 = 3&|

Notice that to extract the variable, we used `&` ([bond](http://code.jsoftware.com/wiki/Vocabulary/ampm)) to join an argument (`3`) to the primitive verb `|` ([residue](http://code.jsoftware.com/wiki/Vocabulary/bar#dyadic)). This is how J implements [currying](https://en.wikipedia.org/wiki/Currying), allowing us to pend the full evaluation of the verb until we receive the second argument.

If we don’t **bond** `3` to `|`, we’d get a syntax error when trying to define this verb:

	  Fizz =: 'Fizz' #~ 0 = 3 |
	
	|syntax error
	|   Fizz=:    'Fizz'#~0=3|

### Buzz for 5

We’ll do the same thing for `5`:

	  Buzz =: 'Buzz' #~ 0 = 5&|

Easy!

### Forking Fizz and Buzz

Let’s apply both `Fizz` and `Buzz` to a given input. We’ll use `,` ([append](http://code.jsoftware.com/wiki/Vocabulary/comma#dyadic)) to create a list from the results of the two verbs:

	  n =: 3
	  (Fizz n) , (Buzz n)
	
	Fizz
	
	  n =: 5
	  (Fizz n) , (Buzz n)
	
	Buzz
	
	  n =: 15
	  (Fizz n) , (Buzz n)
	
	FizzBuzz
	
	  n =: 17
	  (Fizz n) , (Buzz n)
	 

Remember the **fork** pattern from above?

	  f g h y

Which evaluates to:

	 (f y) g (h y)

We can use the fork pattern to simplify our code. Instead of:

	  (Fizz n) , (Buzz n)

We can use:

	  (Fizz , Buzz) n

### Is It Empty?

Executing `(Fizz , Buzz)` on the list of `1` to `100` gives us:

	  (Fizz , Buzz) "0 (1+i.100)
	
	
	Fizz
	
	Buzz
	Fizz
	
	
	Fizz
	Buzz
	
	Fizz
	
	
	FizzBuzz
	
	
	Fizz
	...

We’re returning the right results for numbers divisible by `3`, `5`, and `15`. We still need to return the input number when not divisible by `3`, `5`, and `15`.

To do this, test to see if the value returned is blank. Use `-:` ([match](http://code.jsoftware.com/wiki/Vocabulary/minusco#dyadic)), which takes two arguments, `x -: y`, returning `1` if `x` and `y` match and returning `0` otherwise:

	  n =: 10
	  '' -: (Fizz , Buzz) n
	
	0
	
	  n =: 7
	  '' -: (Fizz , Buzz) n
	
	1

We can use this `1` or `0` to decide whether to return the input number or the result of `(Fizz , Buzz)`.

To simplify the code, we ca turn this into a standalone verb by extracting the `(Fizz , Buzz) n`:

	  is_empty =: '' -: ]
	  is_empty (Fizz , Buzz) 10
	
	0
	
	  is_empty (Fizz , Buzz) 7
	
	1

Note that `-:`  ([match](http://code.jsoftware.com/wiki/Vocabulary/minusco#dyadic)) expects two arguments, so we’re using `]` ([right](http://code.jsoftware.com/wiki/Vocabulary/squarert#dyadic)) to stand in for what we’ll pass as input.

### Fix the Power

We’ll do this by using `^:` ([fixed power](http://code.jsoftware.com/wiki/Vocabulary/hatco)). This verb takes three arguments, `u ^: n y`, where `u` is applied `n` times to `y`. It’s crucial to know that applying `u` zero times to `y` just returns the original value `y`.

For example, using `*:` ([square](http://code.jsoftware.com/wiki/Vocabulary/starco)):

	  square =: *:
	  square 5
	
	25
	
	  (square^:1) 5
	
	25
	
	  (square^:0) 5
	
	5

We can combine this use of **fixed power** with another `[` ([left](http://code.jsoftware.com/wiki/Vocabulary/squarelf#dyadic)), a verb that takes two arguments, `x [ y` and always returns the left argument.

	  1 [ 2
	
	1

Repeated applications of `[` ([left](http://code.jsoftware.com/wiki/Vocabulary/squarelf#dyadic)) do the same thing:

	  1 [ [ 2
	
	1

And repeated applications is what `^:` ([fixed power](http://code.jsoftware.com/wiki/Vocabulary/hatco)) does:

	  1 [^:(1) 2
	
	1
	
	  1 [^:(2) 2
	
	1

But, using the crucial insight from above that raising a verb to the fixed power of `0` just returns the original input:

	  1 [^:(0) 2
	
	2

This gives us a switch to select the left or the right input, depending on whether we raise `[` ([left](http://code.jsoftware.com/wiki/Vocabulary/squarelf#dyadic)) to `0` or `1`:

	  1 [^:(1) 2
	
	1
	
	  1 [^:(0) 2
	
	2

This, along with the test we wrote for whether `(Fizz , Buzz)` returns a blank, allows us to write this:

	  n [ ^:('' -: ]) (Fizz , Buzz) n

If the result of `(Fizz , Buzz)` is blank, then `('' -: ])` returns `1`, which raises `[` to the `1` power, which returns what’s on the left, which is `n`, the input.

If the result of `(Fizz , Buzz)` is not blank (i.e., **Fizz**, **Buzz**, or **FizzBuzz**), then `('' -: ])` returns `0`, which raises `[` to the `0` power, which returns the original input, which is the output of `(Fizz , Buzz)`.

### Hook, Line, Sinker

Notice that our code:

	  n =: 10
	  n [ ^:('' -: ]) (Fizz , Buzz) n
	
	Buzz

Can be rewritten as:

	  f =: [ ^:('' -: ])
	  g =: Fizz , Buzz
	  y =: n
	
	  y f g y
	
	Buzz

In J, the `y f g y` pattern is known as a `hook`, and it can be written more simply as:

	  (f g) y
	
	Buzz

In combinatory logic, this is known as the [S combinator](https://en.wikipedia.org/wiki/SKI_combinator_calculus).

Rewriting the entire sentence using the `hook` structure, we get:

	  n =: 10
	  ([ ^:('' -: ]) Fizz , Buzz) n
	
	Buzz

We can define this as a standalone verb:

	  fb =: [ ^:('' -: ]) Fizz , Buzz
	  fb 3
	
	Fizz
	
	  fb 5
	
	Buzz
	
	  fb 15
	
	FizzBuzz
	
	  fb 17
	
	17

###  Apply to the List

Now apply `fb` to the list `1` to `100`, using `"0`  ([rank](http://code.jsoftware.com/wiki/Vocabulary/quote)) to apply the verb to individual atoms of the list:

	  fb"0 (1+i.100)
	
	|domain error
	|       fb"0(1+i.100)

Oops! This is the same error that we got back in the second way to look at FizzBuzz. We’re mixing `literals` and `integers`, which raises a `domain error`. To fix that, we need to convert all the results to `literals`.

Do that by applying `":` ([default format](http://code.jsoftware.com/wiki/Vocabulary/quoteco)) to the results of our verb, which will convert the `integers` to `literals`:

	 fb =: ": [ ^:('' -: ]) Fizz , Buzz

Now we can try again:

	  fb"0 (1+i.100)
	
	1
	2
	Fizz
	4
	Buzz
	Fizz
	7
	8
	Fizz
	Buzz
	11
	Fizz
	13
	14
	FizzBuzz
	16
	...

Perfect.

### Coda

It’s interesting to observe that adding the `":` ([default format](http://code.jsoftware.com/wiki/Vocabulary/quoteco)) to the verb:

	 fb =: ": [ ^:('' -: ]) Fizz , Buzz

Converted it from a **hook** into a **fork**: 

	  f =: ":
	  g =: [ ^:('' -: ])
	  h =: Fizz , Buzz
	
	  (f g h) 15
	
	FizzBuzz
	
	  (f g h) 17
	
	17

Which is evaluated as:

	  (f 15) g (h 15) 
	
	FizzBuzz
	
	  (f 17) g (h 17)
	
	17

_EOF_

