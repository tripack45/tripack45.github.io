---
title: C++ templates are Church complete (without recurive templates)
date: 2024-11-27 15:40:00
tags:
- C++
- Templates
- Metaprogramming
- Recursive types
- Lambda Calculus
- Type Theory
category:
- [Tech, Programming]
---

It is a well known fact that `templates` in C++ are Turing complete. In this post I demonstrate that templates are *Church complete*, in the sense that you can encode (untyped) lambda calculus in them. 

<!-- More -->

# Introduction

It is well known that lambda calculus and Turing machines are "equally expressive" in the computability sense. So this is not surprising. What is surprising is the amount of feature the encoding exercised to make the language Turing complete. Usually to
encode a Turing Machine one relies on various methods of (1) template specialization 
(2) [SFINAE](https://godbolt.org/z/abxhKMshc), which implicitly uses function overloading resolution mechanism, and 
(3) recursive templates. 
Therefore, the consensus on the internet seems to be that the Turing Completeness of C++ Template system 
is an accidental result of mixing wide range of language features. 

This post seeks to prove otherwise. I argue that it is the fundamental design of the template language that renders this system 
Turing-complete. The code I am about to show you relies on **only template definitions and (static) class (template) member access notations**. 
In particular, it does not use recursive templates. This is important because programming language experts recognize that any 
use of general recursion tends to lead to Turing completeness. The lack of any apparent recursiveness in the definitions makes this 
result ever more surprising. 

# Templates as ML-`functor`s

What happened here is that we recognize template as a dynamic version of the ML module language, i.e. modules (functors) without 
signature ascription. A `class` in C++ can be viewed as an ML-module. A class `C` can be turned into an ML module in the following sense (I'm going to use *members* and *components* interchangably):

* `C`'s static members become the module's value components.
* The module it contains abstract type of the same name `type C`.
* `C`'s methods are function values of the module, accepting an extra argument of `C` (or `C ref`).

A template therefore is `functor`, mapping modules (classes) to modules (classes). In C++, classes are allowed to have templates 
as members. This corresponds to allowing functor members in a module. 

In the ML module system, the user must declare the content of a module using a `signature`. It is especially important for non-value components of the module, if `M` is a *module* (or functor) member of the another module, then it most be known, at 
the time of definition, what the contents of `M` is. 

It has been shown that ML modules can be characterized by various forms of dependently-typed calculus, c.f. [1-ML](https://people.mpi-sws.org/~rossberg/1ml/), [F-ing Modules](https://people.mpi-sws.org/~rossberg/f-ing/), and recent work to [unify modules under *phase distinctions*](https://arxiv.org/abs/2010.08599). A common understanding in all these work is that the (static) signature mechanism is critical to ensuring the consistency and decidability of type checking of the system.

# Template is a dynamic language

A key distinction, however, is that C++ templates does not require the user to declare the contents of a class. Because of this, we can package a template into a class, and send it as an argument to itself.  To wit, the following class `C` packages up a template `C::V` (reminder that `struct`s in C++ are just classes, but all members are public).

```c++
struct C {
    template <class Y> using V = ...;
}
```

The move is reminiscent of constructing values for the *recursive type* `mu(X. X -> X)`. It is known that untyped lambda-calculus
can be [recasted as a uni-typed language with this recursive type as the only type](https://www.khoury.northeastern.edu/~cmartens/Courses/7400-f24/pfpl/21-untyped-lambda-calculus.pdf). If the template langauge is well-typed (and does not contain unrestricted recursive types), it would not be well-founded.

To get non-termination, we simply pick a definition that "calls" `Y` using itself 
(`typename Y` is just `Y`, `template V` is just `V`, they exist to satisfy obscure syntax rules of the lanauge):

```c++
/* C := \x. (x x) */
struct C { template <class Y> using V = typename Y::template V<Y>; };
/* Omega := (C C) */
struct Omega { using V = C::V<C>; };
```

There you go, two lines of code using minimal feature to get non-termination: 

```
<source>: In substitution of 'template<class Y> using C::V = typename Y::V<Y> [with Y = C]':
<source>:4:30:   recursively required by substitution of 'template<class Y> using C::V = typename Y::V<Y> [with Y = C]'
    4 |     template <class Y> using V = typename Y::template V<Y> ;
      |                              ^
<source>:4:30:   required by substitution of 'template<class Y> using C::V = typename Y::V<Y> [with Y = C]'
<source>:8:21:   required from here
    8 |     using V = C::V<C>;
      |                     ^
<source>:4:30: fatal error: template instantiation depth exceeds maximum of 900 (use '-ftemplate-depth=' to increase the maximum)
    4 |     template <class Y> using V = typename Y::template V<Y> ;
      |                              ^
compilation terminated.
```

This shows that the Turing-completeness (therefore undecidability of type checking) of the C++ template langauge is deeply rooted
in the choice that template does not require signatures on classes.

# The code

Code in this section is available on [Compiler Explorer](https://godbolt.org/z/abxhKMshc).

In this section I present a program that computes factorial of 6 using the method laid out in two ways, `Fact` and `ZFact`.

```c++
int main() {
    std::cout << Fact::V<N<6>>::x << std::endl;
    std::cout << ZFact::V<N<6>>::x << std::endl;
    return 0;
}
```

You may check the assembly generated to verify that the result is indeed computed at compile-time.

The differences between these are as follows. The class `Fact` applies the recursion trick above to obtain a 
recursive-definition free factorial. `ZFact` on the other hand, pushes this further
and defines factorial through a [Z-combinator](https://en.wikipedia.org/wiki/Fixed-point_combinator). What the 
combinator does is to *factor-out* the recursive structure, so any recursive function can be obtained by 
writing down a prototype (in this instance, `FactProto`) then appeal to the combinator. It is a known lambda-calculus move. 

There are a number of supporting code. Classes `True`, `False` and `If` implements branching through [Church Encodings](https://en.wikipedia.org/wiki/Church_encoding).
In principle, I could do the same for the numerals (classes `N`, `Mul`, `IsZero` and `Pred`), 
but we could very likely run out of template instantiation depths. Compilation time could also explode. 
Therefore, I relied on integers (and `std::conditional`). I think the fact that you can mix and match different data-types 
like for the same member `V` further emphasize the resemblance between the template language and a dynamically typed language.

With all said, there's the code.

```c++
#include <type_traits>
#include <iostream>

struct True {
    template <class X> struct H {
        template <class Y> using V = X;
    };
    template <class X> using V = H<X>;
};

struct False {
    template <class X> struct H {
        template <class Y> using V = Y;
    };
    template <class X> using V = H<X>;
};

struct If {
    template <class B> struct H1 {
        template <class T> struct H2 {
            template <class F> using V =
                typename B::template V<T>::template V<F>;
        };
        template <class F> using V = H2<F>;
    };
    template <class B> using V = H1<B>;
};

/* Integer related convenience constructs */
struct Z { static constexpr int x = 0; };
struct S { 
    template <class N> struct V {
        static constexpr int x = N::x + 1; 
    };
};

template <int n>
struct N {
    static constexpr int x = n;
};

struct Mul {
    template <class X> struct A0 {
        template <class Y> struct A1 {
            static constexpr int x = X::x * Y::x; 
        };
        template <class Y> using V = A1<Y>;
    };
    template <class X> using V = A0<X>;
};

struct IsZero {
    template <class N> using V = std::conditional_t<N::x==0, True, False>;
};

struct Pred {
    template <class N> struct V {
        static constexpr int x = N::x - 1;
    };
};

/* First implementation */ 

struct FactSelf {
    template <class Self> struct A0 {

        // Both if branches needs to be "lazy" by giving it a dummy argument
        struct TBr {
            template <class Dummy> using V = S::V<Z>; 
        };

        template <class N>
        struct FBr {
            template <class Dummy> using V = 
              typename Mul::V<N>::template V<typename Self::template V<Self>::template V<Pred::V<N>>>;
        };

        template <class N> using V =
            typename If::V< IsZero::V<N> >::template V<TBr>::template V< FBr<N> >::template V<Id>;
    };

    template <class Self> using V = A0<Self>;
};

using Fact = FactSelf::V<FactSelf>;

/* Second implementation through Z Combinator */

struct ZComb {
    /* \x . x x */
    struct Self {
        template <class X> using V = typename X::template V<X>;
    };
    
    /* \x. f (\v. x x v) */
    template <class F>
    struct FSelf {

        /* \u. (x x) u */
        template <class X> struct H {
            template <class U> using V = typename X::template V<X>::template V<U>;
        };
        
        /*\x. f (\u. (x x) u) */
        template <class X> using V = typename F::template V< H<X> > ;
    };

    /* (\x. x x)(\x. f (\u. x x u)) */
    template <class F> using V = Self::V<FSelf<F>>;
};

struct FactProto {
    template <class Self> struct A0 {

        // Both if branches needs to be "lazy" by giving it a dummy argument
        struct TBr {
            template <class Dummy> using V = S::V<Z>; 
        };

        template <class N>
        struct FBr {
            template <class Dummy> using V = 
              typename Mul::V<N>::template V<typename Self::template V<Pred::V<N>>>;
        };

        template <class N> using V =
            typename If::V< IsZero::V<N> >::template V<TBr>::template V< FBr<N> >::template V<Id>;
    };

    template <class Self> using V = A0<Self>;
};

using ZFact = ZComb::V<FactProto>;

int main() {
    std::cout << Fact::V<N<6>>::x << std::endl;
    std::cout << ZFact::V<N<6>>::x << std::endl;
    return 0;
}
```