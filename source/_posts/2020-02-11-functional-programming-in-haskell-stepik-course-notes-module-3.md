---
title: Functional Programming in Haskell - Stepik course notes - Lists
date: 2020-02-11 10:02:15
tags:
---

This is the third module (out of 5) of my English summary of the Haskell MOOC on Stepik, available only in Russian. Read the previous summaries in the links below.

1. [Introduction](/2020/01/functional-programming-in-haskell-stepik-course-notes-module-1/)
2. [Programming fundamentals](/2020/01/functional-programming-in-haskell-stepik-course-notes-module-2/)
3. **Lists** (this page)
4. Data types
5. Monads

<!-- more -->

## Lists

Lists are a fundamental data structure in functional programming, similar to arrays in imperative languages. Whereas an array in imperative languages is a consecutive chunk of memory containing some elements, lists in functional languages express a recursive data structure.

### Working with lists

We already encountered in previous modules ways to create a list, such as concatenating two existing lists together. We'd like to define some basic operations on lists, and express all others via those basic operations. There are two such operations: constructing an empty list and adding (prepending) an element to the head of the list. Let's look at a few examples:

#### List construction

To create an empty list we use the square brackets `[]`:

```
> []
[]
```

To add an element to the head of the list we use the `:` operator (pronounced *cons* for *construction*):

```
> 3 : []
[3]
```

Here, appending 3 to the head of an empty list results in a single-element list `[3]`. We can also prepend several elements:

```
> lst = 5 : 3 : []
> lst
[5,3]
```

The `:` operator is right-associative, therefore no braces are required: the expression is evaluated as `5 : (3 : [])`.

The way Haskell displays lists with square brackets is syntactic sugar for the list construction syntax with the `:` operator. They are equivalent:

```
> [5,3] == lst
True
```

The empty list `[]` and the binary `:` operator are called *data constructors*, and they are the means to create a list in Haskell. We can use them as any other operator. Consider a function that always adds the value 42 to the head of the list:

```
> cons42 = (42 :)
> :t cons42
cons42 :: [Integer] -> [Integer]
> cons42 [1,2,3]
[42,1,2,3]
```

We use operator *sectioning* to partially apply 42 to the `:` operator, resulting in a function that can be applied to a list.

#### List deconstruction

The opposite operation of constructing a list is called *deconstruction*. Recall that for pairs, constructed with the `(,)` operator, we used the `fst` and `snd` functions to deconstruct the pair, returning the first and second elements, respectively. Similarly, a list can be constructed using the `[]` and `:` operators, and deconstructed using two functions `head` and `tail`:

The `head` function returns the element at the head of the list (the first element):

```
> :t head
head :: [a] -> a
> head [1,2,3]
1
```

The `tail` function returns the remainder, i.e. everything except the head element of the list:

```
> :t tail
tail :: [a] -> [a]
> tail [1,2,3]
[2,3]
```

Both `head` and `tail` are so-called *partial functions*, they are only defined for lists containing at least one element. Applying `head` to an empty list, for example, results in an error:

```
> head []
*** Exception: Prelude.head: empty list
```

The `head` and `tail` functions allow us to deeply inspect the list structure. Let's look at a few examples:

```haskell
second :: [a] -> a
second xs = head (tail xs)
```

The function `second` will return the second element of the list. It works in the following manner: it takes a list, specified here by the name `xs` (pronounced *exes*, which is a convention used in Haskell - `xs` is a *list of `x`*, a plural of `x`). We first get the `tail` of `xs`, and apply `head` to the result:

```
> second [1,2,3]
2
```

Just like `head` and `tail`, the `second` function is also a *partial function*, which requires a list with at least two elements, otherwise, it fails:

```
> second [1]
*** Exception: Prelude.head: empty list
```

The error message comes from our use of `head`, and it might be better to handle the error case explicitly, providing a better message.

We could also rewrite the `second` function in the *point-free* style, by using function composition and omitting the `xs` argument on both sides:

```haskell
second :: [a] -> a
second = head . tail
```

A much more convenient mechanism for deconstructing lists is *pattern matching*. We've already encountered pattern matching in previous modules, working with numbers. Pattern matching on more complex structures works similarly, but instead, we use constructors as patterns.

Let's rewrite the `fst` function on pairs using the pattern matching syntax:

