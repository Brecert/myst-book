
# Chapter 3: Flow Control

Flow control is the act of controlling the execution of code based on some set of conditions. Myst provides two native methods of flow control - conditionals and loops.



## Conditionals

Conditional expressions indicate that the code in the body of the expression should only be evaluated if the given condition is met. Conditionals can also provide alternative bodies and conditions to evaluate if the original condition is not met.


### When

Unlike most popular languages, Myst does not have an `if` expression. Instead the same concept is expressed using the keyword `when`. For example, the Ruby code:

```ruby
if 1 < 2
  # ...
end
```

would be written in Myst as:

```myst
when 1 < 2
  # ...
end
```

Whether the body of a `when` expression is evaluated depends on the truthiness of the condition. As covered previously, only `nil` and `false` are considered falsey, and so any other value would result in the condition being satisfied, thus evaluating the body.

If a `when` expression's condition is satisfied, the result of the expression is the value of the last expression in the body. If it is not, the result will be `nil`.

In languages with `if` expressions, there is often also a mechanism for providing an alternative to execute when the condition is not met. Most often, this uses the `else` keyword. Myst follows suit, so the following will return `"condition was not met"`:

```myst
when 1 > 2
  "condition was met"
else
  "condition was not met"
end
```

In addition to this, most languages also provide a way to define other conditions for the alternatives. For example, Ruby has `elsif`:

```ruby
if 1 > 2
  "1 is more than 2"
elsif 1 == 2
  "1 is equal to 2"
else
  "no condition was met"
end
```

In Myst, however, multiple conditions can be chained together simply by using the `when` keyword again. So, in Myst, the above would be written as:

```myst
when 1 > 2
  "1 is more than 2"
when 1 == 2
  "1 is equal to 2"
else
  "no condition was met"
end
```

This is the primary motivation for using `when` instead of `if`. By re-using the same keyword for all conditional expressions, even when defining alternatives for another condition, the code is kept more visually-aligned.

All of the conditions being evaluated start in the same column, meaning the code can be scanned more quickly, and avoids having a jagged pattern, as in the Ruby version above.

The condition for a `when` expression can be any other expression. For example, a common usage will have query method calls as the condition:

```myst
when some_string.empty?
  # ...
end
```

For clarity, the examples in the rest of this chapter will continue to use comparison operators, but anywhere a condition is given, know that it could also be a method call, standalone variable or literal, or any other valid expression.


### Unless

Myst also provides an `unless` keyword to define an "inverse condition". In other words, where a `when` expression would evaluate its body when the condition is truthy, an `unless` expression evaluates its body when the condition is falsey.

`unless` expressions can stand on their own, or be mixed into `when` chains to more cleanly define behavior:

```myst
unless true == false
  "true was not equal to false"
end
```


### Nesting

Because of the chaining behavior of conditional expressions, it is not possible to directly nest a conditional within another conditional. For example, this code:

```myst
when x > 2
  when x == 3
    # ...
  else
    # ...
  end
else
  # ...
end
```

is not valid. In fact, it will actually cause a parse error. Despite what the indentation suggests, the above is actually parsed as a single when chain with two conditionals (`x > 2` and `x == 3`) and an `else`. So, when the parser sees the first `end`, the chain is ended, but then the parser sees an `else` that it doesn't expect.

This design is intentional. Directly nesting conditionals naturally leads to high complexity and can quickly become confusing. Forcing all conditional expressions to appear in a flat chain makes the path to complexity much less natural, and pushes for better code refactoring into smaller, simpler components.

However, if a nested conditional is _truly_ necessary, it is possible to use parentheses around the inner conditional to avoid ambiguity. The above example could then be written as:

```myst
when x > 2
  (when x == 3
    # ...
  else
    # ...
  end)
else
  # ...
end
```

This is not recommended, and is really just a side effect of how the language is parsed. A more proper refactoring of this code would look like this:

```myst
when x == 3
  # ...
when x > 2
  # ...
else
  # ...
end
```



## Loops

Loops execute blocks of code repeatedly based on a conditional value that is evaluated before each iteration.

Myst only provides two unbounded looping constructs, `while` and `until`. Notably, there are no _bounded-iteration constructs_ (e.g., a `for` loop). The general use case of a `for` loop in most languages is to iterate elements in a collection, but the general preference in Myst is to use `.each` for these iterations.

We'll see some examples of bounded iteration later on in the Modules section.

### While

A `while` expression is used to execute a block of code repeatedly until the condition is not met. For example:

```myst
n = 0
while n < 10
  # do some work
  n += 1
end
```

The above code would execute 10 times, after which the condition `n < 10` would be false, and looping would stop.

Unlike conditional expressions, loop expressions cannot be chained or define alternatives. Each expression is considered independent of the others, which also means that loop expressions can be nested without any extra syntax:

```myst
a = 1
while a < 10
  b = 1
  while b < 10
    # this will execute 100 times.
    b += 1
  end

  # this will execute 10 times.
  a += 1
end
```


### Until

Similar to how `unless` is the inverse of `when`, `until` is the inverse of `while`. Its body will be executed repeatedly until the condition _is_ met. The first example of `while` could be re-written with `until` like so:

```myst
n = 0
until n >= 10
  # do some work
  n += 1
end
```

`until` (and `unless`) are provided as a convenience for more clearly showing the intent of a flow control expression. As such, they should be used sparingly, and only when the expression reads more easily when used.