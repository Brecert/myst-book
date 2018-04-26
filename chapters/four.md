# Chapter 4: Pattern Matching

Pattern matching is a powerful method of making assertions on values, and decomposing them into smaller parts to pluck out what is important.

## Match operator

Myst provides the "match" operator, `=:`, to perform pattern matching anywhere. When a match fails, an error is raised and execution is stopped to handle the error.

Most commonly, the match operator is used to decompose collections that are returned from methods:

```myst
[a, b, c] =: [1, 2, 3]
```


## Splat collectors

When pattern matching with lists, a common operation is plucking a single element out of the list. The normal way of doing this in most languages is to get the list, then access the first element separately:

```ruby
list = method_that_returns_a_list
element = list[0]
# Or, in one line:
element = method_that_returns_a_list[0]
```

Because list matches are exhaustive, doing this with a pattern match would require knowing how long the list would be, or doing the long hand method of getting the list, then getting the element separately.

Thankfully, Myst provides an easier way of plucking specific values out of lists (even if the length of the list is unknown), using what's called a "splat collector".

Similar to the splat _operator_ (`*`) covered earlier that expands a single list into multiple individual values, a splat _collector_ does the opposite; it collects multiple indivual values into a single list. When combined with list decomposition via pattern matching, getting any value (even more than one) out of a list is easy. For example, to get the first element of an arbitrary-length list as in the examples above, use a splat at the end of the list pattern:

```myst
[element, *_] =: method_that_returns_a_list
```

Using an Underscore for the splat just indicates that we don't intend to use the value that is collected by the match.

Splat collectors can appear at any point in a list pattern, but only one may be given. For example, to get the _last_ element of a list, simply put the splat collector first:

```myst
[*_, element] =: method_that_returns_a_list
```

Or, to get both the first and last element of the list, put it in the middle:

```myst
[first, *_, last] =: method_that_returns_a_list
```


Another use case of splat collectors is getting the remaining values in a list for use later on. In this case, just replace the Underscore used above with a normal variable. For example:

```myst
[head, *tail] =: method_that_returns_a_list
```

This is great for more functional-style programming, where lists are essentially treated as "cons cells" (like linked lists).


## Matching literal values

While using pattern matching for decomposition is useful, the most common use case is to assert that a _value_ matches a specific _pattern_ (hence the name). To do this, the pattern can provide literal values that the right hand side will be expected to match.

For example, combined with a splat collector, checking that a list starts with a `1` is easy:

```myst
[1, *_] =: some_list
```


## Matching types

Another form of pattern matching is checking the type of a value. Just like how literals can be used in patterns, specifying a type will check that the value at that position is of the given type.

For example, checking that the result of a method is an integer value can be done with:

```myst
Integer =: method_that_should_return_an_integer
```

Matching types may not seem useful for now, but later on, we'll see how type matching can be used to clean up function definitions and other conditional logic.
