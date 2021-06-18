I've been diving into the [Elixir](https://elixir-lang.org/) programming language, and it's been incredibly eye-opening. Elixir is a completely functional language that compiles to Erlang's [BEAM](https://en.wikipedia.org/wiki/BEAM_%28Erlang_virtual_machine%29) virtual machine. Coming from a Ruby and Python background means I have to re-learn a number of concepts given the design of the language and re-orient myself around ideas like immutable data. Furthermore, Elixir has powerful tools like multi-headed functions, pattern matching, and arity which require an entirely different approach to programming. In this post, I want to talk about one feature which it does share in common with Python in particular: comprehensions.
<!--more-->

The traditional Python comprehension looks something like this:

```python
double_evens = [i * 2 for i in range(10) if i % 2 == 0]
> [0, 4, 8, 12, 16]
```

Like many things in Python, it has the benefit of being incredibly easy to read. Within the brackets, you put [(operation) for (iterable) if (condition)] for your result. A nearly English-like grammar.

In Elixir, the same code would look like this:

```elixir
double_evens = for x <- 0..9, rem(x,2) === 0, do: x * 2
> [0, 4, 8, 12, 16]
```

In an Elixir comprehension, you're first assigning a generator, then a filter, then an operation. In both cases, multiple lines of code can be neatly and concisely compressed into a single, readable line.

