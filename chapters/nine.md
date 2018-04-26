# Chapter 9: Exception Handling

Exception handling in Myst is really a more complex form of `break` and `next`. Exceptions are generally used to immediately halt execution of a function and panic upwards until an appropriate handler is found.

Myst adopts the `raise` and `rescue` keyword terminology from its Ruby ancestry. While the term "throw" may sometimes be used interchangeably with "raise", Myst does _not_ implement the `throw` and `catch` keywords that Ruby does, as the same behavior can easily be implemented with `raise`/`rescue`.


## Raise

Raising an exception in Myst is done using the `raise` keyword. `raise` expects to be given a value as an argument, but this value can be _any_ value, even `nil`:

```myst
raise "woops"
raise 1024 - 512
raise %Foo{}
raise nil
```

When ever a `raise` is encountered, execution of the current function will immediately stop (after the value has been created), and Myst will backtrack up the callstack until an appropriate handler is found (we'll see what this looks like in a bit).

If no matching handler for the exception is found, the program will exit with a non-zero exit status and a message of the error with the callstack that caused it.

While Myst allows any value to be raised, it can be useful to use common, default exception types to simplify how they are handled.


## Rescue

Once an exception has been raised, the only way to recover and continue execution of the program is with a `rescue` clauses.

`rescue` clauses are most commonly defined as suffixes for method definitions. The most basic example is a `rescue` with no parameter:

```myst
def bar
  raise "woops"
end

def foo
  bar
  :finished_normally
rescue
  :rescued
end

foo #=> :rescued
```

Notice that the return value of calling `foo` is `:rescued`, _not_ `:finished_normally`. Because `bar` raised an exception, the language immediately started panicking. This panic was stopped by the `rescue` on `foo`, meaning `:finished_normally` was never encountered in the main body of `foo`.

`rescue` clauses may also provide a pattern-matched parameter to check against the exception being raised:

```myst
def baz
  raise "woops"
end

def bar
  baz
rescue n : Integer
  :rescued_integer
end

def foo
  bar
rescue e
  :rescued_anything
end

foo #=> :rescued_anything
```

In this case, `baz` raised the String value `"woops"`. While `bar` defines a rescue clause, its parameter specifies an Integer value, which does _not_ match the String that has been raised, so panicking continues.

`foo`'s rescue clause simply defines a name for the exception, which will always match, so the exception is caught and the clause is evaluated, returning `rescued_anything`.

Just like a `when` chain, multiple `rescue`s can be given on the same method to match against different exceptions in the same place:

```myst
def foo
  raise "bar"
rescue Integer
  # do something...
rescue "bar"
  :rescued_bar
rescue e
  # do something else...
end

foo #=> :rescued_bar
```

The parameter for a rescue clause is just like a normal function parameter. Beyond the type matching shown above, this means that the parameter can be matched against literal values, destructurings, or even value interpolations! Myst's Spec library makes good use of this to define an `expect_raises` assertion:

```myst
def expect_raises(expected_error, &block)
  block()
  raise %AssertionFailure{@name, expected_error, "no error"}
rescue <expected_error>
  # If the raised error matches what was expected, the assertion passes.
rescue received_error
  raise %AssertionFailure{@name, expected_error, received_error}
end
```

Here, `expect_raises` calls `block`, then defines two exception handlers. The first dynamically interpolates the `expected_error` as the parameter of the rescue, which will only succeed if `block` raises a matching error, meaning the assertion passes.

Otherwise, the second handler matches any other raised exception, and raises a _new_ failure that the received exception did not match what was actually raised, causing the assertion to fail.


## Ensure

Sometimes raising an error in a block of code could leave a program in a bad or currupted state. Leaving a file open, not calling a callback function, etc. These are all things that could cause a successful recovery of a program with `rescue` to actually cause further failure. To help address this, Myst provides `ensure`.

`ensure` clauses come at the end of a `rescue` chain (or on their own), and will _always_ be executed, even while panicking up the callstack during a `raise`; _even when the exception has not been rescued_.

Here's a trivial example that shows the semantics of `ensure`:

```myst
@did_ensure = false
def foo
  raise "woops"
  :finished
rescue Integer
  :rescued
ensure
  @did_ensure = true
  :ensured
end

foo #=> :rescued
@did_ensure #=> true
```

So here's an interesting caveat. We clearly hit the `ensure` clause, because `@did_ensure` got set to true. But, the return value of `foo` was `:rescued`. Why? Because `ensure` _cannot_ affect the return value of a function.

More than anything, this is for logical simplicity when dealing with no errors. Here's an even simpler example:

```myst
def line_count
  f = File.open("test.txt")
  f.lines.size
ensure
  f.close
end
```

Here, the use of the `ensure` block is just to guarantee that the file `f` gets closed properly. But, we want the return value of the function to be the number of lines in the file.

If `ensure` _did_ change the return value, we'd have to save the line count into a local variable, then remember to add that variable as the return value after `f.close` in the `ensure` clause. Even if we used `return f.lines.size`, the same problem would occur, since `ensure` will _always_ run after a function completes.

So, for simplicity, `ensure` is guaranteed to _not_ affect the return value of a function.


## Exception handling on non-methods

Beyond exception handling on method definitions, Myst also allows exception handlers to be defined on blocks and anonymous function clauses using the `do...end` syntax. Since blocks and anonymous functions are semantically equivalent to normal functions, exception handlers work exactly the same way:

```myst
block_result = [1, 2, 3].each do |e|
  raise "woops"
  e
rescue
  :rescued
end

result #=> :rescued

func = fn
  ->(2) do
    raise "woops"
  rescue
    :rescued
  end
end

func(2) #=> :rescued
```

Note that exception handlers are _not_ allowed with the brace-block syntax. The result is too visually jarring and is inconsistent with keyword blocks always terminating with an `end` keyword.