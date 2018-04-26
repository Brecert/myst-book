# Chapter 8: Blocks and Anonymous Functions

Previously in this guide, we looked at functions and how they can be defined. Beyond those basics that we've already seen, Myst also supports blocks, anonymous functions, and function capturing.

Blocks and anonymous functions are both analagous to [function pointers](https://en.wikipedia.org/wiki/Function_pointer) from lower-level languages like C or Rust. The primary purpose of both is to allow users to dynamically inject code into other methods to either modify their behavior or define handlers to be called later.

Function capturing acts like a bridge between functions and blocks, allowing any function to be used as if it were a block.



## Blocks

Blocks are single-clause functions, defined as suffixes for Calls to other methods. They only exist within the scope of the Call, and will be destroyed as soon as the Call returns. Myst provides two different syntaxes for defining blocks, referred to as `do...end` blocks and brace blocks.

`do...end` blocks, as their name suggests, use the `do` and `end` keywords to delimit the block. Inside, the block can contain any set of expressions, just like a normal function:

```myst
5.times do
  IO.puts("hello")
end
```

The brace block syntax, on the other hand, uses curly braces for the delimiters:

```myst
5.times{ IO.puts("hello") }
```

In general, `do...end` blocks are used for blocks that span multiple lines, while brace blocks are used for single-line blocks.

When a Call also has other arguments, the block is given _after_ the closing parenthesis for the arguments:

```myst
object.method(1, 2, 3) do
  # do something
end
object.method(1, 2, 3){ }
```


### Parameters

Blocks can also define a list of parameters by specifying them between pipe characters after the opening delimiter:

```myst
[1, 2, 3].each do |element|
  IO.puts(element)
end
[1, 2, 3].reduce{ |acc, elem| acc += elem }
```

Since blocks are really just function clauses, the parameter structure is exactly the same, with the pipe characters around them being the only difference. Blocks can even have pattern matched and interpolated parameters:

```myst
object.method{ |a : Integer, <(a*2)>, *rest| :something }
```


### Closures

Both blocks and anonymous functions are implemented as [closures](https://en.wikipedia.org/wiki/Closure_(computer_programming)), meaning they store and have access to their _lexical environment_ (the scope where they are defined). This has a few implications, but a simple example is easily the best explanation:

```myst
sum = 0
[1, 2, 3].each do |element|
  sum += element
end
sum #=> 6
```

In this example, the block given to `each` creates a closure over its _environment_. Here, the environment is just the local `sum` variable defined on the first line, but will also include the value of `self` and any other variables defined in the containing scope.

Inside of the block, the code can access, modify, and even re-assign this `sum` variable, and the result will persist even after the block has finished running. This is how the `sum` variable gets the value of `6` at the end of our example.

However, any new variables created within the block will be limited to that block's own scope, and will not be available from outside:

```myst
[1, 2, 3].each do |e|
  sum ||= 0
  sum += e
end
sum #=> No variable or method `sum`
```


### Accepting block parameters

Like any other parameter, functions must explicitly show that they accept a block parameter. This is done by adding a parameter to the end of the parameter list, prefixed with an ampersand (`&`) to indicate that it is a block parameter.

The block parameter can be given any name, but most commonly it is left as `block`:

```myst
# This clause of `foo` accepts a block
def foo(a, b, &block)
end

# This clause does not
def foo(a, b); end
```

Block parameters will be matched just like every other parameter. Unlike Ruby or Crystal, there is no way to implicitly accept a block parameter in Myst.

Inside of the clause, the block parameter is accessible as a normal function, and can be called as such:

```myst
def apply(a, b, &block)
  block(a, b)
end

apply(1, 2){ |a, b| a + b } #=> 3
```

Block parameters can be called any number of times and with any set of arguments, so long as the block given by the Call accepts those parameters. Any mismatch will result in a match error when attempting to call the block.



## Anonymous Functions

Anonymous functions exist part way between blocks and regular functions. Like blocks, anonymous functions are closures over their lexical scope, but like functions (and unlike blocks), anonymous functions can define multiple clauses.

Unlike regular functions, anonymous functions do not have a name, and are _not_ added to the scope of `self`. Instead, anonymous functions are _local values_ that only exist within the scope where they are defined.

Anonymous functions are defined using a special block structure using the `fn` keyword and _stabs_ (`->`) to define clauses. Here's a simple example:

```myst
fn
  ->() { :no_arguments }
  ->(n : Integer) { :int_argument }
  ->(_) do
    a = 1
    b = 2
    a + b
  end
end
```

Here, each `->` indicates a new function clause, followed by the parameters for that clause given within the parentheses, just like normal functions. Finally, the body for the clause is given just like a block, either as a brace-block or within a `do...end`. The body can be given inline or over multiple lines.

Anonymous functions can also be written on one line, but this is generally not recommended (use a block instead):

```myst
fn ->() { :do_something } end
```

Since anonymous functions are just regular expressions that create a value, they can be used as the right side of an assignment. For example:

```myst
func = fn
  ->() { :something }
end
```

Once an anonymous function has a name, it can be invoked just like any other function. extending the above example:

```myst
func() #=> :something
```

The only additional requirement here is that parentheses _must always_ be given, even with no arguments. This avoids ambiguity between variable references and function calls. This also means that the anonymous function object can be passed around _without_ being called by omitting the parentheses:

```myst
func = fn
  ->() { :something }
end

other_reference = func # `func` will _not_ be called here.
other_reference() #=> :something
```



## Break and Next

Beyond `return`, Myst provides two distinct flow control keywords for use within blocks and anonymous functions: `break` and `next`.

`next` is exactly like `return`, but meant for use within blocks to avoid visual ambiguity between a return from a block and a return from the containing method:

```myst
sum  = 0
[1, 2, 3].each do |e|
  when e == 2
    next nil
  end

  sum += e
end
sum #=> 4
```

`break`, on the other hand, has some special semantics. Like `return` and `next`, it accepts an optional value, and will return from the block immediately. However, `break` will _also_ cause the _containing_ function to return immediately as well, using the value given to `break` as the return value. For example:

```myst
def foo(&block)
  block(1)
  :from_foo
end

foo{ break :from_block } #=> :from_block
```

`break` and `return` can also be used in anonymous functions in the same way:

```myst
func = fn
  ->(4) { break :broken }
  ->(n) { IO.puts(n) }
end

result = [1, 2, 3, 4, 5].each(&func)
IO.puts(result)
```

The output of the above would be:

```text
1
2
3
:broken
```



## Captures

In the last example above, you may have noticed that we used an anonymous function as the block parameter to `each`. This was done using the _function capturing syntax_.

Function capturing has two different uses, the first is as shown above, where the captured function is used as a block parameter, and the second is just capturing a function into a variable (much like how anonymous functions can be assigned).

Capturing a function looks exactly like how block parameters are defined in function clauses, using an ampersand (`&`) prefix to the function being captured:

```myst
sum = 0
def print(a)
  IO.puts(a)
end

[1, 2, 3].each(&print)
```

Functions can also be captured from within modules, types, and instances:

```myst
defmodule Foo
  def foo
    :hi_from_module
  end
end

deftype Bar
  def bar
    :hi_from_instance
  end
end

mod_func  = &Foo.foo
inst_func = &%Bar{}.bar
mod_func()  #=> :hi_from_module
inst_func() #=> :hi_from_instance
```

The most common use of function capturing, however, is inline capturing of anonymous functions. This style makes anonymous functions look a lot more like blocks given to calls directly:

```myst
found_2 = [1, 2, 3].each(&fn
  ->(2) { break true }
  ->(_) { false }
end)

found_2 #=> true
```

What we've seen here have been somewhat trivial examples, but hopefully it has shown the flexibility that blocks and anonymous functions provide, and how they can be used somewhat interchangeably to create simple, powerful, flexible expressions quickly and easily.
