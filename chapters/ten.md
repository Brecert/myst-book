# Chapter 10: Loading Code

So far, all of the examples we've seen have had their source code live in a single file. But, as a codebase grows, keeping everything in a single file will inevitably get messy.

Splitting code into multiple files makes helps with readability, usability, and traceability (through version control systems such as `git`).



## Require

Myst provides a single way to include code from another file: the `require` keyword. Using `require` will instruct the program to search the file system for a file with the name given by its argument.

The argument to a require can be any expression that evaluates to a String:

```myst
require "some_file.mt"
base = "folder/path/"
require base + "file.mt"
```

Any expression that does _not_ evaluate to a String will raise an error.


### Absolute paths

The most strict path specification for a file is an absolute path. These paths start at the root directory of the file system, following the given path to find the target file.

```myst
require "/home/me/src/foo.mt"
require "/etc/scripts/bar.mt"
```

Any path starting with a forward slash (`/`) is considered absolute. In almost all cases, absolute paths should be avoided with preference to relative or resource paths, as the code will inherently be less portable. Only use absolute paths for scripts with static locations on the system (basically never).


### Relative paths

Relative paths are the most common paths for requiring code in userland. Relative paths start either with a single (`.`) or double (`..`) dot, just like in plain Unix.

Relative paths will be looked up relative to the file that the `require` is given in, not necessarily the root directory of the project.

```myst
require "./foo/bar.mt"
require "../../lib/util.mt"
```


### Resource paths

The last type of path that can be given is referred to as a _resource path_. Resource paths check a number of paths for the existence of a file, starting with the directories given by the `MYST_PATH` shell environment variable.

If this variable is not set, or if no matching file is found in any of those paths, the current working directory (given by the `pwd` shell command) will be checked. Finally, the path will be treated as a relative path from the file containing the `require` expression.

Most often, resource paths are used to load third-party code or stdlib components:

```myst
require "http.mt"
require "foo.mt"
```


### Reloading files

Myst does not currently provide a mechanism for reloading a file. Once a file has been required in a program, it will be remembered, and any future attempts to require it will simply return `false`.

Files that have already been required are tracked by their _absolute path_, not necessarily the path that was given in the expression. For example, both of these paths refer to the same file, so only the first `require` will actually execute:

```myst
require "./foo.mt"
require "./bar/baz/../../foo.mt"
```



## File Structure

While there is no enforced file structure for a Myst-based project, there are a few general guidelines that will help keep your codebase easy to navigate and work with.


### Single responsibility

Much like the [single responsibility principle](https://en.wikipedia.org/wiki/Single_responsibility_principle) for modules and classes, each source file should generally have a single responsibility.

Most often, this lines up well with having a single module or type per file, though some deeply connected objects may make more sense in a single file, and a single type may be better spread into multiple files.

Giving each file a single responsibility also helps with naming conventions. In most cases, the file name can just match the name of the type or module it contains. For example:

```text
# In the file "foo.mt"
defmodule Foo
end

# In another file, "bar.mt"
defmodule Bar
end
```


### Module folders

When a module contains multiple types or modules within it, each should still be given their own file, but those files should be kept in a folder with the name of the containing module.

If there is any top-level code for the module, a file with the same name can also be given:

```text
# Structure:
#
# root
# |- foo.mt
# \- foo
#    |- bar.mt
#    \- baz.mt

# In foo.mt
defmodule Foo
  def create
    # ...
  end
end

# In foo/bar.mt:

defmodule Foo
  deftype Bar
  end
end

# In foo/baz.mt

defmodule Foo
  defmodule Baz
  end
end
```

Another way of using the top-level file for a module is to require all of the submodules and types, to simplify the end-user's experience when requiring the module:

```text
# In foo.mt
defmodule Foo
end

require "./foo/bar.mt"
require "./foo/baz.mt"
```

With this pattern, end-users can simply require the `foo.mt` file and get all of the submodules included automatically:

```text
require "./foo.mt"

%Foo.Bar{} #=> new Bar instance
```