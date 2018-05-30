---
title: Real Smart Compilers
date: 2018-04-30 12:38:00
tags:
- CMU 15513
- Bug analysis
category:
- [Tech, Programming]
---
This post contains solutions to specific problem used in CMU 15513. If you are taking the course, please do not read further.

In this post, I will introduce you to a bug of the program. At first I thought this is a hardware bug. Then I suspect this is a compiler bug. Yet it turns out, the real issue is undefined behavior in my program. We will see how compiler makes "wrong" optimizations, when undefined behavior is involved.

<!-- More -->

# The problem

## Lab setup

In the first lab of the course, the datalab, we are asked to solve a seriers puzzles on integers, under the following rule:

- Only bitwise manipulations are allowed. No branches, loops etc.
- Only arithemetic operations allowed is `+`. No `>`, `<` or `-`. 
- Only logic operator allowed is `!`
- Only `int` types are allowed.
- For each puzzle, only a given number of operations are allowed.
- You can only use integer constants within 0 to 255 (0x00 to 0xFF);

The function that I am trying to implement, is the following:

``` c
/*
 * bitCount - returns count of number of 1's in word
 *   Examples: bitCount(5) = 2, bitCount(7) = 3
 *   Legal ops: ! ~ & ^ | + << >>
 *   Max ops: 40
 *   Rating: 4
 */
```

## Solution
 
The solutions is quite elegent. If you count the bits one after another, very easily exeeds the number allowed operations. You have to do the counting "in parallel", taking advantage of the hardware. The idea is to count bits first in group of 2, then in group of 4, then 8, followed by 16 and 32. A 8-bit example is provided the following: suppose we need to count `1011 1010`. 
 
- You first group the number in pair, and count the number of 1s in each pair. 
    - Take the odd bits of the number `1011 1010 & 0101 0101 = 00 01 00 00`
    - Take the even bits `1011 1010 & 1010 1010 = 10 10 10 10`. Right shift by 1 `01 01 01 01`
    - Add previous results gives `01 10 01 01`. This is number of 1s in each pair. 
- Count the number of 1s in each group of 4.
    - Extracts odd pairs. `01 10 01 01 => 00 10 00 01`
    - Right shift by 2 then extract. ` 01 10 01 01 => 00 01 10 01 => 00 01 00 01`
    - Add together. `00 11 00 10`
- Do the same again with different shift. 
- Extract the lowest 16 bits. This gives the count. 

The code implementing the algorithm is provided as follows. The code is pretty cryptic. You can skip the code for now. We will refer to specific parts of the code later.

``` c
int bitCount(int x) {
  int mask2_half = 0x55 | (0x55 << 8);
  int mask2      = mask2_half | (mask2_half << 16);
  int count2_0   = x & mask2;
  int count2_1   = (x >> 1) & mask2;
  int count2     = count2_0 + count2_1;
  int mask4_half = 0x33 | (0x33 << 8);
  int mask4      = mask4_half | (mask4_half << 16);
  int count4_0   = count2 & mask4;
  int count4_1   = (count2 >> 2) & mask4;
  int count4     = count4_0 + count4_1;
  int mask8_half = 0x0f | (0x0f << 8);
  int mask8      = mask8_half | mask8_half << 16;
  int count8_0   = count4 & mask8;
  int count8_1   = (count4 >> 4) & mask8;
  int count8     = count8_0 + count8_1;
  int count16    = count8 + (count8 >> 8);
  int count32    = count16 + (count16 >> 16);

  printf("sizeof(int) = %d\n", sizeof(int));
  printf("%x, %x\n", mask2, count2);
  printf("count2 >> 2 = %x, mask4 = %x\n", count2 >> 2, mask4);
  printf("(count2 >> 2) & mask4 = %x, count4_1 = %x\n", (count2 >> 2) & mask4, count4_1);
  printf("%x, count4_0 = %x, count4_1 = %x, %x\n", mask4, count4_0, count4_1, count4);
  printf("%x, %x\n", mask8, count8);
  printf("%x, %x\n", count16, count32);

  return count32 & 0xff;
 ```

