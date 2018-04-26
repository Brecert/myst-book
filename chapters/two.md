# Chapter 2: Basic Operations

With the knowledge of values and how they are managed, let's look at some ways that they can be manipulated.

Almost all of the basic operators are actually defined as methods on the types of objects. For example, the expression `1 + 2` calls the `+` method on the value `1`, with `2` as an argument. This method-based approach means that these operators can be overridden on any type at any time.

The examples in this section will almost exclusively use literal values to clearly show the types involved in the operations, but all of the operators can be used with variables or sometimes even types or modules instead with the same effects.

## Mathematic operations

Most primitive values define the basic arithmetic operations, `+`, `-`, `*`, `/`, and `%`. Some examples:

```myst
1 + 1       #=> 2
2.6 + 3.8   #=> 6.4
10 * 10     #=> 100
120 % 100   #=> 20
"h" + "i"   #=> "hi"
```

Collections also implement some operations like `+`:

```myst
[1, 2] + [3, 4] #=> [1, 2, 3, 4]
{a: 1} + {b: 2} #=> {a: 1, b: 2}
```

For Maps, keys are unique, so adding two Maps that contain the same key will give the key in the result the value of the second Map. For example:

```myst
{a: 1, b: 2} + {a: 2, c: 3}
#=> {a: 2, b: 2, c: 3}
```

This can be useful for quickly applying default values to a Map, but may also lead to some unexpected behavior if it is not understood beforehand.

Most value types define a number of other mathematic operations, such as `String#*` and `List#-`. There are too many to cover comprehensively in this guide, so be sure to check the documentation for a more complete look at the operations that are available.



## Collection access

To access individual values within a collection, Myst provides _access notation_ via square braces (`[` and `]`) with an index argument between them:

```myst
list = [1, 2, 3]
list[0] #=> 1
map = {a: 1, b: 2}
map[:b] #=> 2
```

Lists use zero-based indexing, meaning the first element of a List is at position `0`, and the last element of the List is at position `n-1`, where `n` is the number of elements in the List.

In a List, the index argument is generally only allowed to be an Integer. For Maps, the index can be any type, but will generally be a Symbol, following the preference that Map keys should use Symbols.

In any case, if the element at the given index does not exist (e.g., the key is not in the Map, of the index is past the end of a List), the access will return `nil`.

It is also possible to directly assign an individual element of a collection using _access-assignment notation_. This looks just like a collection access used as the target of an assignment. For example:

```myst
list = [1, 2, 3]
list[0] = 4
list #=> [4, 2, 3]
```

Notice that this directly modifies the collection itself, but is not considered an assignment _to_ the collection.



## Logical operations and truthiness

Myst provides two native logical operations, "or" (`||`) and "and" (`&&`). These operate based on the truthiness of the first operand. The result of a logical operation is the value that determined the action taken by the operation - no conversions to Booleans or other operations are done to the value.

All values are considered truthy unless they are either `nil` or `false`. This means that, unlike some languages such as C or JavaScript, `0`, `[]`, `{}`, and `""` are _all_ consider truthy.

Logical operations in Myst are also short-circuiting, meaning that the right side operand will only be evaluated if the left side operand does not satisfy the operation. For an Or operation, that is when the left side operand is falsey, or truthy for an And operation.

Below are some examples of logical operations and the results they would yield.

```myst
false || false  #=> false
false || true   #=> true
true  || false  #=> true
1     || 2      #=> 1
nil   || 2      #=> 2
false && false  #=> false
false && true   #=> false
true  && false  #=> false
true  && true   #=> true
nil   && 2      #=> nil
[]    && true   #=> false
```

It's worth noting that since there are two falsey values, the result of a failed logical operation (one that is not satisfied) can be _either_ `nil` or `false`, but no other value. For example:

```myst
[] && false   #=> false
false || nil  #=> nil
```

Unlike most other operators in Myst, `||` and `&&` are native to the language, and cannot be overridden. Operators that _are_ overridable have a receiver (the left) and an argument (the right), but the logical operators are defined by the two operands equally. Because of that, it does not make sense for logical operations to be overridable.



## Comparison operations

Myst provides all of the common comparison operators: `<`, `<=`, `==`, `!=`, `>=`, and `>`. These operations _should_ always return a boolean value, but can be changed by overridden behavior.

All types define `==` and `!=`. These are also the only comparison operations that booleans and `nil` define by default.

```myst
true == false #=> false
true != false #=> true
nil  == nil    #=> true
nil  == false  #=> false
```

An important thing to note in the example above is that `nil` will only ever be equal to `nil`. The fact that `nil` and `false` are both falsey does not apply to equality operations.

Equality operations are always based on values rather than object ids, as in Java and other similar languages. That means there should never be a concern about using `.equals` or `.eql` or other special methods instead of `==` for things like Strings.

```myst
"hello" == "hello" #=> true
```

The remainder of the comparison operators are implemented selectively for each type.  For numeric types, the default implementations for all of these operators all perform their mathematic definition:

```myst
1  <  2       #=> true
10 == 20      #=> false
5  >= 5.4     #=> false
```

