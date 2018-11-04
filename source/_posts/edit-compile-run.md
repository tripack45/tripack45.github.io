---
title: The edit-compile-run loop and programmer happiness
date: 2018-11-03 16:52:50
tags:
  Dev Experience
categories:
- [Tech, Programming]
---

![The solution is Test-Driven Development! Or is it?](/images/tdd.jpeg)

Source of image is [here](https://www.icemobile.com/test-driven-development)

<!-- More -->

# Background

Last year when I was teaching VE482 as an TA, we had this idea to include a seperate lab for file sytems. The idea is detatialed as follows:

- Provide with each student (group) an image of EXT2 filesystem.
- Tell students there is a file that has been deleted from the filesystem.
- The file system is broken (in that some meta data is removed, so that they can't simply mount it and recover the file using existing tools).
- The name of the deleted file is provided, yet the path is not known.
- Students are asked to recover the file, fix the FS, and mount it.

There are two ways to do this, 

- Use what is known as `simplefs`, which is kernel module that implements a very simple FS. Start from there and work based on that.
- Use *FUSE project*, hich standards for *Filesystem in User Space*. It's a piece of program that sends kernel space calls into userspace. The FS author intercepts such calls at userspace (by linking the handling code with `libfs`). 

We were not able to make it happen last year because of limited time. This year the instructor looked into the first option, and we had a discussion on the second option. The instructor found that the original code `simplefs` does not immediately compile (because it's out dated), yet it can be fixed with a simple fix. On the other hand, it seems to be a lot of trouble going through the docs for `FUSE` APIs, as these APIs are not that straight forward.

The goal is to design a lab work that requires a reasonable amount of workload. We had a few email correspondence a which might be an `simpler` solution for students.

# Mythical *faster* development

Now the instresting thing here is that what is precieved as `simpler` might not be the actual less time consuming one. For instance, *dynamic languages* such as Python and Javascript is getting much more porpularity over the years. One preception of those languages is often developing in those languages are *faster*. Now why developing in those languages are `faster`, or is it indeed `faster`? One of the major reason is usually better library support. Availablity of already written code without doubt will cut down develop time. Besides that, is there anything special about the language itself that makes it *faster*? 

Well, the biggest trait of such dynamic languages are their *dynamicity*, specifically, the fact that they are dynamically typed (along with runtime reflections and inspections). But that doesn't quite make sense, because according to existing research (for starter, [this one from Cambridge Univ](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.370.9611&rep=rep1&type=pdf)) programmer spent almost half thier development time on debugging. 

I'm not sure if it the case with others. Last year I wrote an interpreter for a Java-ish language in Scheme. Scheme is a Lisp-like program, specifically, is not statically typed. This gives me nightmare, because I spent days after days dubbuging errors that could be avoided if the language is typed. Examples are

- Typos that results in references to unknown symbols (or worse, to a different, but existing symbols).
- Forgotting to call wrapping functions. (And error progogates and an runtime exception is thrown far away).
- Logic errors. Some ideas simply won't work as long as you write out the type, yet it took me hours to realize that after lots of debugging.
The lesson here is really dynamically typed languages significantly prolong the debugging process, because you often let more subtle errors, that could be caught statically become runtime errors. This is similar for Python. I'd admit that Python is somewhat better than scheme in combatting these situations because the native support for many common datastructures does not allows the errors to propogate by too far, yet the same problem will exhibit themselves when programs get larger. 

Still people believe that thoses dynamically typed languages are *faster*. This is the story of the mythically *faster* dynamically typed languages. (See side notes)

# The edit-compile-run cycle

What I believe, makes the dynamically typed languages feels `faster`, is the shorter `edit-compile-run` loop. Some [authors](https://mlafeldt.github.io/blog/fast-feedback-is-everything/) have wrote about it. Basically human brains are driven by stimulus, and people are most stimulated when they see things poping onto the screen. That is exactly the kind of satisfactory you get when your program runs. Clearly witness the programs running provides more stimulus than having a compile error. This is why many would still prefer a language that gives you a runtime error over one that gives a compile error, even if logically the former indeed makes your life harder.

Back to the FUSE issue. The biggest advantage of using FUSE is the short “edit-compile-run” loop. The thing with kernel programming is that the iteration cycle is very long. A typical cycle (on a real machine) will be *edit-compile-compile\_error -edit-compile-run-crash-reboot-find\_file-edit-...*, and compilation is very slow (e.g. minix takes around 20 minutes to compile), and reboot and re-edit is even more slow. This doesn’t get too much better if you use a virtual machine, because although reboot will be faster, and you don’t need to re-locate your source file after reboot, you do have an extra step to sync your file into VM.

Now FUSE solves this because now you focus on one file, and you tweek the FS just like tweeking any other C program they wrote in other courses. I figured that might be better accepted by students. Because every FUSE service is just a usual process (communicating with the in-kernel part of `libfuse` through IPCs), you can minize the period of the cycle.

Another thing to be noted is that this effect of edit-compile-run cycle is prevasive. 

- When I was doing the project on Minix kernel scheduler, the major execitement bummer is the 20 minutes compile time. Whenever I change something, I have to wait 20 minutes to for the OS to compile (or to receive a compile error). This basically discourages me from making any further changes as soon as I finished what is required by the assignment. So as you can see, for me the very long cycle is real productivity blocker. (I did, at the begginning of the projects, tries to do cross compiling, but unfortunately it never worked out for me.). 

- Novice students like IDEs more over command line, because that click-build-run button is far more faster than *terminal-type-error-type-error-type*. As someone who just arrived at the world of programming, your urge is to get your code running (receiving feedback on screen) as soon as possible. You hate reading documents, you hate having to copy/paste some mysterious "compile" commands each time you make changes (and you certainly dislike typing the commands yourself  when you are not shell-proficient). This is why IDEs are uniquely attractive, because they allows you minimize the cycle by doing all these jobs for you. And it is better, because it is able to provide feedback even when you haven't runned the program yet (syntax highlighting, and more importantly semantic analysis that catches errors on the fly). Now as you become more proficient, you started to have your own ways of doing things. This is when you shift to command lines and terminals, since there is by no means an IDE an provide you with that kind of flexibility that scripting do. To minimize the cycle further, you write make files, you introduce version controlling and all kinds of automations. 

- Similar things happen with the programs that first year students wrote. New programmers often starts with programs that involves a lot of `print`s. Examples are 
  * program that "calculators" that prompts the user for two number and an operation, and preforms that operation and returns the result. 
  * Programs that does unit conversions.
  * Programs that calculate interests of savings.
  * Ohh this one is classics: a program that prints the *Pascal Triangle*.
  * Another classic example: a program that prints a heart.
  * An question and answer games.
  * And games. Many programmers's first project is to write some sort of a game, that ideally involves some GUI. Altough very few of them actually finishes it. 
  * And a lot of struggling with how to properly perform IOs (e.g. how to write format strings in ``scanf`s and `printf`s, how to flush the `stdin`)
  * I guess this is also why *Hello world*s are so popular as the first program of any new language introduced.

This list goes on and on. Once you really learned programming, you will one day realize that thoses programs are perhaps among the most uniteresting ones (except the game one, but novice programmer rarely realize the real fun part of interactive programs, which inevitable requires understanding of concurrency and asynchronous programming). The logic is straight forward, and there is really nothing valuable learned from the experience (even worse, sometimes you get used to really shitty designs, e.e. C++ stream IOs, and you will in the future pay a price getting out of those habbits). 

# Two ways out

There are in general two ways of dealing with *the shorter the "edit-compile-run" cycle, the happier the the programmer* observation.

## Working with the cycle

The first one is find a way wot work with it. The idea is to accept the obervation as a fact about how humans works, and design our process around this. 

An example that goes against the principle is often the advertisement of advanced editor users (think about those ethusiatic Vim/Emcas/VS Code/Atom users who keeps telling you how greater their editor of choice is, and you should make the same choice). As I have noted before, people tends to choose the tools that minimized cycle. You only look for better editing facilities when the editting time becomes your blocker of further reducing that cycle. This is exactly what happend to me myself. I started off using *spacemacs* when I finally cannot put up with all those keystrokes spent on moving around character by charcter. Before that, the major blocker is really me being infamiliar with all the tool chains. So instead of constantly talk people into using Emacs/Vim, ask them what is currently stopping from minimizing the cycle!

Another thing: We should really build tools that reduces such cycle. If we can get programmers to iterate faster, programmers will be able to iterate more (willingly) and finally deliver better product. A language that typically does not very well is C++, which requires you to manually manage dependencies (and linking them, which is another nightmare), and need you to type long building commands. A language that does that really well is Golang. Personally I don't like the whole `gopath` design (because it forces you to group programs by language, not projects). But it does allow you to refer to dependencies much easier. This furhter enables very easy to use `go test` and `go fmt` commands, that reduces cycle by freeing the programmer from typing paths. Judging by the popularity of Go, I think this strategy really works.

## Test driven development

A third example is the attemp to combat the property of dynamically typed languagesnamely, *unit tests* and *test-driven development (TDD)*. The ideas are:

- Begin programming by first writing tests.
- Not only write tests that tests behavior of entire module, but more importantly tests that are designed for each small enough module (unit tests). 
- Write your program with the aim to pass all the tests.

This process is designed in a way that is perfect for dynamic languages. Unit tests are small so that you can start executing them with very little amount of coding (shorter cycle). And the fact that they are unit tests makes it easy for programmers to map runtime errors back to faults in program logic (less guessing because you are looking at a smaller code base). Finally each run you receive useful feed back (either you passed all tests in which case you are happy, or you fail one of the tests). You can rapidly iterate such process, and this keep you excited.

(Interesting) critics of TDD can be found in the side note below. 

## Fighting the cycle

The second of them it to fight it. That's more of the path I took. For instance, convince your self to first read the docs then start writing programs, and to pick a language that has a strong type systems that helps you to defend against runtime errors.

To fight a human instinct sounds like a hard-to-swallow pill, but that is often neccessary. As I have noted, programming is after an act of logical reasoning. And resonning cannot be done by simply testing. When you do proof of maths, do you:

- Write a random proof that makes reference to an non-existing theorem
- Submit to your TAs / teacher then gets rejected with a low grade
- Change a few things and submit again
- Iterate until your professor cannot point out an error (doesn't mean your proof is correct)

Or do you plan ahead and try to argue thoroughly and reasonably? Programming after all is really the same thing. You cannot, one way or another, in the end escape it, and lets just let the machine help as much as possible.

For those who are teaching programming, I think it’s also worth sharing these insights with students, because sometimes to become a good programmer, you need to learn to combat those human instincts. One example is to defy the seduction of “dynamically typed” languages, and favor those statically typed languages, even if you might spend 30mins debugging type mismatches in 50 lines of code you just wrote. And understanding why you hate writting things like the minix kernel projects helps to convince your self that “the problem is not hard, it’s just running it takes some patience”.

# Side notes:
## The dark side of TDD

Now there are many advocators of TDD, and I doubt whether this is always a good idea. Programming is after all a process of logical thinking. One of the long standing theorem in PL community is that *programs(types, actually) are just proofs*. In laymen's terms, good program have to be written in a way that reasoning about their correctness is easy, otherwise there is a fat chance that a bug exists. You can started off thinking hard and do the right thing the first time, or you can keep running your code and modifying it according to runtime errors (TDD way).In the end, you must reach the same thing: a logically sound program.

To make this TDD method work, you will need to break up the program into really small pieces, otherwise the curse of dynamic language sneaks up on you once again, and you have a thorough (dense, and logically written) test suite. This usually means more man power. Now watch:

- For a "traditional" way of programming to work, someone need to design and code the entire program (or his portion of the job) on his own. The person will need to have good logical thinking. 
- For TDD to work, you can 
   * Hire one "architect" who is really good at programming, ask the person to break the program (logically) into small parts, and describe what each parts do.
   * Hire a bunch of "median" "test engineers" and write detailed tests according to the specification.
   * Hire another bunch of "software development engineers", and write the actual code. 

This process will make sense for large companies. As you could imagine, "architects" are most hard to find, yet there are plenty of "test engineers" and SDEs are flooding the job market. Plus, this scheme scales out well. The architect does not need to write actual code, so he can handle more projects simutaneously. As the project scales, the company will need to hire many more SDEs, but only a handfule architects. 

Of course small companies cannot afford something like this. So does personal projects. I doubt if there is anything to gain from TDD for personal project, because by breaking the program appart, you already done the logical reasoning, which is teh hard part. 

TDD also doesn't work with critical softwares. Softwares like compilers, OS (and any security/crypto/embbeded related software) cannot be trusted "Without proper logical reasoning in advance. Just passing the test is far from sufficient to establish trust in those software. In the end TDD won't give any advantage (although it's almost always better to have more tests, but that's not a TDD-only thing). 

## Dynamic languages at large companies

Some may argue that Python is fine for large projects, because even large companies that run real large systems use it (for instance, Facebook is built on Php). But this claim does not quite hold because Facebook employs a large team of PL specialists that focuses on writting statically analysis tool to catch errors when code is being commited (sort of the compile time for dynamically typed languages). 

