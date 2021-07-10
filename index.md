# The environment model of evaluation.

There comes a time in every programmer's life when he/she needs to implement a computer language (sometimes disguised as a DSL) for fun or for profit. DSLs are every where and crop up when translating business rules into computer programs. The focus is often on writing an interpreter. Compilers are taken to be more difficult.

While it is easy to develop the syntax of the language using a parser generator, it is more involved to create an evaluator that represents the semantics of the language.

In this article, I show how one can create a basic evaluator for a target 
language by reusing equivalent abstractions in the source language. The
focus is on how function abstraction and application translate over using the environment model of evaluation. Language Primitive operations  (e.g `+`, `-`, `*`) and Constants (`strings`, `numbers`) are easy to implement. 

I recommend two books that cover the environment model excellently.
- Structure and Interpretation of Computer Programs(SICP) - Gerald Jay Sussman and Hal Abelson
- Writing an Interpreter in Go - Thorsten Ball

### Terminology
**Target Language** - The language being interpreted

**Source Language** - The language the interpreter source code is written in. The source language should allow functions as first-class citizens.

To develop some context, let us look at how an evaluator for basic arithmetic expressions is written.
```
eval(AST::Const(a)) => a
eval(AST::Left(l), '+', AST::Right(r)) => eval(l) + eval(r)
eval(AST::Left(l), '*', AST::Right(r)) => eval(l) * eval(r)
```
On the rhs, `a` is a value and `+` and `*` are binary operations in the source language.

How should function abstraction and function application be expressed?
Essentially we want to write:
```
eval(AST::App(l, r)) => ??
eval(AST::Lambda(args, body)) => ??
```
We can evaluate function application as:
```
eval(AST::App(l, r)) => (eval(l)) (eval(r))
```
The juxtaposition of `(eval(l))` and `(eval(r))` represents function application in the source language.
It is of the form:
```
    f (x)
```
where
---
`f = eval(l)` - Is a valid function/lambda in the source language.

`x = eval(r)` - should typecheck to the input type of f (in the source language !)

`f` should be generated from the pattern match:
```
eval(AST::Lambda(args, body)) => ??
```
With the restriction to use equivalent abstractions in the source langauge, we can imagine args translated to as:
```
eval(AST::Lambda(args, body)) => (...sourceArgs) => ??
```
The spread operator (`...`) allow creation of functions with variable input arguments.

In source languages that do not have variadic input arguments, we can create a case for each length(up to a maximum)

Given that args should correspond to sourceArgs, let us create a map data structure with AST variables in `args` as keys and the corresponding source variables in `sourceArgs` as values. 
```
    args[1] -> sourceArgs[1]
    args[2] -> sourceArgs[2]
    args[3] -> sourceArgs[3]
    ...
    args[n] -> sourceArgs[n]
```
It is clear that the new function body will depend on information in this map. However, args may not be the only source of variables for the new function body. The current lambda maybe sitting in the body(bodies) of higher lambda(s) and referencing their variables. 
```
(...args1) => 
     ...(args2) =>
         ........ 
           (...argsn)  =>    <---- we are here 
```
In addition, similary names variables in `argsn` should shadow all variables in higher lambdas (`args1` upto `argsn-1`)

To pass down this extra information, let us modify the eval function to take in an extra variable (the environment - `env`) that represents variables in higher lambdas. This environment is initially empty.
```
eval(AST::Const(a), env) => a
eval(AST::Left(l), '+', AST::Right(r), env) => eval(l, env) + eval(r, env)
eval(AST::Left(l), '*', AST::Right(r), env) => eval(l, env) * eval(r, env)
eval(AST::App(l, r), env) => (eval(l, env)) (eval(r, env))
eval(AST::Lambda(args, body), env) => (...sourceArgs) => ??
```
Each lambda should therefore extend env with its own variables shadowing variables from higher lambdas with the same name.

The rest of eval is shown below:
```
eval(AST::Lambda(args, body), env) => (...sourceArgs) => 
        newEnv = extend(args, sourceArgs, env)
        return eval(body, newEnv)

eval(AST::Var(v), env) => find(v, env)
```
`find` and `extend` are functions defined on the environment data structure and are easy to implement.

In conclusion, we have shown how to translate lambdas (representing functions) and applications using their equivalents in the source language. This can be helpful in prototyping the semantics of new languages. Tracking variable bindings using alternatives like De Bruijn notation is usually more difficult and unnecessary when the focus is first getting the syntax and semantics right.