Operations between a Float and an Integer operand on either side will also work as expected.

```myst
1.0 == 1      #=> true
1   == 1.1    #=> false
```

Collection types also implement `<` and `<=` to check whether the left side is a subset of the right side. Specifically, `<` is a _proper subset_ check, meaning the right side must be a larger collection than the left side, while `<=` also allows the collections to be equal. The order of elements in the collection is not important for either of these operations.

```myst
[1, 2] <  [1, 2]        #=> false
[2, 1] <= [1, 2]        #=> true
{b: 2} <  {a: 1, b: 2}  #=> true
```



## Unary operations

Unary operations are operations with a single operand (the receiver), and are always written as prefixes for the operand. The three unary operations supported by Myst are `!`, `-`, and `*`, any combination of which can be chained together.

`!` performs a logical "not" on the operand. Truthy values become `false`, while falsey values become `true`. The result is always a Boolean. For example:

```myst
!true   #=> false
!false  #=> true
!nil    #=> true
![]     #=> false
!!100   #=> true
```

The last line above is an example of a "double not". This is a common pattern for quickly getting the truthiness of a value as a Boolean.

The unary version of `-` is a negation. For numeric types, this is essentially equivalent to the binary operation with `0` as the receiver (i.e., `-2` is the same as `0 - 2`).

```myst
a = 100
-a        #=> -100
--a       #=> 100
```

Again, `-` can be chained to perform "double negation", though this is not immediately useful with any of the native types.

Finally, `*` is known the "splat" operator. Usage of the splat operator goes far beyond unary operation usage, and will be further explained in later sections, but the general idea of the operation is to return a List representation of the operand.

For example, `Map#*` returns a List of 2-element Lists with keys as the first elements and values as the second elements. For example:

```myst
*{a: 1, b: 2} #=> [[:a, 1], [:b, 2]]
```

The splat operator for a List (at least as a unary operation) will simply return the List itself.

```myst
*[1, 2, 3] #=> [1, 2, 3]
```

There generally are not many use cases for the splat operator as a unary operation. It is much more commonly used in the special cases described later on, namely in pattern matching and expanding arguments for functions.



## Operational assignment

As mentioned in the previous chapter, Myst provides a shorthand to perform an operation on a variable and assign it back into the same variable, known as _operational assignment_.

Operational assignments take the form `op=`, where `op` is the operator to apply before re-assigning the variable. The semantic expansion of operational assignments goes from `a op= b` to `a = a op b`. Some examples:

```myst
a = 1
a += 2  #=> 3
a *= 2  #=> 6
a %= 3  #=> 0
```

Operational assignment works for every _binary_ operation shown in this chapter. However, the logical operators (`||` and `&&`) have special behavior when written as operational assignments.


### Or-assignment

`||=`, referred to as "or-assignment" loosely expands from `a ||= b` to `a || a = b`, rather than the normal `a = a || b`. This is a "loose" expansion, because `a` does not have to exist before the assignment is written.

The expression `a ||= b` will always assign `b` into `a`, so long as the `a` is _not_ a truthy value. If `a` does not yet exist, it is considered falsey by the assignment and created as a new variable with the value of `b`.

If the value of `a` is truthy, the assignment will not occur, and the existing value for `a` will be returned as the result.

```myst
a ||= 2  #=> 2
# `a` is truthy, so no assignment occurs.
a ||= 3  #=> 3
z ||= nil   #=> nil
# `nil` is falsey, so assignment occurs.
z ||= false #=> false
z ||= 3     #=> 3
z ||= 4     #=> 4
```

A consequence of how `||=` works means that the target is guaranteed to exist after the assignment, making it useful for ensuring default values for variables that may not exist beforehand.


### And-assignment

`&&=` ("and-assignemnt") acts exactly like `||=`, but with an `&&` operation instead. In this case, the left side must be truthy for the assignment to occur.

```myst
a = false
a &&= 1     #=> false
a = 1
a &&= 2     #=> 2
z &&= 1     #=> nil
z &&= 2     #=> nil
```

The last two lines above may be a little surprising - one of the few cases of potentially unintuitive behavior in Myst. As mentioned above, `&&=` acts _exactly_ like `||=`, and a consequence of `||=` is that the target will _always_ exist after the operation is completed.

However, since the assumed value of a non-existent variable is falsey, the assignment in an `&&=` operation is not guaranteed to happen. Logically, this makes sense, but practically it can become annoying. For example, consider the code required if `&&=` did _not_ always initialize the target variable:

```myst
z ||= nil
z &&= 1
```

Here, we're trying to say that if `z` does not exist, create it with a `nil` value, then do the and-assignment. But, as you may have picked up, this actually won't always work, because if `z` exists beforehand but is falsey, the or-assignment will occur, potentially changing the value of `z` (if it was `false`, it would become `nil`).

To avoid this extra line that is still not guaranteed to work, Myst defaults the target variable to `nil`, which will ensure that the assignment does not occur (since a non-existent value is considered falsey), but will also ensure that the target variable is assigned after the operation is completed (matching the semantics of `||=`).