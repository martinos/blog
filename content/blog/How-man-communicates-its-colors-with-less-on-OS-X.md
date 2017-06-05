+++
date = "2010-10-19"
Categories = ["Development","Ruby"]
Description = ""
Tags = ["Development","Ruby"]
title = "How Man Communicates Its Colors With Less On Os X"
+++
Today I was playing with the `man` utility and I discovered that it does not use the same strategy as `ls` for coloring its output.  

If we type:

```bash
ls
```

Colors will be displayed and the files will be listed in multiple columns, but if we send `ls` stdout to `cat`:

```bash
ls | cat
```

Colors will be removed and there will be one file per line. Why?  
Because when `ls` detects that its stdout is a tty, it outputs the list in multiple columns with colors. If not, it will output one file per line without any color.

If we type:

```bash
man ls | cat
```

We don't get any colors.`man` behaves like `ls`. But if we type:

```bash
man ls | cat | less
```

`less` will output its colors to the terminal. It does not behave like `ls`.

So let's see the invisible characters that man outputs with `cat`'s `-v` option.

```bash
man ls | cat -v
```

We will discover that each colored character is printed, followed by a backspace and the same character is printed again. To help you understand here is an example:

To send a "HELLO" colored string to the `less` utility type:

```bash
printf "H\bHE\bEL\bLL\bLO\bO" | less
```

The `\b` a backspace character.

# Conclusion
On OS X, `man` communicates its colors by adding a backspace to the colored character and a copy it whether the stdout is a tty or not. `less` interprets those three characters as one colored character.

UPDATE: On Linux,`man` does not output backslashes when stdout is not the tty.