## The bug

The function fails when I call it with -1 with argrument. In 2s' complement representation, -1 becomes `0xffff ffff`. Clearly there are 32 bits in the number. Well, program output is the following:

```
 ❯ ./btest -f bitCount -1 -1
sizeof(int) = 4
55555555, aaaaaaaa
count2 >> 2 = 2aaaaaaa, mask4 = 33333333
(count2 >> 2) & mask4 = 2222222, count4_1 = 2222222
33333333, count4_0 = 22222222, count4_1 = 2222222, 24444444
f0f0f0f, 6080808
60e1010, 60e161e
ERROR: Test bitCount(-1[0xffffffff]) failed...
...Gives 30[0x1e]. Should be 32[0x20]
```

I know this is not a "bug" with my code. Now the university provides each student an Intel sponsered course server. The grading is done on that server. This same program executes successfully on the sever. 

Further more, the course work comes with a formal verification tool, based on binary decision diagrams, that formally proves the correctness of the program. Of course the code passes the checker.

# The analysis

## What happened?

We examine the output, and noticed the following:

- `count2` is `0xAAAA AAAA`. Shift of `count2` becomes `0x2AAA AAAA` 
- Masking `0x2AAA AAAA` with `0x3333 3333` gives `0x0222 2222` (value of `mask4_1`).

Essentially,

- `count2 >> 1` should do a **arithmetic** right shift, resulting `0xEAAA AAAA`. Here it does a logical shift.
-  `mask4_1` lose an `2` during the `&`ing. 

A few more comments:

- An arithematic right shift duplicates the sign bit. I.e. it divides the integer value by 4. A logical shift, on the other hand, always put 0 on most significant bit.
- If the masking is done right, the right shift and left shift should generate identical ouput. As long as the intermediate result is not required, it should be OK to use either.
- However since I am printing the intermediate result, it has to be a arithematic shift.

## The scope of the bug

I started off by suspecting this is a hardware bug. My local machine runs on I7-7567U CPU, running OS X. The bug is reproducible on a I7-7820HQ machine also running OS X. The cluster, on the other hand, runs on 2013 versions of Xeon CPUs. This could be a bug that is introduced in recent archs. 

I post this issue on to course Piazza and seek help from the instruction team. A reply from the Kevin Geng, one of the TAs is quoted as follows

> ... I'm personally a bit curious about this, ... Though I think the probability of it being a hardware bug is fairly low -- it's more likely to be a compiler bug, at best.

I started by looking into other configurations. By default the code is compiled by the following command:

``` bash
gcc -O1 -Wall -m32 -lm -o source.c
```

With a bit of the help from my friend Wchg, I have the following result:

- On my friends 2016 version Macbook Pro, the problem exists.
- On my local machine, Changing the optimization level from 1 to 0 solves the issue (`-O1` to `-O0`).
- On my friend's linux machine with Core i9 (I think), install latest version of `clang` and compile with it, the problem exhibits. 
- On all systems, compiling with `gcc` solves the problem.

So I guess Kevin is right. This might as well be a compiler issue. However the code only contain logical operators, it is somewhat astonishing that clang could get something that trivial wrong. That being said, there also exists a very tiny possibility that this indeed is a hardware bug. Perhaps somehow only the code sequence generated by clang triggers that bug.

## Disassembling the code

To get to the bottom of this, I disassembled the compiled binary. The following command extracts the assembly code of `bitCount()` function from the executable `btest`. `otool` is the equivalent of `objdump` on OS X, I believe.

``` bash
otool -tvV btest -p _bitCount
```

The disassembly is long indeed. We extract the relavent portion from it:

``` asm
btest:
(__TEXT,__text) section
_bitCount:
00002510	pushl	%ebp
00002511	movl	%esp, %ebp
00002513	pushl	%ebx
00002514	pushl	%edi
00002515	pushl	%esi
00002516	subl	$0x1c, %esp
00002519	movl	0x8(%ebp), %ebx
0000251c	movl	%ebx, %eax
0000251e	andl	$0x55555555, %eax
00002523	shrl	%ebx
00002525	andl	$0x55555555, %ebx
0000252b	addl	%eax, %ebx
0000252d	movl	%ebx, %eax
0000252f	andl	$0x33333333, %eax
00002534	movl	%eax, -0x28(%ebp)
00002537	movl	%ebx, %ecx
00002539	shrl	$0x2, %ecx
0000253c	movl	%ecx, -0x14(%ebp)
0000253f	andl	$0x13333333, %ecx
00002545	movl	%ecx, -0x18(%ebp)
...
```

Interesting things happens on address, `0x0000 253F` and `0x 0000 252F`. Both lines uses an integer counter to preform the masking. The compiler got the first integer correct, `0x3333 3333`. However the second integer is problematic. It is generated as `0x1333 3333`.

On the other hand, on line `0x0000 2539`, we see that a `shrl` (logical right shift) is generated, instead of an arithmetic shift. 

I checked the code compiled with -O0. The compiler generated arithmetic shifts.

All the evidence suggests that somehow the compiler generates wrong binary code. It looks much like a compiler bug.

# Root cause

## The reply

**The following content is adpated from an reply by CMU 15513 instruction team. The author is Kevin Geng.**

 I've been able to reduce the test case to the following:
 
```c
int bitCount(int x) {
  int mask2      = 0x55555555;
  int count2_0   = x & mask2;
  int count2_1   = (x >> 1) & mask2;
  int count2     = count2_0 + count2_1;

  printf("count2 = %08x\n", count2);
  printf("count2 >> 2 = %08x\n", count2 >> 2);

  return 0;
}
``` 

On my personal machine, running with `-O0` outputs `count2 >> 2 = eaaaaaaa` and running with `-O1` outputs `count2 >> 2 = 2aaaaaaa`. Obviously, this is wrong. ... it should still be defined as an arithmetic right-shift; certainly it wouldn't be reasonable to believe that the default for `clang` is to output a logical right-shift. And in general, `clang` does use an arithmetic right-shift; though while `gcc` [guarantees](https://gcc.gnu.org/onlinedocs/gcc/Integers-implementation.html) this behavior, I couldn't find any similar sources for `clang`. 

Compiling with clang's [UndefinedBehaviorSanitizer](https://clang.llvm.org/docs/UndefinedBehaviorSanitizer.html), though, reveals what the actual culprit is: the computation of `count2` results in a signed overflow, which is undefined behavior. This is the real reason why Clang is able to emit a logical right-shift. I think the reasoning goes something like this:

- Both `count2_0` and `count2_1` are the result of and-ing with the mask `0x55555555`.
- The mask removes sign bit. So both `count2_0` and `count2_1` must be positive.
- The compiler may assume undefined behavior will never occur. Then sum must be positive. 
- Since `count2` must be positive, logical and arithmetic right-shift are equivalent.
- Since `count2` is positive, the sign bit must be zero. Thus right shifting `count2` by 2 clears the most significant 3 bits.
- If the most significant 3 bits of `count2` are all zero, masking it with `0x3333 3333` or `0x1333 3333` is equivalent. The compiler may choose either constant. 

In conclusion, there is no bug of any sort. 

## Take away

I've always know that compiler are allowed to assume that undefined behaviors may not happen, and preform optimizations based on that assumption. A well known example, is that the compiler may optimize the following code into infinite loops:

```c
int main() {
    unsigned int i = 1u;
    while (i) {
        i++; 
        // Unsigned int overflowing is undefined behavior
        // Assuming no overflow
        // i increases monotically -> greater then zero
        // while condition always met -> infinite loop
    }
    return 0;
}
```

It just never occured to me that such "aggresive" optimization may happen with rather conservative optimizations. On the other hand, it seems like the `gcc` is either more conservative, i.e. obeying more rules, or "less clever", since it missed all these reasoning. 

The take ways are:

- Compilers are very clever. Complicated deduction can be made with little assumptions.
- Use `clang`'s UndefinedBehaviorSanitizer to check code validity, if weired things happens.