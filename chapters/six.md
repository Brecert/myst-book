# Chapter 6: Modules

As codebases grow, functions and objects tend to form into implicit groups around various actions, concerns, or other concepts, but with no tangible way to associate them. Just as likely is the risk of a new value wanting the same name as an existing value, causing a _collision_.

To deal with these issues, Myst has Modules, which are simply collections of values that live under a single name in the containing scope. This both creates an explicit, visible connection between the values and reduces the likelihood of name collisions by keeping all of the values within the module out of the containing scope.



## Defining Modules

Modules are defined using the `defmodule` keyword with a name for the module, followed by a body of expressions. Module names are required to be Constants. The return value of a module definition will be the module itself.

The body of a module is just like the main body of a program. Any series of expressions can be written inside of the module body, and any expression that would add a value to the containing scope will instead write to the module's scope.

A basic module definition might look something like this:

```myst
defmodule Foo
  def bar(a, b)
    a + b
  end

  baz = 2
end
```

This example creates a module `Foo` with a variable property `baz`, and a function `bar`.



## Using Modules

Accessing a property of a module is done with the _postfix Call_ syntax. For example, getting the `baz` property or calling the `bar` method from the `Foo` module defined above would look like this:

```myst
Foo.baz       #=> 2
Foo.bar(4, 5) #=> 9
```

Here, `Foo` is referred to as the _receiver_ of the Call, and `bar` or `baz` are the _names_ being called.

Since `bar` was defined inside the module, it is not accessible from outside of the module's scope. More importantly, this means that `bar` could also be defined _outside_ of the module as a separate function. For example:

```myst
def bar(a, b)
  a + b
end

defmodule Foo
  def bar(a, b)
    a - b
  end
end

# This calls the first definition of `bar`.
bar(4, 2)     #=> 6
# This calls the second definition.
Foo.bar(4, 2) #=> 2
```

In this example, there are two distinct function objects. The first definition of `bar` creates a new function object in the main scope of the program.

The second definition attempts to lookup an existing function object with the name `bar`, but will not find one, because the main scope has been hidden by `Foo`'s module scope. So, the second definition will _also_ create a new function object, but will save it in the module's scope instead.



## Nesting

Since the body of a module can contain any set of expressions, it is also possible to define modules within other modules. This is generally used to associate many related modules under a namespace. For example:

```myst
defmodule Foo
  defmodule Bar
    def foo
      :bar
    end
  end

  defmodule Baz
    def foo
      :baz
    end
  end
end
```

Accessing properties of these inner modules is done by digging down to them from the top scope with the same postfix Call syntax:

```myst
Foo.Bar.foo #=> :bar
Foo.Baz.foo #=> :baz
```

When two or more modules appear next to each other in a Call, it is often called a _path_. So, with the above, you might see `Foo.Bar` and `Foo.Baz` referred to as _paths_.



## Includes

Modules are the basis of composition in Myst. When multiple Modules (or Types, as we'll see in the next chapter) share a lot of the same functionality or need to conform to a specific API, it's a good idea to pull that functionality into a separate Module. This avoids repetition in code, helping reduce the size of the codebase, and limit the potential for copy-paste errors to occur.

Once the functionality has been abstracted into a separate Module, it can be added to those other Modules using the `include` keyword. A simple example:

```myst
defmodule Foo
  def foo
    :foo
  end
end

defmodule Bar
  include Foo
end

Bar.foo #=> :foo
```

Notice that `foo` was never defined directly on `Bar`, but was still able to be called. This is the effect of `include`. When a Module gets included, it is added as an _ancestor_ of the including object. When Calls are looked up (the call to `foo` in this case), the language will first look at the receiver to see if it has a property with that name.

If no property is found, the language will then check all of the ancestors of the receiver, in order, to try to find the property. Since `Bar` included `Foo` in the above example, the language will try to find `foo` on `Bar`, fail, and then successfully find `foo` from the `Foo` module.


### Dynamic Includes

The argument to `include` can be any expression that evaluates to a Module object (only Modules can be included). And, since `include` is a normal expression like everything else, it can be used anywhere, even inside of function definitions. Here's a more complex example:

```myst
defmodule Foo
  def include_other(module)
    include module
  end
end

defmodule Bar
  def bar
    :bar
  end
end

Foo.include_other(Bar)
Foo.bar #=> :bar
```

In this example, the `include` is given inside of the definition of `include_other`, and the `module` argument to it is the local variable of that function. When `include_other` is called at the end of the example, it is given the module `Bar` as an argument, so the `include` ends up being `include Bar`. With that, `Foo` now has `Bar` as an ancestor, and the method `bar` can be called from `Foo` directly.

Objects are also not limited to any number of includes. Any object can include any number of other modules, though doing this too much can cause the ancestor list to grow long and may slow down Calls.


### Hiding Properties

One final thing to note about includes is that the properties of the other module are _not copied_ into the including object. This means that defining a function with the same name as one from an included module will create a new function object, and may hide the other function from the included module.

This can have unintended effects where the top-down behavior of function clause lookup is expected but not followed:

```myst
defmodule Foo
  def add(a, b)
    :add_from_foo
  end

  def sub(a, b)
    :sub_from_foo
  end
end

defmodule Bar
  include Foo

  def add(a, b)
    :add_from_bar
  end

  def sub(a)
    :sub_from_bar
  end
end

Bar.add(1, 2) #=> :add_from_bar
Bar.sub(1, 2) #=> no matching clause
```

The behavior of `add` in the above example may be intuitive. Since `Bar` defined `add` directly, it makes sense that it is being called, instead of `Foo.add`, which was defined first, but only as an included property.

On the other hand, the behavior of `sub` may not be so obvious. The definition in `Bar` only accepts one parameter, while the definition in `Foo` accepts two. The call `Bar.sub(1, 2)` will look for a clause that accepts two parameters, but since the definition in `Bar` created a new function object, the two-parameter definition from `Foo` has been hidden. So, the `sub` property is _found_ in Bar, but there is no matching clause, causing an error.



## The Top-Level Module

As mentioned above, the body of a module is "just like the main body of a program". Part of the reason that this works is that the main body of a program is itself a module. Most often, this is referred to as the _top-level module_, or just _the top-level_.

The most important implication of this is that `include` can be used at the top-level:

```myst
defmodule Foo
  def foo
    :called_foo
  end
end

include Foo
foo #=> :called_foo
```

This is actually a common pattern, often used to define _domain-specific languages_ for things like spec frameworks or script helpers, without having to constantly reference module names throughout the program.