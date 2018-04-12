# Feedback

This document focuses on some issues encountered while evaluating the submissions.
Please go through them and improve your code-base with the newly acquired knowledge.
It will be updated periodically.

## General

- **consistent formatting:**
  Use tools like `clang-format` or similar.
  Integrate them into your IDE or setup a Git commit hook.
  Or at least, keep an [EditorConfig](http://editorconfig.org/) around.
  For some people this even causes compiler warnings, like *misleading indentation*

- **README:**
  Your project should have *one* README which contains at least:
    - a list of all required dependencies
    - instructions on how to build everything and execute unit / integration tests
    - a section listing known issues, like memory leaks, crashes, or misbehaviours

- **library symbols:**
  Double check that all exported symbols of your library are prefixed.
  Have a look using `nm -g --defined-only libmCc.so`.
  If one of your symbols has no prefix, either add one or make it `static`.

- **enable and minimise warnings:**
  Make sure warnings are enabled and investigate them.
  Most of the time there is a pretty good reason why a warning occurs.
  *Statement with no effect* really means you should have a look at it, there may be something wrong!
  Also, **specifically** suppress instances of unwanted *unused-variable* / *unused-parameter* warnings --
  they just clutter the output and distract from the important ones.

- **streamline build infrastructure:**
  Do not use multiple build systems.
  It's perfectly fine if you want to use CMake, but drop Meson if you do.
  Also, let the build system take care of things, like setting up the include paths for the compiler or checking for dependencies.

- **separate interface from implementation:**
  The `include` folder should only contain the interface to the library you are building.
  Header files, which are part of the implementation (like a `struct` or `define` that is needed by the parser) should go else where, like the `src` directory.

- **do not directly print to `stdout` / `stderr`:**
  Even in error cases, reporting a failure should be through some internals means via the library's API, not by cluttering the standard outputs of an application.

- **usage information:**
  Your binaries should provide usage information, especially when they accept multiple different options.
  Furthermore, please follow a conventional, compact output format.

- **embrace `stdin` / `stdout`:**
  Even when your binaries are made to work with files, accept `-` for `stdin` / `stdout` and make sure this actually works.

- **remove / substitute debug output:**
  Often, people add `printf` statements for debugging in their code.
  Take them out or at least suppress them for release builds.
  Or introduce a logging mechanism with tunable log level.

- **embrace out-of-source builds:**
  Most build systems, like CMake, are made for out-of-source builds.
  This means that all auto-generated files go into the build directory.
  Nothing should be written to your source directly while building and testing.

- **keep things focused and composable:**
  For instance, the integration test script should not invoke the build system.
  Its purpose is to run the integration tests.
  If you desire a more automated way for building and running them, use a dedicated *run target* which executes the runner and depends on the `mCc` binary.
  Similarly, `mC_to_dot` should not execute `dot` / `xdot` / `xdg-open`.
  It should *only* output the AST in the DOT format.
  Use a 3 line shell script if you want to combine these commands.
  Of course, this also applies to source code via the composition of functions.

- **correct usage of assertions:**
  > Assertions are not a mechanism for handling run-time errors.

  See <https://ptolemy.eecs.berkeley.edu/~johnr/tutorials/assertions.html>.

## Assignment 1

- Your parser should not accept input which does not conform to the input grammar.
  `1 + 2;` is a valid statement, but not a valid input program.

- Unit test the most obvious parts requested by the assignment.
  For instance, there should be a dedicated test for operator precedence.

- Examine `realloc`'s return value before overriding anything.
  In case of a failure, it will be `NULL`, but the original block is not freed.

### Not Checking AST Constructor Return Values

The AST constructor functions, like `mCc_ast_new_expression_literal` return `NULL` on failure.
This return value has to be checked even inside the parser.

Let me illustrate this issue with a quick example.
Using the provided code-base, say we want to parse the input `2 + 3`.
Let's assume the left- and right-hand-side have been successfully constructed and we now need to construct the binary operation.
These are the corresponding rules in the parser:

    toplevel : expression { *result = $1; }

    expression : single_expr binary_op expression { $$ = mCc_ast_new_expression_binary_op($2, $1, $3); }

In other words, `$1`, `$2`, and `$3` are valid and passed on to `mCc_ast_new_expression_binary_op`.
Now, for some unforeseen reason, `mCc_ast_new_expression_binary_op` fails and returns `NULL`.
This is what happens next:

1. Both, left- and right-hand-side are leaked since we just lost these pointers.
2. `$$` is `NULL`, if this would be part of another expression, we'd now pass `NULL` as argument to another AST node constructor.
   A well placed `assert` may catch that during development, but no guarantees about production can be made.
3. Since this is the final expression, the top-level rule will use this as result.
4. Because the parser actually finished it will report `MCC_PARSER_STATUS_OK`, yet the result is `NULL`.

This is probably the worst case that can happen, but you get the idea.
So, check your return values -- always.
