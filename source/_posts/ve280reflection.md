---
title: Class initialization in C++ and what a course on C++ did right
tags:
  - C++
  - Class Initialization
  - VE280
  - Programming Languages
categories:
  - [Tech, Programming Languages]
date: 2022-12-30 22:09:59
---

In C++, given the following definitions of classes `A`, `B` and `C`, each with a constructor and two members, 
what is the order of initialization of the members and constructor?

VE280 was (is?) a sophomore course in UMJI, that for all intents and purposes, is a course that teaches students programming in 
C++. Among the kind of contents being taught (and sometimes tested) are questions like the one before. These questions occuppies 
a weired position in the technical discourse: some people take pride in being able to answer these questions correct, prides 
themselves as "experts of C++", while others dismiss it as pedantic, archaic, call the aforementioned *language lawyers*, and 
argues that it has no place in a university lecture room. 

I **was** a language lawyer. Back in 2018 and 2019 I TAed the summer instance twice and wrote an extensive collection of 
[recitation notes](https://github.com/tripack45/VE280-Notes), which admittedly goes far beyond what's required to pass the course. 
They are more akin to reflections of mine on the subject [1]. Five years later on an totally unrelated occasion I went back and 
read what I wrote then and reflected on it. I found the truth (as always) lies in between the competing views. Furthermore, 
I think VE280 did something right.

```c++
class A {
  A() { ... }
  T1 a1;
  T2 a2;
}

class B : public B {
  B() { ... }
  T1 b1;
  T2 b2;
}

class C : public B {
  C() { ... }
  T1 c1;
  T2 c2;
}

C c();
```

<!-- More -->

# The puzzle

Let's start by revealing the solution to the puzzle. If you were to construct and object of class `C`, entities are constructed 
(invoked) in the following order: `a1`, `a2`, `A()`, `b1`, `b2`, `B()`, `c1`, `c2`, `C()`. Deconstruction is called in the reverse order [6]. 

Why this order? Well the technically correct answer is of course *because the C++ standard says so*. But one has to wonder 
whether this choice is in fact *arbitrary*, or at least if it *makes sense* [2]. Though not explicitly explained (as far as 
I can tell) but there exists a very compelling reason why it is designed as such, and it is exactly the thing that VE280 did right.

Class implements ADTs (Abstract Data Types [3]). Every ADT (at least almost ways) must maintain some sort of invariant. 
For example, a `vector<T>` class may maintain a buffer pointer `T* buf`, and two integers, one `l` for the buffer size and 
the other `n` for the number of element. A class invariant is a predicate between class members, that must remain true when
execution is outside (before and after) of the class methods. In this example, part of the invariant says `n <= l` (accounting 
for element width), and *`buf != nullptr` if `l >= 0`*. 

Class invariants are at the same time the correctness conditions of the program and the technical tool that enables encapsulation. 
Essentially the concept of a `vector` *is* a collection of data in some relation: in other words it is *well-formed* iff its members
inhabit some relation that is the invariant.

The class constructor is desired because it is in charge of setting up the initial invariants when the object is created. Notice 
that data members of a class may themselves also be ADTs, hence has their own set of invariants to maintain. Now for the 
constructor to do its job it must be able to assume that the members are *well-formed*, i.e. has their invariants setup, hence 
initialized. Therefore, members must be initialized before the constructor itself. It is demanded by the need to write *correct* 
code, or in other words, ones' need to argue that the program behaves correctly.

By the same token the base class has an invariant of its own to setup, and the derived class should be able to assume the 
well-formedness of the base class, therefore the base class must initialize fully before anything in the derived class happens. 

These gave us two rules of thumb [4]:

1) The base class initializes before the derived class, and that 
2) Within the same class, members initialize before constructor.

Combining these two gave us the order in the solution. 

# What the course did right

The argument that underpins the discussion above is that language features must be designed in a way that works for reasoning
about programs. They "*must*" not in the sense that it is forced by a theorem, but due to the fact that the first requirement 
of any program is that it is correct: i.e. it does the job the programmer set out to do. The programmer, must convince, often 
others, but at least themselves that the program does the job. 

This reasoning of programs is only tractable when it is modularized: i.e. you break down the arugument into what each sub-piece 
of program does then combine them. On this note it is then understandable why OOP, and especially subtyping (i.e. inheritence) 
became popular: because subtyping allows one to reason by refinement: the idea is that *any two objects implementing the same 
base interface are "equal" to the client*, therefore one can *refine* an existing implementation and reuse the same reasoning. 

What VE280 did right was putting the reasoning of programs as a dedicated topic. It talks about preconditions, postconditions 
and effects of functions. For ADTs, it explained what is a class invariant and what is the desired property of subtyping. 
Admittedly these notions are delivered in a crude manner and the homework certainly did not emphasis them nearly enough but 
the matter of the fact is that it exists. This is information that you are unlikely to come across in a "language book" 
(e.g. C++ Primer). Unfortunately majority of the audience seem to have missed the point (like I said, the delivery can be 
improved). 

But nevertheless it is amount sufficient information to afford us a fresh take on the language features are put together. 
As we have seen earlier language design are often "forced" if you put forth the explicit goal of designing an 
easier-to-reason-with language. In fact, imho this is why functional programs are "easier to program correctly". Correctness
statements in functional languages are often significantly simpler: you talk about values of expressions and nothing else, 
contrary to imperative programs, where you talk about values of pointers, ownership of the objects among other things [5].

The set of notes I prepared for VE280, in retrospect, did not seem to properly present the idea. In fact I fell into the same
fallacy that tripped many other: the focus was *technique* and *philosophies* or *patterns* of programming, instead of a 
throughout backbone of program reasoning. This is especially obvious in the first chapter where I talked about abstractions: 
the emphasis was on the "social" aspect of programming. I talked about code organization, ease of refactoring, etc. Now 
it is clear to me that these are all *consequences* of having a modular logical structure of the program.

# Footnotes

[1] Although it was my genuine intention to help students by providing information that organizes the topics taught into 
a narrative that is coherent, it is admittedly very much debatable whether it is a good idea, or even TAs should attempt a
move like this, and frankly I'm not ready to defend my position.

[2] This is also what books such as *C++ Prime* never tells you: the book discusses what is the language but almost never 
why it is designed as such. Rules of the language can seem very arbitrary and obscure unless the language designer motivates 
the design. That being said, *"what the language is"* is objective, while the *why* is subjective and speculative, which may 
not fit into the tone of the book.

[3] The notion of *ADT* lacks a precise definition. In fact attempting to define it formally in a way that is portable across
language seems very intractable. 

[4] These does not tell us the order of initialization between members. One could argue that they should be ''unordered'' because 
the invariant of one member should not depend on the other, but there exists situations that complicates it (mostly from the fact
that C++ works with pointers and references). Both argument has merits. The standard made an executive decision to enforce 
initialization according to the order of occurence in declaration, which allows a larger class programs to have defined behavior.

[5] It is possible to write imperative language in a fashion that is very much functional: design your program in a way that 
correctness conditions of programs relies on values as much as possible, instead of layout of things in the memory.

[6] Original writeup contains significant errors. Kudos to [Peiyuan Qi](https://peiyuanqi.me/) for fixes.