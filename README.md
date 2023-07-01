# Inko Debugging Tools

Provides a simple package `debug` that contains two new `Format` implementations to aid
in debugging Inko programs.

## Installation

```sh
inko pkg add github.com/dusty-phillips/inko-debug 0.0.2
inko pkg sync
```

## Use

The main interface is the `debug` function, which will elegantly output any value that
implements [std::fmt::Format](https://github.com/inko-lang/inko/blob/main/std/src/std/fmt.inko#L83)
prefixed with the filename and line number the `debug` call was called from:

```
import debug::(debug)


fn pub a_problematic_function() {
  let some_val = ...
  debug(some_val)
}
```

During development this is a little more elegant and a little more useful than importing
`STDOUT` and `std::fmt::fmt` in all your code files.

## Other niceties

Filename and line number in debug output are coloured.

If your custom values use the `Formatter.descend` method to delineate nested objects,
and adds newlines to those nested objects, the debug formater will automatically add
indentation of two spaces per nesting level to each line.

If you want to format indented lines without debug's filename and line number, you can use
the `ind_fmt` function, passing in the value you want to format. It returns a string.

If you want to format the lines with filename and line number, but not automatically print
to STDOUT, use the `debug_fmt` function, which returns a string.

For more control, you can also access the `debug::(IndentFormatter)` and `debug::(DebugFormatter)`
classes directly. Both implement `std::fmt::Formatter`.

## Example of fmt method for nested output:

The standard `Array` class doesn't use `Formatter.descend`, so it puts all elements on one line by default.
Here is a wrapper to illustrate how to put each value on a separate line:

```
import std::fmt::(Format, Formatter)
import debug::(debug)

class MyArray[T] {
  let @array: Array[T]

  fn static new() -> MyArray[T] {
    MyArray {
      @array = []
    }
  }

  fn mut push(value: T) {
    @array.push(value)
  }
}

impl Format for MyArray if T: Format {
  fn pub fmt (formatter: mut Formatter) {
    formatter.write("[\n")
    # Descend increments the nesting value, so IndentedFormatter
    # will add two spaces to the line.
    formatter.descend fn () {
      @array.iter.each fn (el) {
        el.fmt(formatter)
        formatter.write(",\n")
      }
    }
    formatter.write(']')

  }
}


class async Main {
  fn pub async main() {
    let array = MyArray.new()
    array.push("A")
    array.push("B")

    debug(array)

    # Recursive nesting works, too
    let array2 = MyArray.new()
    array2.push(array)

    debug(array2)

  }
}
```

The output of the above `debug` call:

```
main.inko(39) [
main.inko(39)   "A",
main.inko(39)   "B",
main.inko(39) ]
main.inko(44) [
main.inko(44)   [
main.inko(44)     "A",
main.inko(44)     "B",
main.inko(44)   ],
main.inko(44) ]
```

Admittedly, a wrapper of Array just for formatting is pretty silly, but this can
be useful for deeply nested or recursive structures.
