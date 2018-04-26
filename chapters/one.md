# Chapter 1: Values and Variables

Myst is a strong, dynamically typed language. At a high level, "strong" means that every value has a specified type, and will retain that type unless explicitly told to change, and "dynamic" means that variables can hold any type of value at any time.

The exact meanings of "strong" and "dynamic" aren't entirely universal - people have varying ideas on the minor details - and can get [surprisingly complex](https://en.wikipedia.org/wiki/Type_system#Type_checking) depending on how exact the definition is.

However, as with many other concepts, Myst implements a more pragmatic approach. Later on, we'll see that Myst provides a few different ways to restrict the types that variables can accept. For now though, "strong" and "dynamic" are sufficient descriptions of Myst's type system.


## Value Literals

Myst has a number of [literal representations](https://en.wikipedia.org/wiki/Literal_(computer_programming)) for values. These are the most basic way of creating new values in a program. Some examples of literals include:

```myst
nil           # Nil (`null` in some languages)
false         # Boolean
true          # Boolean
1             # Integer
1.0           # Float
"hello"       # String
:hello        # Symbol
[1, 2, 3]     # List
{a: 1, b: 2}  # Map
```

`nil`, `false`, and `true` are all _constant literals_. That is, they always refer to the same object, and the same keyword is always used to refer to them. All other literals are mutable, in the sense that the exact representation can vary.

### Numerics

Numerics are values that represent numbers. Myst supports Integers and Floats as numeric values.

An Integer is any whole number, such as `2`, `100`, or `987654321`. Integer literals in Myst can also contain underscores to help separate groups of digits. This is particularly useful for large numbers. For example, compare these two representations of 1 billion:

```myst
1000000000
1_000_000_000
```

In the version with underscores, it's clear to see that there are 3 groups of `000`s, allowing someone reading the code to quickly see that the value is 1 billion.

Floats are real numbers that may contain decimal values, such as `1.0`, `2.345`, or `123.456789`. Notice that `1.0` is a whole number, and thus _technically_ an integer, but the representation as `1.0` indicates to Myst that the value is a Float. This relates to the strong typing mentioned above, where operations like division may behave differently based on the type of the operands.

As with Integers, underscores are allowed in Float literals, both before and after the decimal point:

```myst
123_456.789_0123
```

There is no restriction on how many underscores can be given in a literal, or that they have to form groups of 3. The following are all valid numeric literals, though they are generally avoided for consistency:

```myst
1_
100_00
255_._255
1_2_3_4
9_____9
```

### Strings

A String is any series of zero or more characters contained between two double quote (`"`) characters. The following are all valid strings:

```myst
"myst"
""
"hello, world!"
"1 + 2 == 3"
```

All whitespace within a string is preserved, meaning Strings can span multiple lines:

```myst
"hello,
world"
```

Alternatively, this could be condensed using an _escape sequence_ for the newline character:

```myst
"hello,\nworld"
```

Other escape sequences include tab characters (`\t`), null terminators (`\0`), and a double quote within the string (`\"`). If a String needs to contain the `\` character itself, it must also be escaped to avoid ambiguity:

```myst
"hi \o"  #=> The `\` will escape the `o`.
"hi \\o" #=> The String will contain a `\` character.
```

### Symbols

Symbols are similar to Strings, but with a different representation within the language that we'll cover soon.

A Symbol literal is a _name_ or _identifier_ prefixed with a colon. The following would all be interpreted as Symbols in Myst:

```myst
:hello
:two_words
:true
:false
:nil
:__with_some_underscores___
:numbers123
```

Additionally, Symbols can be created from a String literal the same way. This is useful for creating symbols with spaces in the name or with other special characters:

```myst
:" "
:"hello, world"
:"+"
:" /afs% 1234"
```

Now, the way that Symbols are different than Strings is how they are stored within the language. Each Symbol is represented as an Integer. More importantly, every Symbol with the same name is represented by the _same_ Integer. So, a Symbol, `:name`, may be represented by the Integer `153`, then another Symbol, `:name`, would also be represented by `153`.

This makes comparisons with Symbols extremely efficient (just comparing two integers, rather than each character in the name), which makes them great for cases where the name of the Symbol isn't necessarily important, but performance matters (such as Maps, as seen below).


### Collections

Apart from the individual literal types, Myst also has literal representations of two collection types: Lists and Maps. These are very common in Myst code, so it's important that they be easily and clearly represented.

A List is an ordered collection of values. Any combination of values can be added to a List. List literals are surrounded by square braces (`[` and `]`), with each entry separated by a comma (and optionally some whitespace). All of the following are valid List literals:

```myst
[]
[1]
[1, 2, 3]
[nil, 1.0, "hello"]
[
  1, 2,
  true, false,
  "hi", :hello
]
```

Lists can also be nested within each other to create more complex structures:

```myst
[
  [true, 0],
  [false, 1, 2],
  [
    ["hi", []],
    [:hello, nil]
  ]
]
```

We'll see some more ways that Lists are useful in Myst later on.

A Map, on the other hand, is a collection of key-value pairs. In Myst, the order of the entries is also preserved. By default, all Map keys will be Symbols, but the value can be of any type.

A Map literal is created using curly braces (`{` and `}`), with comma-separated entries, similar to Lists. Each _entry_ of a Map is written as a key name followed by a colon, followed by the value for that key. All of the following are valid Map literals:

```myst
{}
{a: 1}
{a: 1, b: 2}
{true: true, false: false}
{
  number1: 1234,
  __thing: {
    string: "hi",
    symbol: :hi
  }
}
```

Notice that the Symbol keys do not _start_ with a colon, because the colon is instead placed _after_ the name.

Generally, Maps do _not_ put a space between the braces and the entries of the Map, but _does_ include a space between each entry. This is to avoid ambiguities with other braced constructs that we'll see later on.

It's possible to use other types of values as keys for Maps, which we'll see later on, but the preference is to always use Symbols. Earlier, we saw how Symbols can be compared efficiently as Integers, which makes their use as Map keys very effective. Using other types will generally slow down performance.

A final note on collections: nesting is not limited to the same type. It's just as easy to create Lists of Maps or Maps of Lists. For example:

```myst
[
  {a: 1, b: 2},
  {a: 3, b: 4}
]
```



## Variables

Variables are names that store values. For the most part, any value can be assigned to any variable at any time.

All variable names in Myst are _identifiers_. An identifier is any series of letters, numbers, and/or underscores, with the restriction that the first character can not be a number. The casing and structure of the name determines it's _kind_.

There are three main kinds of variables in Myst - Locals, Underscores, and Constants - each with a specific purpose and use case.

### Locals

Local variables start with a lowercase alphabetic character and are generally written in `snake_case`. Locals can be freely assigned and referenced any number of times, and are generally used to hold temporary data.

For example, when doing a complex calculation, intermediate results are generally held in local variables to make the final calculation more clear.

Local variables are tied to the _lexical scope_ that they are created in. In other words, variables created within a block of code will be destroyed once the block of code is finished running.

All of the following are examples of valid local variable names:

```myst
x
a
something_longer
number1
num_456
```

### Underscores

Underscores are local variables that start with an ASCII underscore character, `_`.

Just like locals, underscore variables can be assigned to and referenced any number of times. However, referencing an underscore variable will emit a warning.

Generally, underscores are used to indicate that a variable is known to exist, but should not be used in the code. We'll see examples of how this is helpful later on, namely with function parameters and pattern matching.

In some languages, there is [a pattern](http://po-ru.com/diary/rubys-magic-underscore/) of using a single underscore character to indicate that a value is unwanted. Some languages even treat the single underscore specially.

Other languages, Myst included, treat all underscore variables equally. As such, it's often useful to provide a more descriptive name for the underscore variable in case it may become useful in the future.

For example, this code is entirely valid:

```myst
[first_name, _, _, _] = some_method
```

We'll cover exactly what this means later on, but it's obvious that there are three values we don't _currently_ care about. The issue is that without looking at the definition of `some_method`, it's hard to tell _what_ those values are, and if they could be useful in the code that follows it.

A better version of the above would be something like this:

```myst
[first, _last, _age, _city] = some_method
```

Here, we can see what the other three values are, but they are still marked as undercsore variables, meaning we don't currently have a use for them.

### Constants

Constants start with uppercase alphabetic characters, and are written either in `UpperCamelCase`, or `SCREAMING_CASE`. Constants also have different semantics than underscores and local variables.

A constant variable can only be assigned once. Assigning to a constant more than once will raise an error. As such, constants are generally used to hold values that are not expected to change very often. For example, the seasons of a year:

```myst
SEASONS = ["spring", "summer", "fall", "winter"]
```

Note that while the constant _itself_ cannot be reassigned, its value may still be changed by other operations. For example, a new season could be added just by adding to the end of the list.

```myst
SEASONS[4] = "some new season"
```

We'll cover more about what this code means later on, but the important part here is that constants can still be modified, even though they can only be assigned once.

The other primary use of constants is to represent types and modules. Earlier in this section, you may have noticed that we always refer to the type of a value with a capital letter at the start. This is to match the name of the type within Myst. Types like `Integer`, `Float`, and `List` are all constants within Myst.

Types and modules will be covered in more detail later on, but know for now that a variable written in `UpperCamelCase` is generally referring to either a type or a module.


## Assignments

Assignments are the primary way of creating new variables. All assignments consist of a _target_ (the left side), an assignment operator, and a _value_ (the right side).

### Simple assignments

Most assignments are technically _simple assignments_. Myst also has two other kinds of assignments ("match" and "operational") with different semantics.

A simple assignment is made with a variable name (called the _target_) followed by an equals sign (`=`) and then some value to be assigned. We've seen this a few times already, but here's a more complete example:

```myst
x = 1 #=> 1
x     #=> 1
x = 2 #=> 2
x     #=> 2
```

The comment on each line shows what the value of `x` is after the line has finished running.

### Operational assignments

It's fairly common for an assignment to include the target in the value being assigned. For example, incrementing a variable by one might look somtehing like this:

```myst
x = x + 1
```

Especially with longer variable names, the repetition of the name creates unnecessary noise and often makes the assignment harder to understand at a glance.

To improve this, Myst provides _operational assignments_, which include the operation being performed as part of the assignment operator. The above example could be written as an operational assignment like this:

```myst
x += 1
```

The full semantics of operational assignments will be covered in the Basic Operations section.

### Match assignments

Match assignments use the _match operator_ (`=:`). Match assignments are unique in that an assignment will not necessarily occur. Instead, the "assignment" is an expectation about the _value_ of the assignment, rather than the _target_.

The full semantics of match assignments will be covered later on in the Pattern Matching section.


## Interpolations

One of the truly unique features of Myst as a language is the ability to interpolate (or _inject_) any value into another other section of code. An interpolation is creating using angle braces (`<` and `>`) with some expression between them.

In the Maps section, we saw that Map keys are almost always Symbols, but that it's possible to use other values for keys as well. One way to do this is to use interpolations:

```myst
{
  <true>:       :true,
  <"hello">:    :string,
  <[1, 2, 3]>:  :list
}
```

With this syntax, it's possible to use _any_ value as the key for a Map entry, even Lists or other Maps. But, as mentioned before, using keys other than Symbols will hinder performance.

Interpolations also allow operations within the interpolation for even more powerful behavior. For example, variables can be referenced within an interpolation, and calls to other functions can be made:

```myst
x = 1
list = [1, 2, 3]
{
  <x>: :one,
  <list[2]> :three
}
```

In fact, _any_ expression can be used inside of an interpolation, but will require additional parentheses to avoid ambiguity with the less-than and greater-than operations:

```myst
x = 15
{
  <(1 + 2)>: :three,
  <(x / 3)>: :five
}
```

All of the examples so far have shown interpolations used for creating keys in a Map. While interpolations are allowed anywhere in Myst code, they generally don't have any additional effects that make them useful. For example, these two lines are equivalent:

```myst
x = <(1 + 2)>
x = 1 + 2
```

Another common use for interpolations is inside of Strings. In addition to evaluating an expression in-place, String interpolations will also convert the result into a String representation before inserting it into the String:

```myst
name = "John Smith"
"hello, <(name)>!" #=> "hello, John Smith!"
```

Note that string interpolations _always_ require the additional parentheses around the expression being interpolated.

String interpolations are expanded by the language into separate components for the literal String parts and the expressions, then summed together to create the result. The exact expansion would look like this:

```myst
# Given a string like this:
"1 + 2 = <(1 + 2)>!"
# The language would interpret it as:
"1 + 2 = " + (1 + 2).to_s + "!"
```

This expansion is _semantically_ the same as what the language ends up doing, but would perform slower than the interpolated version since the language can make some optimizations when the values are interpolated.

The last main use case of interpolations is in pattern matching, which we'll see later on in the Pattern Matching section.