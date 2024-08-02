---
title: 'Effect Handlers - 01 : Basics & Examples'
date: 2024-06-19
permalink: /posts/2024/06/algebraic-effect-handlers-01/
tags:
  - Notes
  - Programming Languages
  - Effects
  - Functional Programming
redirect_from: 
  - /posts/algebraic-effect-handlers
---

These short notes summarise my own initial exploration of effect handlers, as a

foundation for further studies of lexical effect handlers.

Effect Handler is a relatively new concept in programming languages proposed by Plotkin and Pretnar that has now been incorporated into [OCaml 5.0](https://ocaml.org/manual/5.2/effects.html). This note will keep most of the discussion at an "intuitive" level and could contain some mistakes.

## References

Here are some useful materials I referred to for my own reading:

- Matija Pretnar, [An Introduction to Algebraic Effects and Handlers. Invited tutorial paper](https://www.sciencedirect.com/science/article/pii/S1571066115000705)
  
- Andrej Bauer, [[1807.05923] What is algebraic about algebraic effects and handlers?](https://arxiv.org/abs/1807.05923)
  
- Sam Lindley, [Effect handler oriented programming - YouTube](https://youtu.be/X30xmcOow2U) (multiple versions of this talk available).
  
- Sam Lindley, [Handler Calculus](https://youtu.be/-OXQkJMU1d4).
  
- [EHOP homepage](https://effect-handlers.org)
  
- Ningning Xie, [Keynote: Algebraic Effect Handlers with Parallelizable Computations](https://youtu.be/XCVg_cc9Jo4)
  
- [Concurrent Programming with Effect Handlers](https://github.com/ocaml-multicore/ocaml-effects-tutorial/tree/master)
  
- Philipp Schuster, Jonathan Immanuel Brachthäuser, Marius Müller, and Klaus Ostermann. [A typed continuation-passing translation for lexical effect handlers](https://youtu.be/71fV7zYyD-Q).
  
## Introduction

The essence of effect handler is to cope with the "ad-hoc" and "hard-wired" interactions and the consequences of effectful programming, as suggested by its name.

Imagine programs as blackboxes that takes input and gives output. In real-world scenarios programs must interact with the environment and will encounter **"effectful"** operations. Such effects are very pervasive and covers different aspects of a program. These interaction with the world must also be properly handled alongside pure functional programming.

To give some examples, these include: IO, Read/Write, Exception, Mutable State, Concurrency, Backtracking.

However, if we **directly** merge them into the functional programming system without alternation, problems will arise as in many programming languages these effects are often **ad-hoc** and **hard-coded**. These effects are usually implemented separately and it would be hard to alter the behaviours of such effects.

We must introduce some mechanism for **composable and structured control-flow abstraction in a modular way**. Another thing may come to mind would be **monads** stuff, including like **state monads** and **monad transformers**. I'll only talk about their differences with algebraic effect handlers later should I have time and let's keep the focus to the **effect handlers**, proposed by Plotkin & Pretnar at 2009 [1].

Effect handlers allow us to **define and use computational effects** in a flexible way with allowing lots of space for customisation. It has multiple applications and has trigued great interest from Academia to industry.

## Understanding Algebraic Effects via Examples

These examples were taken and altered from Sam Lindley and Ningning Xie's talk [2][3].

### Algebraic Effects & Effect Types

Lets first consider the following **effect signatures** (signatures of Effects).

**idea**: we start with _a signature of operations_ that our effect is allowed to do.

```ocaml
Choose {
    choose : () -> Bool
}
```

```ocaml
Exn {
    fail : () -> a
}
```

The `Choose` effect type contains a single **operation** of `choose`, which further takes a single unit and returns a `Bool`.

Similarly, the `Exn` (for _Exception_) effect type contains a single operation of `fail`, which takes a unit and returns a polymorphic variable `a` (=return a value of any type).

Under a real-world context, it should not be hard to grip the meaning of these and we could already start to using effect operations in a program.

---

Let's consider a normal coin tossing, which gives a result of either head or tail.

The result of a normal toss, should be of type `Toss`, which contains only `Heads` and `Tails`.

```coq
Definition Toss := Heads | Tails
```

```ocaml
toss : () -> < Choose | ε > Toss
toss () = if choose() then Heads else Tails
```

The semantics of the `toss` function is quite straitforward. Considering its type: the type annotation of the function contains the **effect information** that says "it's a function that may perform effectful operations via the `Choose` effect type" (well, maybe a few others from the $\epsilon$ but we don't care about that for the moment).

---

Let's now consider a slighly more complicated coin tossing, **"drunk coin tossing"**. Now imagine you are a drunk man and you want to toss a coin, you may not be able to successfully complete such mission, introducing the possible scenario of failing (i.e. Exceptions).

```ocaml
toss : () -> < Choose | ε > Toss
toss () = if choose() then Heads else Tails

(* extension *)

drunkToss : () -> < Choose | Exn | ε > Toss
drunkToss = if choose() then
                if choose() then Heads else Tails
            else fail()
```

now the two `choose()` function in `drunkToss` definition has different meanings. The first one is to indicate "drunk or not" - if you are drunk, then you may fail to toss a coin, whilst the second one is to describe the behaviour of coin tossing itself as before.

Notice now we are using the effectful operations `choose()` and `fail()` at the same time. Therefore the effect signature of `drunkToss` also says that now, both effect types of `Choose` and `Exn` could be appearing in the computation of `drunkToss()`.

### Effect Handlers

The detailed semantics of algebraic effect operations are actually given by the effect handlers, explaining "how are we going to **handle** those effects". They define what happens when an operation is performed.

---

Consider the `maybeFail` being the effect handler for the `Exn` effect.

```ocaml
maybeFail = {
    fail |-> \forall x k . Nothing
    return |-> \forall x . Just x
}

(* some examples *)
handle maybeFail 42 ---> Just 42
handle maybeFail (fail()) ---> Nothing
```

In `maybeFail`, it defines what happens when `fail` is performed: giving `Nothing`. I

t should also contains a case for `return` when the computation does not contain the effect and effectful stuff and then just returns normally (giving `Just x`).

Here, `maybeFail` turn the result of such computation procedure, roughly speaking, into a `Maybe` datatype.

In the case of handling with `fail`, `maybeFail` turns the result into a function that simply accepts two arguments, where

- the first argument `x` in the code is the argument to the operation, in this case `unit`, as `fail` only takes unit type.
  
- the second argument `k`, which is the **continuation** captured from where the operation is performed that the handler is used to perform this operation. In this simple case, `maybeFail` does not make use of such **continuation** and just aborts it and return `Nothing`.
  
In the case of coping the `return` operation, `maybeFail` just captures the return value and captures it inside a `Just`.

Have a look at the examples in the code. This is what the operational semantics of the handler will give us.

---

`maybeFail` is a handler for `Exn`. And similarly we can define a handler for `Choose`. Consider `trueChoice`:

```ocaml
trueChoice = {
    choose |-> \forall x k. k True
    return |-> \forall x . x
}

(* examples *)
handle trueChoice 42 ---> 42
handle trueChoice (toss ()) -> Heads
```

This handler basically says "you always return a **True** when you need to make a choice".

In the `choose` case, `x` is again an argument to the operation, in this case a unit, and `k` is the **continuation**. The whole `choose()` operation in this case will be interpreted by handler, saying that "let's continue the original computation via continuation `k`, with having the choice of `True`".

In the return clause case: just giving an identity function on the return value.

---

From the last example we see that it is possible to "pass on" the continuation `k`. Therefore, it is not difficult to come up with the idea that:

An effect handler may choose to resume the original computation via `k` for multiple times.

Here let's give an example that is not considered to be a "linear" handler.

```ocaml
allChoices = {
    choose |-> \forall x k . k True ++ k False
    return |-> \forall x . [x]
}

(* examples *)
handle allChoices 42 -> [42]
handle allChoices (toss()) -> [Heads, Tails]
```

Here we can see the continuation `k` is used twice with argument `True` and `False` respectively, and their results are concatenated. The `return` case here also wraps the return result value into a singleton list. This results in the second computation example of `handle allChoices (toss())` returning `[Heads, Tails]`.

---

Up to now we are all handling a single operation. Effect handlers could be composed together to handle multiple effects. Resuming from the definition of `drunkToss` above, we can use an example to illustrate handler composition.

```ocaml
handle allChoices
    (handle maybeFail drunkToss())
(*
    result: [Just Heads, Just Tails, Nothing]
*)
```

For detailed reasoning, use the operational semantics of effect handlers, which we will discuss later. For a simpler reasoning, `maybeFail` decides upon `fail()` and, based on the result in the two `choice()` function, could give different results of either `Just Heads` (1st `choose` = true, 2nd = true), `Just Tails` (1st = true, 2nd = false), or `Nothing` (1st = false). Therefore, the following `allChoices` handler will explore all possible choices and results of the `choose` function and concat the results together. If you actually expand the computation you will see the result given in the example.

This is also the starting point of **lexically scoped effect handlers**, where we try to remove the dependency of runtime stack (dynamic scope) for different handlers and try to analyze the results under a lexical scope.

---

Now if we swap the order we apply the handlers, differences then occur.

```ocaml
handle maybeFail
    (handle allChoices drunkToss())
(*
    result: Nothing
*)
```

This turns the computation result into (in a sense) a `Maybe` of a list. As long as a single `fail` operation is performed in the program, this `maybeFail` will turn the whole computation result into `Nothing`. For details refer to the operational semantics.

---

These examples show that effect handlers can be composed in a modular way (handler composition) and in composition we do not need to worry about the legality of using different orders (only concerned with detailed programs).

Hopefully in the next note I can write more about the operational semantics and different classifications (deep, shallow, sheep) etc.

## Bibliography

[1] Plotkin, G., & Pretnar, M. (2009, March). _Handlers of algebraic effects_. European Symposium on Programming (pp. 80-94). Berlin, Heidelberg: Springer Berlin Heidelberg.

[2] Lindley, S. (2022, July 11-15). _Effect Handler Oriented Programming_ [Invited Talk]. SPLV 2022, Edinburgh, Scotland, UK.

[3] Xie, N. (2024, May). _Algebraic Effect Handlers with Parallelizable Computations_ [Keynote]. Lambda Days 2024, Krakow, Poland.
