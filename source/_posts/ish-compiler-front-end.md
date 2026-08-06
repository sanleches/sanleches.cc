---
title: Ish, My Small Compiler Front End in C
date: 2026-08-06 14:00:00
category: projects
tags:
  - C
  - compilers
  - parsers
  - language design
---

I wanted to understand compilers without hiding everything behind a parser generator, so I started [Ish](https://github.com/sanleches/Ish), a tiny language front end written in C11.

Calling it a complete compiler would be cheating. Right now Ish reads source, tokenizes it, parses it, builds an abstract syntax tree, tracks symbols, and reports a few semantic errors. It does not generate machine code, bytecode, or an executable yet. It is the front half of a compiler, which is still the half where a surprising amount of interesting stuff happens.

## A tiny language on purpose

Ish syntax looks like this:

```ish
funk sayHello() {
    print("Hello, world!");
}

sayHello();
```

There are variable declarations, assignments, function declarations and calls, `while`, `return`, `print`, `input`, arithmetic, and comparisons. Running parser mode with `--dump-ast` turns that example into a tree containing a program, function declaration, print expression, literal, and function call.

It does not print `Hello, world!` yet because there is no runtime. At this stage, successfully understanding the program is the output.

## The pipeline

The executable has separate modes for inspecting each phase:

```bash
./build/ish r examples/hello.ish
./build/ish s examples/scanner-smoke.ish
./build/ish p examples/hello.ish --dump-ast
```

The reader loads source into a custom dynamic buffer with fixed, additive, and multiplicative growth modes. The scanner walks that buffer and emits tokens. String literals go into a separate string pool, so a string token stores an integer offset instead of owning another allocation.

The parser is hand-written recursive descent with one token of lookahead. Operator precedence is encoded by the function call structure: comparison expressions call additive expressions, additive expressions call multiplicative expressions, and those call primary expressions. The parser structure becomes the precedence table.

The AST uses a fixed arena of 512 nodes. Nodes refer to their first child and next sibling by integer index rather than heap pointers. It is compact, easy to dump, and makes the memory model obvious.

## Symbols while parsing

Ish currently performs basic name checking during parsing. Declarations are inserted into a fixed symbol table, undefined uses are reported, and symbols are removed from the end when a scope closes. That works for duplicate declarations and simple undefined-name checks, although it also means function calls generally need the declaration to appear first.

The cleaner long-term design is a separate semantic pass over the AST. That is also where type checking belongs. The project already stores declared types, but it does not enforce them yet.

## Why bother doing this by hand?

Because a scanner stops feeling mysterious after you write one. The same thing happens with precedence, symbol scope, AST layout, and diagnostics. Keeping the language small means I can still understand the complete path from a character in a file to a node in a syntax tree.

The next genuinely big step is choosing a backend: an intermediate representation and VM, or code generation. Until then, Ish is intentionally a compiler-construction playground with visible internals and a small C codebase.

Code: [github.com/sanleches/Ish](https://github.com/sanleches/Ish)
