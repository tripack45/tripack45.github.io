
title: "SIGBOVIK'19: Precise ECG Platform on Modern Processors"
date: 2019-04-02 14:04:50
tags:
- SigBovik
- Jokes
categories:
- [Tech, Fun]
---

[Sigbovik](http://sigbovik.org/2019/) (Special Interest Group on Harry Q. Bovik) is an annual Quasi-conference held on April 1st at CMU for mostly computer science students to have "geek fun". Papers submitted usually includes silly ideas with (often) serious execution. This year, I and [@codeworm96](https://github.com/codeworm96) submitted a paper titled [Precise ECG platform on Modern Processors](/blob/sigbovik19.pdf). We won the *Most Likely to Void a Warranty* award from the program comittee.

<!-- More -->

![SigBovik'19](/images/sigbovik19.jpeg)

Our work focuses on the question of how to fry an egg (physically) on modern processors without overheating both the egg and the processor. As a matter of fact, people have already tried to fry stuff on various hardware. For instance, people have tried to fry bacon on GTX480 GPUs. The problem is people have very few control over the power of the processor. We propose a technique to control the power output of processors by controlling it's utilization over time. 

The way we did it involves two components. We first develop a way to assert a specified load onto modern, multicore processors by putting processors to sleep randomly. The more often processors are put to sleep, the lower the utilization. Then we use a feedback control technique to compensate for environmental disturbance, since usually more than 1 program runs on a single machine.

The paper is written in a manner that makes it look awfully like an early draft for a serious research paper. We bascially invented a field "Edible Content Generation", which basically translates to "cooking". Implementation is written in Rust. This is, in fact, the very first Rust project for me. Thanks to the help from @codworm96. 

The names and contact information (except for the email addresses) are both jokes. *S. Normalized Inflook* means "Strongly Normalized Infloop", which is a PL joke (strong normalization informally means run to completion, infinite loops by definition will not complete). *Ivybridge N. Skylake* is three recent Intel architecture codenames, with *N.* stands for *Nehalem*. The departments are jokes on CMU's *Computer Science Department* and *Information Networking Institute*. The institution name is abbreviates to *TAITS*, which is a famous PL theorists. Name of the city is an interplay between Boston and Pittsburgh. The name of the country, *United State Monads of America*, is a Haskell joke.

A link to the paper is [here](/blob/sigbovik19.pdf). Many (around 3) say it is fun to read.

A link to the online version of conference proceeding is [here](http://sigbovik.org/2019/proceedings.pdf).
