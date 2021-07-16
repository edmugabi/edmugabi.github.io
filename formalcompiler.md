# Can a compiler be formally described? 

More specifically, given a formal description of the execution semantics of a physical computer and a formal description of a computer program, can we always compile the program to  generate a binary is the most time and memory efficient?

Currently, building and optimising compilers is a specialised software engineering task. 

This journal is my attempt to research the problem in the open. It 
is not an answer to the question posed. Rather, it contains notes and articles collected on the problem. 
You the reader are warned about the possibility of finding half finished thoughts and more questions than answers. I am open to suggestions. Correct me at edmugabi@gmail.com

Why an open journal? The task is huge, difficult and important. It is possible that I may never be able to complete it. It is then better to have some form of continuous output that to finally reject with no output.

Lets get started.

## First Restatement of the problem:
1. Express the computer execution semantics as a typed higher order equational theory (E).
2. Express the program an an equation ie s = t.
3. Check if P is valid is E. Equivalently check if s can be rewritten to t using equations in E as rewrite rules.


## Guidelines:

- Try to solve small subproblems.
- Make as many assumptions as necessary to limit the scope of the problem.

## Scoping Assumptions:
- Ignore instruction pipelining. Assume only one instruction is executed at a time
- Ignore instruction caching.