```
> fst' ((,) x y) = x
> fst' ("Hi",3)
"Hi"
```

Here we define a function `fst'` (to avoid clashing with the built-in `fst`) that takes as an argument a *pattern* for a prefix-style construction of a pair, namely `(,) x y`, returning the first argument `x`. When applied to an actual pair value `("Hi",3)`, it behaves as expected, returning the first result.

Let's write a similar implementation of `head'` for lists:

```
> head' ((:) x xs) = x
> head' [1,2,3]
1
```

Here we are using the list constructor `:` in a prefix style. When passing our `head'` function an actual list, it binds to the pattern `(:) x xs`, breaking to the first element `x` and the remainder `xs`. We can also rewrite it in a more convenient infix-style. Let's define the `tail'` function:

```
> tail' (x : xs) = xs
> tail' [1,2,3]
[2,3]
```

Notice that the argument `x` is not used in the function body. We can, therefore, omit it, replacing it by `_`:

```
> tail'' (_ : xs) = xs
> tail'' [1,2,3]
[2,3]
```

Finally, let's rewrite the `second` function using pattern matching:

```haskell
second' :: [a] -> a
second' (_ : xs) = head xs
```

The argument to the `second'` function is deconstructed to a first element (which we omit) and the remainder `xs`, and the function `head` is applied to `xs`, giving us the correct result:

```
> second' [1,2,3]
2
```

Using pattern matching we can remove the call to `head` entirely, by deconstructing the list further:

```haskell
second'' :: [a] -> a
second (_ : x : _) = x
```

Here, we're deeply inspecting the list by breaking it to the first and second elements, followed by the remainder. Since we don't care about the first element and the remainder, we substitute them with `_`, resulting in a pattern that matches a list with at least two values, binding `x` to the second, and returning it.

```
> second'' [1,2,3]
2
```

#### Recursion over lists

In previous examples, we've seen how using pattern matching we can inspect a list several levels deep. However, there exists a more powerful way of inspecting a list, by using recursion. With recursion, we can traverse the entire list until it ends, inspecting each element along the way. 

Previously, we used the `:` constructor in pattern matching to break apart the *head* and the *tail* of a list, resulting in a tail that is one element shorter than the previous list. We can continue performing the recursive operation until the tail becomes empty. All we need to do, in this case, is to provide an additional pattern match on an empty list, guaranteeing a terminating condition for our recursion.

Let's define a function `length`, giving us the number of elements in the list. Since this function already exists in the *Prelude*, we will explicitly import the Prelude, *hiding* existing implementations:

```haskell
import Prelude hiding (length,(++),null)
```

Our `length` function can be defined as follows:

```haskell
length :: [a] -> Int
length []     = 0
length (x:xs) = 1 + length xs
```

The length function is defined using two equations: one for the empty list constructor, and one for the non-empty list (`:`) constructor. The first case matches against an empty list - the length of an empty list is always 0. The second case matches a non-empty list. It breaks it to the head and the tail, we're calling the `length` function recursively on the tail and adding 1 for each element we encounter. This happens until we exhaust all elements, terminating with the empty case. Since we're not using the `x` argument, we can replace it with an underscore `_`:

```
length (_:xs) = ...
```

Let's look at another, slightly more complicated example - concatenating two lists:

```haskell
(++) :: [a] -> [a] -> [a]
[] ++ ys     = ys
(x:xs) ++ ys = x : xs ++ ys
```

The concatenation operation works as follows: If the first argument is an empty list, we return the second list `ys` untouched as the result - there's nothing to do. Otherwise, we will break the first list to the head `x` and the tail `xs`, and reconstruct it by recursively applying `++` on the tail `xs` and `ys`, prepending `x` to the result. Recall that the `:` operator is right-associative, so the operation is performed like this:

```
x : (xs ++ ys)
```

What is the complexity of this function? If the first list has *N* elements, and the second has *M* elements, the function has linear complexity O(N). Notice that the second list `ys` is reused as-is. Since lists in Haskell are immutable, it's safe to reuse any structure without worrying that it might change from the outside.

The third example is another function from the standard library, `null`, which checks whether the list is empty:

```haskell
null :: [a] -> Bool
null []  = True
null _   = False
```

It returns `True` for an empty list and `False` in any other case.

#### Recursion over lists (2)

