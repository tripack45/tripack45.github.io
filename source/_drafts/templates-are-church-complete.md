---
title: C++ templates are church complete (without recurive templates*)
date: 2021-09-30 19:38:00
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

It is a well known fact that `templates` in C++ are Turing complete: in theory, you can compute anything you want to compute at compile time. And indeed people have developed elaborate libraries for this type of programming, dubbed *template metaprogramming*. But why is it even possible in the first place? Templates, a feature that is designed for generics, somehow gained full blown programming status. What is the minimum amount language features that enables this? 

This post shows that the template system is fundamentally turing complete with just template instantiations and member access alone. It is possible to embed untyped lambda calculus into the system using template instantion and member access. I support this claim by showing how using encode booleans, branches and Z-combinator (a variant of the *Y-Combinator*) using just those two features, discuss why this is possible from a type theory standpoint, and finally what it means for us.

<!-- More -->

# Introduction

In the appendix I present a program that computes factorial of 6 in two ways. 

```c++
int main() {
    std::cout << Fact::V<N<6>>::x << std::endl;
    std::cout << ZFact::V<N<6>>::x << std::endl;
    return 0;
}
```

`Fact` computes the factorial by direct construction of recursion, `ZFact` obtains recursion through a fixed point function (a function that turns non-recursive functions into recursive ones). We will break down the construction in later parts of this post. 

Before that, let me first explain the title of this post. 

We all familiar with *Turing Completeness*: a language is Turing complete if we can use it emulate any turing machine of our choice, hence it is computationally as powerful as any "real" language out there. Up until this day people have found numerous seemingly weak systems that turns out to be Turing complete: well known examples includes [Conway's Game of Life](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.386.7806&rep=rep1&type=pdf), the [x86 MOV instruction](https://github.com/xoreaxeaxeax/movfuscator) and Minecraft's redstone systems, less expected ones includes [Powerpoint Slides](https://www.andrew.cmu.edu/user/twildenh/PowerPointTM/Paper.pdf) and C's preprocessor. Of course C++ templates is also in this list. [Here](https://rtraba.files.wordpress.com/2015/05/cppturing.pdf) is how you can encode any turing machine in C++ templates. 

Now the Church-Turing thesis says there exists an equally powerful abstraction, namely the *(untyped) lambda calculus*. Turing proved in his work that the lambda calculus and turing machines can encode each other, hence they are eqaully powerful. 

# Appendix

```c++
#include <type_traits>
#include <iostream>

struct Id {
    template <class X> using V = X;
};

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