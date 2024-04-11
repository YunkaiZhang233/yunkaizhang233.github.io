---
title: 'WACC and Beyond: Some Ideas on the Compiler Project'
date: 2024-03-26
permalink: /posts/2024/03/wacc-and-beyond/
tags:
  - compiler
  - coursework
  - imperial
  - plan
  - functional programming
  - Rust
redirect_from: 
  - /posts/wacc
---

**WACC** (officially pronounced "whack") is a simple variant of the While-like language family
that appeared in a series of courses on program reasoning and verification at Imperial.
The WACC Compiler project is one of the major group projects for the second year younglings in computing to overcome, and I am glad to say that we survived it with lots of fun and learning a lot. (Personally I was credited as one of the bug hunters for the reference compiler script!)

Moreover, I came up with some very interesting and super cool ideas, which I would be discussing later in this post and (hopefully!) eventually implement in.

If you are not familiar with it, the task is to write a compiler for the _WACC Language_ manually (i.e. craft the scenes from the middle game, such as Abstract Syntax Tree and Intermediate Representation all on ourselves) with several internal milestones.

The development was splitted into three stages:

1. Front-end
2. Back-end
3. Extensions

We got three weeks (along with other courses going on) on the previous two stages
and two weeks at the end of the term for the extensions.

## What have we DONE?

We were given a detailed (16 pages) specification of the syntax and semantics of the language, some relative definitions, and some other related references. Its syntax was mainly described in Backus-Naur Form (bnf) notation, whilst its semantics were grounded by sets of rules and behaviour restrictions.

To quickly get a rough feeling of how this language is like, here is a sample program from 

## What am I GOING TO DO/IMPROVE?