Until now we've used recursion over lists, having an empty list as the terminating condition. However, this is not the only possible way to use recursion with lists. Sometimes, we'd like to terminate on some other condition, such as reaching a single-element list. Let's look at a few examples:

```haskell
import Prelude hiding (last,init,reverse)
```

Let's start with the `last` function, returning the last element of the list. In Haskell, this requires us to traverse all elements of the list, until we reach the last one, resulting in a linear complexity O(N).

```haskell
last :: [a] -> a
last (x:[]) =  x
last (_:xs) =  last xs
```

Here, our terminating condition is a list that has a head `x` and an empty list in the tail, guaranteeing this to be a single-element list. The "last" element of a single-element list is that element, so we return `x`. In the recursive case, we throw away all elements except the last one, recursing on the tail of the list, until there's only one element left.

The second function is called `init`. It is similar to `head`, but works in the opposite: it returns all elements except the last one:

```haskell
init :: [a] -> [a]
init [x]    =  []
init (x:xs) =  x : init xs
```

Here, the terminating condition matches against a list containing exactly one element (this is a more compact form of pattern matching against a single-element list, compared to the previous example). In the case of a single-element list, we throw the element away, returning an empty list. In the recursive case, we rebuild the list until we reach the last element, throwing it away. Since we're not using the `x` argument in the first case, we can omit it by writing:

```
init [_] = ...
```

Applying the function to a non-empty list gives us the desired result:

```
> init [1,2,3]
[1,2]
```

Just like all previous functions on lists, `init` is a partial function (or, not a *total function*), which will fail on an empty list. We could augment our definition with a more detailed error message in case of an empty list:

```haskell
init :: [a] -> [a]
init []  = error "List must not be empty"
init ...
```

The next function is `reverse`, which reverses the elements of the list. In this case, we'll need an additional list to store our reversed elements, as we're traversing the original list. We can use a helper function `rev` with an accumulating parameter:

```haskell
reverse :: [a] -> [a]
reverse l   = rev l [] where
  rev []     a = a
  rev (x:xs) a = rev xs (x:a)
```

Resulting in:

```
> reverse "ABCD"
"DCBA"
```

#### Recursion over multiple lists

If we have functions that operate on multiple lists, we can use pattern matching to break each list to its head and tails, respectively. Let's see a few examples:

```haskell
import Prelude hiding (zip,zip3,unzip)
```

The first function is `zip`, which takes two lists, and produces a list of pairs, combining each head from each list into pairs:

```
> zip [1,2,3] "Hello"
[(1,'h'),(2,'e'),(3,'l')]
```

As soon as one of the lists ends, the `zip` function stops. The `zip` function is implemented as follows:

```haskell
zip :: [a] -> [b] -> [(a,b)]
zip [] _          = []
zip _ []          = []
zip (a:as) (b:bs) = (a,b) : zip as bs
```

Here, the first two conditions check whether the first or the second lists are empty, in which case the result is also an empty list. In all other cases, the first and the second lists are broken to the heads and tails, and heads are combined into a pair. The pair is prepended recursively until one of the lists becomes empty.

The `zip` function can be generalized. Let's look at `zip3`, a function of 3 list arguments:

```haskell
zip3 :: [a] -> [b] -> [c] -> [(a,b,c)]
zip3 (a:as) (b:bs) (c:cs) = (a,b,c) : zip as bs cs
zip3 _      _      _      = []
```

Similarly to `zip`, `zip3` returns a 3-element tuple (triple), if there are elements in all three lists. Otherwise, the function will return an empty list and will finish.

The opposite operation of `zip` is `unzip`: it takes a list of pairs, and produces a pair of lists, each containing the first and second elements from the pairs, respectively:

```haskell
unzip :: [(a,b)] -> ([a],[b])
unzip []          = ([],[])
unzip ((x,y):xys) =
  let (xs,ys) = unzip xys
  in  (x:xs,y:ys)
```

Similar to all the above examples, we use pattern matching to break apart our input argument, which is here a list of pairs. In the case of an empty list, the result is a pair of empty lists. Otherwise, we match the head of the list as a pair, giving us the elements `x` and `y` to place in the resulting lists. But where do the lists come from? We can get them by recursively calling `unzip` on the remaining list of pairs `xys`, giving us exactly the two lists we need. By using the `let .. in` construct, we construct those lists by prepending the `x` and `y` values respectively in each list.

This gives us the desired result:

