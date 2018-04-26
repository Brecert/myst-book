# Chapter 5: Functions and Clauses

Functions in Myst are a little different than most other languages. In the basic case, things are fairly normal:

```myst
def add(a, b)
  a + b
end
```

Readers familiar with Ruby, Elixir, or other similar languages will likely recognize this syntax. The `def` keyword indicates that a function is being defined, `add` is the name of the function, `a` and `b` are the parameters of the function, and the body of the function is the expression `a + b`.

Myst has implicit return values just like Ruby, so the result of calling the function will be the result of `a + b`. For example:

```myst
add(1, 2) #=> 3
add(2, 4) #=> 6
```

Functions are useful for separating out logical components and creating building blocks to clean up code, avoid repetition, and better express more complex ideas.

Functions may also be called "methods". In practice, these terms are entirely interchangeable, where "method" is sometimes preferred for functions that are defined for instances of a type. This will be covered in a later chapter.


## Pattern matching parameters

Myst also allows functions to define patterns in place of parameters. In fact, all parameters are really just the left-hand side of a pattern match. Giving a pattern as a parameter creates an expectation that the argument given to the function matches that value. For example:

```myst
def add(0, n)
  n
end
```

This version of `add` expects that the first argument given to it will be `0`. If the first argument in a call to this function is _not_ 0, the function is not a match for the call, so it will not be called. In fact, an error will be raised saying that there is no match for the function with the given arguments.

Here are some examples of calls to this function:

```myst
add(0, 3)   #=> 3
add(0, 10)  #=> 10
add(1, 2)   #=> raises a `FunctionMatchFailure`.
```

The first two calls succeed, because the expectation that the first argument is `0` is met, so the definition matches. The third call gives `1` as the first argument, which does _not_ match `0`, so the call fails.

A more complex example is extracting values from a Map argument:

```myst
def foo(thing, {default: default, type: type})
  # `default` and `type` are now available as variables.
end
```


## Multi-clause functions

A consequence of pattern matching parameters in function definitions is that a single function definition will not necessarily work for all calls. This was clearly shown with the `add(1, 2)` example above, and is not an accident.

In Myst, it's possible to define multiple different "versions" of a function with different parameters. Each "version" is called a _clause_, and all the clauses under a single name are collected into a full _function object_ or _functor_ (often just referred to as a _function_ for simplicity).

The clauses of a function are stored in the order that they are defined, so when a call to a function is encountered, the language will run through the clauses in order, attempting to match all of the parameters. The first clause that matches all of the parameters will be used for the call. If no clauses match, an error will be raised, as shown above.

With this new information, a more complete `add` function from the above examples could be written with three clauses.

```myst
def add(0, n)
  :first_clause
end

def add(n, 0)
  :second_clause
end

def add(a, b)
  :third_clause
end
```

Now, `add` should work with any input:

```myst
add(0, 1) #=> :first_clause
add(1, 1) #=> :third_clause
add(1, 0) #=> :second_clause
```

Note that because clauses are checked in order, the definition order is important. Consider the case where the third clause above is given first:

```myst
def add(a, b)
  :first_clause
end

def add(0, n)
  :second_clause
end

def add(n, 0)
  :third_clause
end

add(0, 1) #=> :first_clause
add(1, 1) #=> :first_clause
add(1, 0) #=> :first_clause
```

Here, the first clause is always used, because `a` and `b` don't set any expectations on the arguments.

Another common use case for multiple clauses is working with different _arities_ (the number of parameters for the clause). As described above, Myst attempts to match _all_ of the parameters. If a clause has more parameters than there are arguments for the Call, or vice versa, it will fail to match and the next clause will be attempted.


## Splat collection parameters

Similar to the splat collectors described in the pattern matching section earlier, parameters can also use splat collectors to collect extra arguments. The examples of getting the first and last elements of a list of arguments can be written in methods as follows:

```myst
def head(first, *_)
  first
end

def tail(*_, last)
  last
end

def ht(first, *_, last)
  [first, last]
end

head(1, 2, 3) #=> 1
tail(1, 2, 3) #=> 3
ht(1, 2, 3)   #=> [1, 3]
```

The thing to note here is that the arguments are not given as a single list argument, but as individual arguments.


## Type restrictions

Another way of defining a pattern for a parameter is to give an expected type, rather than an exact value. For example, to match any argument that is an integer, just use `Integer` as the parameter:

```myst
def add(Integer, Integer)
  :matched
end

add(1, 2) #=> matched
add("hi", "") # does not match
```


## Complex parameters

In the above examples, the parameters that define patterns or type restrictions don't capture the argument that is passed in, since the pattern is taking the place of the variable name.

To address this, Myst allows complex parameter definitions that can include patterns, a variable name, and a type restriction, all for the same parameter. A full example might look something like this:

```myst
def foo([a, *_] =: args : List)
  # do some work
end
```

In this example, `[a, *_]` is the pattern for the parameter, followed by the match operator, then a variable name (in this case, `args`), and then a colon and a type name.

There are a few alternatives to this full syntax when all of the components are not needed. At a minimum, one of the components must be given. For example, a parameter that only has a pattern and a type can omit the `=: name`:

```myst
def foo([a, *_] : List)
  # ...
end
```

With the introduction of types and modules, this syntax will become even more powerful, but will be covered later on, after those features have been covered independently.