# An Introduction to Myst

Welcome! In this guide, we'll be going through a tour of the Myst language, including syntax, terminology, some of the standard library, best practices, and more.

All of that will be covered in the sections that follow. This section, however, is focused on laying a foundation for the rest of the guide to build upon.


## Installing Myst

If you'd like to follow along and try out the sample code provided throughout this guide, you'll need to have Myst installed on your machine. See the [Installation guide](https://github.com/myst-lang/myst#installation) for details on how to get set up.

Note that Myst currently does not work on Windows machines. This has to do with how the language is implemented, and is something that is actively being worked on. Sorry to all the Windows users out there. Hopefully we will have an update for you soon.


## Sample Code

Throughout this guide, you'll see a lot of code samples that you should be able to copy and run for yourself to try things out. For example:

```myst
IO.puts("Hello, world!")
```

In more complex code samples, or where it is otherwise helpful, you may see lines with `#` at the beginning of them. These are comments that have no impact on the code itself, but simply provide some context or description for the reader. For example:

```myst
# Something describing the line below
some_line(of, code)
```

Most commonly, this is used to indicate alternatives. In these cases, the comment will normally say "Or", meaning that the code above coulde be used interchangeably with the code below the comment:

```myst
1 + 2 + 3
# Or
2 * 3
```

In a lot of cases, where a particular value is expected, you'll also see a `#=>` followed by some text inside the code sample. This is meant to help indicate the expected behavior of the code. For example:

```myst
1 + 2 #=> 3
```

This sample shows that the expected result of `1 + 2` is `3`. Note that this style doesn't actually have any effect in the code (the `#` indicates that the rest of the line is a comment), and is simply a helpful tool, particularly for complex examples.


## Reference notation

You may also see expressions like `List#push` or `IO.puts` in paragraphs or comments throughout this guid. This notation refers to a function that is contained within some type or module. These references all follow a consistent format, where a `#` between the module and the name indicates an _instance method_, while a `.` indicates a _static method_.

These concepts will be explained in more detail later on, but the basic premise is that _instance methods_ are called on an instance of a type (for example, `1` is an instance of the `Integer` type), while _static methods_ are called directly on the type itself.


## Asking Questions

This guide isn't perfect, and makes some assumptions about prior knowledge that might not be accurate for everyone, but that's okay! Having questions and asking them is an important part of learning.

If you do have a question, the best place to ask is in the [Myst Language discord server](http://discord.myst-lang.org/). There's an entire channel dedicated to asking questions, and an entire community to help answer them.

The questions people ask are an important part in helping us improve this guide. So, no matter how simple your question may feel, _ask it anyway_.

-----

With all that out of the way, let's get started! Head on to [Chapter 1](/get-started/chapter1.html) whenever you're ready.

Welcome to Myst!