```
> unzip [(1,'H'),(2,'e'),(3,'l')]
([1,2,3],"Hel")
```

#### Pattern matching List arguments

Now let's look at a few more functions from the standard library, that mixes list and non-list arguments. Here are a few examples:

```haskell
import Prelude hiding (take,drop,splitAt,(!!))
```

Starting with `take`, a function that takes a specified number of elements from a given list:

```
> take 5 "Hello World!"
"Hello"
```

Let's see how `take` is implemented:

```haskell
take :: Int -> [a] -> [a]
take n _        | n <= 0 = []
take _ []                = []
take n (x:xs)            = x : take (n-1) xs
```

The first case guars against the number of elements being 0 or less, in which case it returns an empty list. The second case checks if the list is empty, also resulting in an empty list. The third case recursively rebuilds the list until the `n` counter is greater than 0, or as long as `xs` contains elements:

```
> take 5 "Hi"
"Hi"
```

Another similar function is `drop`. It has a signature identical to `take`, and it does the opposite - it skips the specified number of elements, returning the remainder. Here is the implementation:

```haskell
drop :: Int -> [a] -> [a]
drop n xs  | n <= 0 = xs
drop _ []           = []
drop n (_:xs)       = drop (n-1) xs
```

Similarly to `take`, this function checks whether the number of elements requested is 0 or less, in which case it returns the list unchanged. The second case checks whether the list is empty, returning an empty list. Finally, it recursively processes the tail of the list until `n` is 0 or there are no more elements to process:

```
> drop 2 "Hello"
"llo"
```

The next function unifies both `take` and `drop`, and is called `splitAt`. It splits the list at a specified position, resulting in a pair of lists: the first containing all elements before the specified position, and the second containing the remainder. It is implemented by using both `take` and `drop`:

```haskell
splitAt :: Int -> [a] -> ([a],[a])
splitAt n xs = (take n xs, drop n xs)
```

Using recursion directly lists is not very common when programming in Haskell, because the standard library contains a big variety of functions that can be combined to provide desired results. Direct, explicit recursion is a lower-level mechanism, used mostly in libraries.

The last in this series of functions is an operator `!!` which allows getting an element from the list by index. Here's the definition:

```haskell
xs     !! n | n < 0 = error "Prelude.!!: negative index"
[]     !! _         = error "Prelude.!!: index too large"
(x:_)  !! 0         = x
(_:xs) !! n         = xs !! (n-1)
```

The first two cases protect against using an index value smaller than 0 and trying to use an empty list. The third case operates on an index value 0, in which case it extract the head from the list and returns it. The last case operates recursively in all other cases, dropping all elements until `n` has reached 0, in which case the desired element is the first in the remaining list unless the remainder is empty.

```
> "Hello World!" !! 6
'W'
```

### Higher-order functions on lists

In the previous module, we learned that a *higher-order function* is a function that accepts another function as one of its arguments. Let's explore a few such functions for lists.

#### Predicates

A *predicate* is a unary function that takes some value and produces a boolean result. Let's look at the `filter` function:

```haskell
import Prelude hiding (filter,takeWhile,dropWhile,span,break)
```

```haskell
filter :: (a -> Bool) -> [a] -> [a]
filter p [] = []
filter p (x:xs)
  | p x       = x : filter p xs
  | otherwise = filter p xs
```

The first argument to `filter` is a predicate and the second argument is a list. The `filter` function applies the predicate to all elements in the list, keeping those for which the predicate returns `True`:

```
> filter (> 3) [1,2,3,4,1,2,3,4]
[1,2,1,2]
```

The `filter` function works as follows: the predicate function is bound to the parameter `p` (we cannot use function types as patterns), and write two cases: in case the list is empty, the result is also an empty list. Otherwise, the list is recursively traversed, checking each element `x` whether it fulfills the predicate condition. If so, it is added to the result, otherwise, the function is applied recursively to the remainder, discarding the element.

The next function is `takeWhile`. It has the same signature as `filter`, and it works by keeping the values as long as they fulfill the predicate condition, otherwise, the function stops.

```haskell
takeWhile :: (a -> Bool) -> [a] -> [a]
takeWhile _ [] = []
takeWhile p (x:xs)
  | p x       = x : takeWhile p xs
  | otherwise = []
```

