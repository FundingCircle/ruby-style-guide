# A Ruby style guide

See `.ruby-style.yml` for the nitty-gritty details.

This guide extends the excellently written, community maintained
[Ruby Style Guide](https://github.com/bbatsov/ruby-style-guide).

This style guide extends the community Ruby style guide and can be considered
to be project specific errata. The only significant differences being:

0. Where a rule in community style guide allows multiple styles,
  we specify a preference for specific variation of that rule.
0. We disable rules enabled by default that we feel detract from code quality or legibility
0. We attempt to clarify rules that are unclear or undefined

## Table of Contents

0. [Rules we disable](#rules-we-disable)
  0. [Class and module children](#class-and-module-children)
  0. [Documentation of modules, classes, and methods](#documentation-of-modules-classes-and-methods)
0. [Rule violations that result in errors](#rule-violations-that-result-in-errors)
  0. [Circular argument references](#circular-argument-references)
  0. [Forgotten breakpoints](#forgotten-breakpoints)
  0. [Duplicate methods](#duplicate-methods)
  0. [Duplicate keys](#duplicate-keys)
  0. [Immutable argument for `each_with_object`](#immutable-argument-for-each_with_object)
  0. [Usage of `eval`](#usage-of-eval)
  0. [Unreachable code](#unreachable-code)
  0. [Useless comparisons](#useless-comparisons)
0. [Differences from the community style guide](#differences-from-the-community-style-guide)
  0. [Alignment of `end` keyword](#alignment-of-end-keyword)
  0. [Line length](#line-length)
  0. [Method length](#method-length)
  0. [Parameter alignment](#parameter-alignment)
  0. [Block delimiters](#block-delimiters)
  0. [Braces around hash parameters](#braces-around-hash-parameters)
  0. [Dot position](#dot-position)
  0. [Array indentation](#array-indentation)
  0. [Hash indentation](#hash-indentation)
  0. [Multi-line method call indentation](#multi-line-method-call-indentation)
  0. [Trailing commas in array and hash literals](#trailing-commas-in-array-and-hash-literals)

## Rules we disable

### Class and module children

Cop: `Style/ClassAndModuleChildren`

By default the ruby style guide enforces "compact" or "nested" module children definitions. We tend to use a mix of both, making this rule unnecessary.

```ruby
# compact
class Foo::Bar
end

# nested
module Foo
  class Bar
  end
end
```

### Documentation of modules, classes, and methods

Cop: `Style/Documentation`

While it is encouraged to document your code, failure to provide inline documentation for _all_ of your code should not result in a warning or build failure.


## Rule violations that result in errors

We consider classes of style violations to be bad enough to be considered errors. It's strongly encouraged to trigger build failures if any of these style violations are encountered.

### Circular argument references

Cop: `Lint/CircularArgumentReference`

```ruby
# bad
def bake(pie: pie)
  pie.heat_up
end

# good
def bake(pie:)
  pie.refrigerate
end

# good
def bake(pie: self.pie)
  pie.feed_to(user)
end

# bad
def cook(dry_ingredients = dry_ingredients)
  dry_ingredients.reduce(&:+)
end

# good
def cook(dry_ingredients = self.dry_ingredients)
  dry_ingredients.combine
end
```

### Forgotten breakpoints

Cop: `Lint/Debugger`

Hard coded breakpoints should _never_ be committed.
Such code will cause all sorts of problems if it were to ever be deployed.

```ruby
# bad
def foo
  binding.pry
  "bar"
end

# bad
def foo
  debugger
  "bar"
end
```

### Duplicate methods

Cop: `Lint/DuplicateMethods`

Duplicate methods cause two very problematic issues:

0. They result in dead code that is never reached
0. In cases like the second example below, they can result in code drift between method implementations. This is particularly insidious because it can result in changes that are clobbered by any duplicated implementation of that method.

```ruby
# bad
def foo
  "bar"
end

def fizz
  "buzz"
end

def foo
  "bar"
end

# bad
def a
  "b"
end

def b
  "b"
end

def a
  "a"
end
```

### Duplicate keys

Cop: `Lint/DuplicatedKey`

Duplicate keys are problems for the same reasons that [duplicate methods](#duplicate-methods) are.

```ruby
# bad
hash = { food: 'apple', food: 'orange' }
```

### Immutable argument for `#each_with_object`

Cop: `Lint/EachWithObjectArgument`

The example below results in a sum of 0.
This happens due to the fact that integers in ruby are immutable, resulting in accumulator of `each_with_object` remaining static through each iteration of the block.

```ruby
# bad
sum = numbers.each_with_object(0) { |e, a| a += e }
```

### Usage of `eval`

Cop: `Lint/Eval`

The usage if `Kernel#eval` is dangerous enough blacklist under normal circumstances. If you ever find yourself needing to use `eval` in code you wish to commit, get feedback from a thorough code review.

There is likely a safer way to solve your problem.

### Unreachable code

Cop: `Lint/UnreachableCode`

Code that cannot be reached is a strong signal of a bug. If the code really should be unreachable, remove it.

### Useless comparisons

Cop: `Lint/UselessComparison`

```ruby
# bad
x.top >= x.top
```


## Differences from the community style guide

## Alignment of `end` keyword

Cop: `Lint/EndAlignment`

Alignment of the `end` keyword can vary a lot due to personal preferences in code style.

Rubocop supports three styles that are commonly used in idiomatic ruby code:

0. `keyword`: `end` shall be aligned with the start of the keyword (if, class, etc.)
0. `variable`: the `end` shall be aligned with the left-hand-side of the variable assignment, if there is one
0. `start_of_line`, the `end` shall be aligned with the start of the line where the matching keyword appears

We dictate the "start_of_line" style to ensure we all write code in a consistent manner.

```ruby
# good
variable = if true
end

# bad
variable = if true
           end

# good
puts(if true
end)

# bad
puts(if true
     end)
```

### Line length

Cop: `Metrics/LineLength`
Severity: `refactor`
Max: 120

The community ruby style guide recommends a line length of 80 characters.
In practice this can be a bit short for the code we typically write, so we recommend a maximum line length of 120 and downgrade the severity of line length rule violations to "refactor", meaning a line length violation should never fail a build.

### Method length

Cop: `Metrics/MethodLength`
Severity: `refactor`

The community ruby style guide recommends a maximum method length of 10 lines.
This is a good goal to shoot for, but there are many situations where forcing code to fit into a series of 10-line methods can harm the clarity and legibility of the code in question.

We downgrade method length rule violations so that we flag long methods with the lowest possible severity so that we raise awareness of long methods without forcing users to contort them without reason.

### Parameter alignment

Cop: `Style/AlignParameters`

Alignment of parameters in multi-line method calls. Rubocop supports two styles:

0. `with_first_parameter`: aligns arguments along the same column as the first parameter
0. `with_fixed_indentation`: aligns the following lines with one level of indentation relative to the start of the line with the method call

```ruby
# `with_first_parameter` style
# bad
method_call(a,
            b)

# `with_fixed_indentation` style
# good
method_call(a,
  b)

# good
method_call(
  a,
  b,
  c
)
```

### Block delimiters

Cop: `Style/BlockDelimiters`

TODO

### Braces around hash parameters

Cop: `Style/BracesAroundHashParameters`

TODO

### Dot position

Cop: `Style/DotPosition`

TODO

### Array indentation

Cop `Style/IndentArray`

TODO

### Hash indentation

Cop: `Style/IndentHash`

TODO

### Multi-line method call indentation

Cop: `Style/MultilineMethodCallIndentation`

TODO

### Trailing commas in array and hash literals

Cop: `Style/TrailingCommaInLiteral`

TODO
