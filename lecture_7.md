# CS247 Lecture 7 _May 27th_

## Valgrind

Mem checker tool that detects memory leaks and memory errors in C/C++ programs.
Can only detect leaks that are triggered at runtime. Does not do static analysis.

REMEMBER THE WISE WORDS OF ALLEN:

- g++ -Wall : all warnings
- g++ -Wextra : actually all warnings
- g++ -Werror : makes warnings errors
- g++ -fsanitise=address : compiles with memcheck
- g++ -g : for gdb

## GDB

use `layout src` for a nice gui, `refresh` to fix it
