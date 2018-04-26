# Chapter 7: Types and Self

Beyond Modules for grouping values together into a namespace, Myst also provides a mechanism for defining Types and Instances.

Types are useful for defining complex data structures and ways to interact with that data more cleanly than could be done with Maps or other values.

Every value in Myst is an instance of a type. For example, the number `1` is an instance of the type `Integer`, and `"hello, world"` is an instance of `String`.

The majority of popular object-oriented languages refer to types as "classes". In practice there is no distinction between the two - the name "type" is simply a preference of the language.


## Definition and Instantiation

To define a Type, Myst uses the `deftype` keyword. The structure is similar to Modules, where the name must be a Constant, and the body is any series of expressions. A simple example of a type might look like this:

```myst
deftype Car
end
```

To create an instance of this Type, Myst uses a special syntax that resembles Struct instantiation from Elixir. For example, creating an instance of the Car class would look like this:

```myst
%Car{}
```

The result of instantiation is a new Instance value of the Type specified by the instantiation. These values are just like any other value, and can be used as such.

```myst
my_car = %Car{}
buy_for(%Car{}, 10_000.00)
```

The type for instantiation can also be specified with an interpolation:

```myst
deftype Foo
  def call; :foo; end
end

deftype Bar
  def call; :bar; end
end

def make_instance(type)
  %<type>{}
end

f = make_instance(Foo)
b = make_instance(Bar)
f.call #=> :foo
b.call #=> :bar
```

The interpolation can contain any expression that resolves to a Type.


## Instance Variables and Initialization

When a new Instance of a Type is created, it goes through _initialization_, where the properties of the instance can be configured. This can be controlled by defining an `initialize` instance method on the Type.

```myst
deftype Car
  def initialize
    # configure the new instance here.
  end
end
```

If no `initialize` method is given on a Type, instances of that Type will be created with no default configuration.

Instance variables are the properties of a Type. In our Car example, we could add instance variables for the color, the number of doors, the body style, etc. Every Instance will have its own set of instance variables.

Instance variables in Myst are created and referenced with identifiers that are prefixed with an `@`. By default, the instance variables of a Type are private, and must be exposed by methods on the Type to be made public. Attempting to access an instance variable before it has been set will create the variable with an initial value of `nil`.

```myst
deftype Car
  def initialize(color, door_count)
    @color      = color
    @door_count = door_count
  end

  def color; @color; end
  def color=(c); @color = c; end

  def door_count; @door_count; end
  def door_count=(dc); @door_count = dc; end
end
```

Here, `Car` has two instance variables, `@color` and `@door_count`. Each instance will have it's own copies of these instance variables. The instance variables are created as soon as they are assigned.

The `initialize` method above accepts two parameters. Arguments are passed to `initialize` by listing them between the curly braces used in instantiation:

```myst
  sedan = %Car{"gray", 4}
  coupe = %Car{"red",  2}
```

The groups of methods at the bottom of this definition of `Car` are generally referred to as _getters_ and _setters_ for the instance variables. These expose the private instance variables as public properties that can be interacted with.

With the way Myst handles Calls and assignments, these methods essentially allow us to treat the properties of the Type as normal variables. For example, updating the color of the `sedan` instance from above can be done just like a normal assignment:

```myst
sedan.color = "blue"
```

These Calls also work with operational assignments, and any other places where variables are used:

```myst
sedan.door_count += 2 #=> 6
```

An important note is that instance variables are _not_ valid outside of instance methods. Using an instance variable in the type definition body or anywhere else will likely cause unexpected behavior.


## Self

When dealing with Instances, there are often cases where methods would like to reference the object that they belong to. To accomodate this, Myst provides a special `self` variable, which refers to that object.

```myst
deftype Foo
  def reflect
    self
  end
end

f = %Foo{}
f.reflect == f #=> true
```

`self` can also be used to disambiguate method calls from local variables. In particular, this is useful for calling getters and setters from initializers. Consider this code:

```myst
deftype Person
  def initialize(given_name)
    full_name = given_name
  end

  def full_name; @full_name; end
  def full_name=(full_name)
    @full_name = full_name
    [@first_name, @last_name] =: full_name.split
  end

  def first_name; @first_name; end
  def last_name;  @last_name; end
end

p = %Person{"Freddy Mercury"}
p.full_name   #=> nil
p.first_name  #=> nil
p.last_name   #=> nil
```

