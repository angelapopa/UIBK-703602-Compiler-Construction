# Questions

This document records questions and answers occurring throughout the course.

## AST Related

> \<QQ\> how should the resulting AST look like, can you give us an example pic?
> Should it only consists of one assignment?
> Else the order of the tasks makes no sense.

The resulting AST is a representation of the corresponding input program.
The exact design of your AST data structure is up to you.
Depending on the design decisions you make, some things will be harder to implement, others easier.

You should integrate a way to visualise this data structure as early as possible, as it eases the process of debugging.
And because it looks nice.

As an example, take the following function, being part of a larger input program:

```c
int fib(int n)
{
	if (n < 2) return n;
	return fib(n - 1) + fib(n - 2);
}
```

The visualisation of an AST representing this function could look like this:

![fib AST](gfx/fib_ast.png)

*Variable names are prefixed with an id followed by an underscore.*

Every statement, expression, literal, operator, etc; is modelled here.
This would be a subtree of the AST representing the whole input program.