Similarly to `filter`, the `takeWhile` function recursively checks the elements using the predicate, and when the predicate returns `False`, the function returns an empty list, thus terminating the recursion, returning any results it has collected:

```
> takeWhile (< 3) [1,2,3,4,1,2,3,4]
[1,2]
```

A function that does the opposite is `dropWhile`:

```haskell
dropWhile :: (a -> Bool) -> [a] -> [a]
dropWhile _ [] = []
dropWhile p xs@(x:xs')
  | p x       = dropWhile p xs'
  | otherwise = xs
```

Using `dropWhile` discards elements as long as they fulfill the predicate, returning the remaining elements:

```
> dropWhile (< 3) [1,2,3,4,1,2,3,4]
[3,4,1,2,3,4]
```

The `dropWhile` function works similarly to `keepWhile`, but returns the remaining list in case of the predicate returning `False`. Here, it uses a syntactic construct `@` ( called *as-pattern*), allowing assigning the matched pattern on the right-hand side of `@` to a symbol on the left-hand side as an alias. In this case, the value `xs` and `(x:xs')` are identical, allowing accessing the entire, un-deconstructed list. This is more convenient when returning the list in the `otherwise` guard, instead of reconstructing it again from `(x:xs')`.

The next function is `span`, which has a slightly different signature:

```haskell
span :: (a -> Bool) -> [a] -> ([a],[a])
span p xs = (takeWhile p xs, dropWhile p xs)
```

The `span` function uses `takeWhile` and `dropWhile` to return a pair of lists, the first containing elements that fulfill the predicate, and the second containing the remainder:

```
> span (< 3) [1,2,3,4,1,2,3,4]
([1,2],[3,4,1,2,3,4])
```

The `break` has a similar signature, and it is used to break a list into a pair of lists at a condition specified by the predicate:

```
> break (> 3) [1,2,3,4,1,2,3,4]
([1,2,3],[4,1,2,3,4])
```

And it is defined as follows:

```haskell
break :: (a -> Bool) -> [a] -> ([a],[a])
break p = span (not . p)
```

It uses `span`, but negates the predicate using the function composition operator `.`. The `not` function has a type `Bool -> Bool`, composed with the predicate `a -> Bool` gives us exactly `a -> Bool`, which is passed to the `span` function.

#### Mapping over lists with `map`

Sometimes we need to process all elements in a container, applying a function to each element and storing the results in a new container. For such operations, there exists a function `map` that takes an arbitrary function and a list. It applies the function to every element in the list and produces another list having the same number of elements, but of a different type.

```haskell
import Prelude hiding (map,concat,concatMap)
```

First, let's see `map` in action:

```
> map (+10) [1,2,3,5]
[11,12,13,15]
```

Here, `map` takes a function `(+10)`, which is a function `Int -> Int`, and a list of numbers. It applies the function to all elements in the list, producing the result where every number was added 10 to it.

While the structure of the container remains the same (i.e. the same number of elements is produced), the type can be different:

```
> map length ["aa","bbb","ccccccc"]
[2,3,7]
```

Here, the function `length` is applied to all the string elements, counting the length of each string, producing a number as a result.

The `map` function is implemented as follows:

```haskell
map :: (a -> b) -> [a] -> [b]
map _ []     = []
map f (x:xs) = f x : map f xs
```

As we saw in the previous section, this function has two cases: in case of an empty list, the result is an empty list, otherwise, it will recursively apply the function `f` to every element, rebuilding the list with the new values.

Another useful function is `concat`, concatenating an arbitrary number of lists into a single list. It generalizes the `++` operator we've seen earlier to work on any number of lists:

```haskell
concat :: [[a]] -> [a]
concat []       = []
concat (xs:xss) = xs ++ concat xss 
```

Giving us the desired result (recall that a `String` in Haskell is an alias for `[Char]`):

```
> concat ["Hello"," ","World!"]
"Hello World!"
```

Finally, the `concatMap` function combines both `concat` and `map`:

```haskell
concatMap :: (a -> [b]) -> [a] -> [b]
concatMap f xs = concat (map f xs)
```

Or, even more concisely:

```haskell
concatMap f = concat . map f
```

The `concatMap` function applies the function `f` to all elements in `xs`. The result is a `[[b]]`, since the `f` function itself produces a list. This list of lists is passed to the `concat` function, flattening the lists into a single list.

Here's an example:

```
> map (\x -> [x,x,x]) "ABCD"
["AAA","BBB","CCC","DDD"]
> concatMap (\x -> [x,x,x]) "ABCD"
"AAABBBCCCDDD"
```

#### `map` usages

Let's look at some common usages of the `map` function:

```haskell
import Prelude hiding (and,or,all,any)
```

Let's define a few functions already present in the standard library:

```haskell
and, or :: [Bool] -> Bool

and []     = True
and (x:xs) = x && and xs

or []     = False
or (x:xs) = or || or xs
```

The `and` and `or` functions take a list of boolean values, returning the logical conjunction and disjunction, respectively.

Additional useful higher-order functions:

```haskell
all :: (a -> Bool) -> [a] -> Bool
all p = and . map p

any :: (a -> Bool) -> [a] -> Bool
any p = or . map p
```

The `all` function return `True` if all elements fulfill the predicate condition, while `any` returns `True` if at least one element fulfills the condition:

```
> all odd [1,3,43]
True
> all odd [1,3,43,2]
False
> any odd [1,3,43,2]
True
> any even [1,3,43]
False
```

They work by first converting the list of values to a list of booleans using `map`, followed by applying the `and` and `or` functions to the results, respectively.

As an exercise, let's reverse all the words in a sentence. Given a string `"Abc is not ABC"`, we'd like to reverse it, producing `"cbA si ton CBA"`. Here's how we can do this:

First, let's break the sentence to a list of words. In Haskell, there exist a standard library function `words` that does just that:

```
> words "Abc is not ABC"
["Abc","is","not","ABC"]
```

It splits the string by whitespace, producing a list containing each word.

The opposite operation is called `unwords`. Applying `unwords` to the result of `words` gives us back the original string:

```
> unwords (words "Abc is not ABC")
"Abc is not ABC"
```

We can write it more concisely using function composition:

```
> unwords . words $ "Abc is not ABC"
"Abc is not ABC"
```

Now we'd like to reverse each word, by applying the `reverse` function to every element using `map`. Since we get the list as the result of `words`, we can compose it like so:

```
> unwords . map reverse . words $ "Abc is not ABC"
"cbA si ton CBA"
```

Giving us the desired result `"cbA si ton CBA"`. Finally, we can extract this to a function:

```
> reverseWords = unwords . map reverse . words
> reverseWords "Abc is not ABC"
"cbA si ton CBA"
> :t reverseWords
reverseWords :: String -> String
```

#### The `zipWith` family

In the previous section, we've encountered the `zip` function, which takes two lists and creates a list of tuples, containing elements from both lists. Haskell has a generalized version of `zip`, called `zipWith`, where instead of creating a tuple, it specifies how the elements are combined.

```haskell
import Prelude hiding (zipWith,zipWith3)
```

Let's look at the definition of `zipWith`:

```haskell
zipWith :: (a -> b -> c) -> [a] -> [b] -> [c]
zipWith _ []     _      = []
zipWith _ _      []     = []
zipWith f (x:xs) (y:ys) = f x y : zipWith f xs ys
```

The `zipWith` function takes a function of two arguments `a -> b ->c` and two lists, `[a]` and `[b]`. If either of the lists is empty, the result is an empty list. Otherwise, it recursively applies the function to both head elements of the two lists, appending it to a list of the result list `[c]`.

Let's see a few examples:

```
> zipWith (+) [1,2] [3,4,5]
[4,6]
```

Here, the `(+)` function is first applied to the elements 1 and 3, producing 4, followed by 2 and 4, producing 6. The function stops when one of the lists ends.

The `zip` function is a special case of `zipWith` and we can express it with the `zipWith` function as follows:

```
> zipWith (,) [1,2] [3,4,5]
[(1,3),(2,4)]
```

The standard library also defines `zipWith3` which operates on three lists.

### Generating lists

#### Infinite lists

#### Functions producing infinite lists

#### Arithmetic sequences

#### List comprehension

### Right folds

#### Right fold definition

#### The `foldr` function

#### Evaluation order (???)

### Left folds

#### Left fold definition

#### Left fold implementation

#### Strict left fold

#### Right fold performance

#### Distributed folds (????)

### Related functions

#### Folds without an initial value

#### Left scan

#### Left scan performance

#### Right scan

#### Unfolding

#### The `Maybe` data type

#### Terminating (????) folds