Something that may or may not be obvious here is that the `full_name` setter (`full_name=`) will _not_ be called, and the `@full_name` instance variable will not be assigned. Instead, the assignment in the initializer is interpreted as a new _local_ variable `full_name`, rather than a call to the `full_name=` method.

One solution to this problem would be to assign the `@full_name` instance variable directly in the initializer:

```myst
def initialize(given_name)
  @full_name = given_name
end

# ...
p = %Person{"Freddy Mercury"}
p.full_name   #=> "Freddy Mercury"
p.first_name  #=> nil
p.last_name   #=> nil
```

However, there could be some cases where there is added functionality for setting a variable, and calling the setter method is more desirable. For example, our `full_name=` setter for the `Person` type above also assigns the `@first_name` and `@last_name` instance variables. Without calling the setter, these variables are not assigned.

The other solution is to use `self` to ensure that the assignment is interpreted as a call to `full_name`:

```myst
def initialize(given_name)
  self.full_name = given_name
end

# ...
p = %Person{"Freddy Mercury"}
p.full_name   #=> "Freddy Mercury"
p.first_name  #=> "Freddy"
p.last_name   #=> "Mercury"
```


## Static Methods

In addition to instance methods, Types can also define _static_ methods. Static methods are methods that can be called from the Type itself, rather than instances of the type. In this sense, static methods essentially act like methods on a Module.

To define static methods, Myst uses the `defstatic` keyword:

```myst
deftype Foo
  defstatic foo
    :static_foo
  end
end

Foo.foo #=> :static_foo
```

One common use case for static methods is defining new ways of constructing instances. For example, we could define various methods for instantiating different types of Cars:

```myst
deftype Car
  defstatic coupe(color)
    %Car{color, 2}
  end

  defstatic sedan(color)
    %Car{color, 4}
  end

  def initialize(color, door_count)
    @color = color
    @door_count = door_count
  end

  def color; @color; end
  def door_count; @door_count; end
end

c = Car.coupe("blue")
c.door_count = 2
s = Car.sedan("gray")
s.door_count = 4
```

Inside of static methods, `self` refers to the Type being modified. This can be useful when creating instances of the type, helping to avoid potential refactoring issues:

```myst
deftype Foo
  defstatic create
    %<self>{}
  end

  def to_s
    "foo"
  end
end

IO.puts(Foo.create) #=> "foo"
```

This example uses the _interpolated instantiation_ syntax (`%<self>{}`) to dynamically specify the type for instantiation. This avoids explicitly repeating the name `Foo`, which could help avoid errors when refactoring the name of the class.


## Re-opening Types

Just like Modules, Types can be _re-opened_ to append to their definition. For example, starting with a simple definition of a Type:

```myst
deftype Foo
  def initialize(value : Integer)
    @value = value
  end

  def add(other : Integer)
    @value += other
  end
end

foo1 = %Foo{10}
foo1.add(2)
```

Later on, potentially even in another file, this type could be re-opened to add new methods to it:

```myst
deftype Foo
  def sub(other : Integer)
    @value -= other
  end
end

foo2 = %Foo{10}
```

Now, the type `Foo` has both an `add` method and a `sub` method. All instances of `Foo` (even ones that were created before the Type was re-opened) can use both of these methods freely:

```myst
foo2.add(3)
foo1.sub(4)
```


## Include and Extend

Just like Modules, Types can include other Modules to bring in new functionality. Using `include` in a Type will add the Module's methods as instance methods on the Type.

```myst
defmodule Foo
  def foo
    :module_foo
  end
end

deftype Bar
  include Foo
end

%Bar{}.foo #=> :module_foo
```

In addition, Myst provides a mechanism for adding Modules as _static_ methods using the `extend` keyword. The result of `extend` is essentially the same as using `include` from within another Module.

```myst
defmodule Foo
  def foo
    :module_foo
  end
end

deftype Bar
  extend Foo
end

Bar.foo #=> :module_foo
```

`extend` can be useful for quickly implementing class-level DSLs such as schema definitions or or common patterns like the [Factory pattern](https://en.wikipedia.org/wiki/Factory_method_pattern).