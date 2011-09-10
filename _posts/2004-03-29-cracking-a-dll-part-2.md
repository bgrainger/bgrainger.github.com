---
layout: post
title: Cracking a DLL, Part 2
---

## Introduction

In [Part 1](../26/cracking-a-dll-part-1.html), I explained the motivation
for cracking X.dll, and gave a brief look at what a bottom-up investigation of the DLL revealed. For a variety
of reasons, that approach ultimately ended in failure. I tried again, this time with a top-down approach. (I also tried harder.)

## Top-down Approach

### Finding Differences

I decided to run the code twice, once with the current date, and again with the system time
set to a month in the future. Any time I reached an "if" statement in the code, I would note whether the code took
the if branch or the else branch. (Technically, I would note whether the program jumped to a different instruction
or not each time it reached a conditional jump instruction. If statements are coded using the x86
conditional jump instructions: `ja` (jump if above), `jae` (jump if above or equal), `je` (jump if equal), `jne` (jump
if not equal), etc. The "conditional" part of the jump tests the processor flags, which are set by a previous instruction
(e.g., `cmp` (to compare two numbers), or `test` (to test a bit in a number)).)

The first time through, I would step through the `CreateData` function in X.dll, and not step into
any functions it called. I would note the return value of each function, and if they returned something different, I'd run the
program again (with both dates), stepping into that function to see where the execution differed that time.

### Set Up
Firstly, I disassembled the entire DLL by running `dumpbin /disasm:bytes X.dll > X.asm`. This
provided the assembly language instructions along with the equivalent machine code bytes. That would let me tie the
disassembly I was seeing in the debugger back to the actual bytes of the DLL if I ever needed to make any modifications.

### Debugging CreateData
I fired up the program and stepped into the `CreateData` function. This immediately dumped me into
disassembly view, since no source code was available. I selected the instructions (from the current location to the first
`ret` (return from function) statement) in the debugger window and copied it into
[TextPad](http://www.textpad.com/). As I recognised standard assembly language conventions (set
up function call frame, allocate local variables, call functions with parameters, etc.), I annotated the source code with what I thought
it was doing. I also annotated the source with the code path that was taken.

Very quickly I found a difference in the behaviour of the program with the different dates. With a valid run date,
the function at offset `0x1C6B4` (hereinafter called `Init`) would return `0x00D40BA8`. In the
future, this function would return 0. This value was stored in memory (in a global variable). The next function that was
called would take a while to execute with a valid date (probably loading data from disk), but would throw an exception
when called with a future date. The `Init` function obviously deserved more scrutiny.

### Debugging Init
As with `CreateData`, I copied the source code for `Init` to TextPad and started annotating it as I stepped
through. This time I found a point where the program would not follow a `jae` instruction when running with a valid date,
but would jump when running with a date in the future:

{% highlight nasm %}
call    20008E88     ; Function calls of mystery
call    2000289C     ;
call    2000530C     ;
cmp     edx, 0       ; First comparison -- edx=0 before and after bomb-date
jne     2001C7C0
cmp     eax, 1087h   ; eax=0x1085 before bomb date, eax=0x1089 after bomb date
jae     2001C7E6     ; jumps after bomb date, doesn't jump before
jmp     2001C7C2     ; program always jumps here (before bomb date)
{% endhighlight %}

The condition being tested was whether `eax` (an Intel x86 register, i.e.,
a variable) was greater than or equal to 4231. There were three function calls in a row immediately before this test; I
decided to examine each in sequence.

When I stepped into the first function, I realised that I had seen this function before: it was the function that
called `GetLocalTime` and validated the date, returning a floating point number. The second function that `Init`
called was simple to figure out; it just converted that floating point number to an integer. (I had become confused
about what was happening where with my bottom-up approach the day before; I now saw that these were two
separate functions.) The third function was complicated and had lots of if/else statements, but the end result was
that it divided one 64-bit integer (the number of days) by another 64-bit integer (the hardcoded value 9) and
returned a 64-bit integer.

The `cmp` tests following those function calls were now simple to figure out. The first checked that the
result of this division was less than 2^32 (specifically, that the upper 32 bits of the result were 0). The second checked
whether the lower 32 bits of the result were less than 4231. In short, X was checking that fewer than 4231 9-day periods
had elapsed since 1 Jan 1900. (It appears that the programmer was trying to be slightly sneaky here. By dividing
by 9, he changed the hardcoded value that he was checking against (4231) to something one might not be expecting.
Indeed, at first look, I had no idea what 4231 referred to.) If the test succeeded, `Init` loaded the magic number
`0x00D40BA8` out of memory and returned it, otherwise it returned 0.

### Changing the DLL

The hardest part was deciding how to change the DLL. Do I extend the number of 9-day periods that are allowed?
Should I skip over the whole date check, and just return the magic number? Perhaps just change the test so that
it always jumps to the correct instruction, whether or not eax &lt; 4231?
Adding or modifying any jump instructions would require me to pull out a hex calculator and perform some subtraction,
because the instruction to which you jump is specified as an offset from the current instruction (to save bytes). This
sounded like hard work, so instead I decided to just comment out the whole `jae` instruction. I did this
by overwriting the appropriate bytes in the file with `0x90`, the opcode for `nop` (no operation; do nothing).
The original source code looked like this (my annotations added after semicolons):

{% highlight nasm %}
push    0            ; use the number 9 as the hardcoded parameter to DivInt64
push    9
call    20008E88     ; call GetCurrentTimeAsDouble
call    2000289C     ; call ConvertDoubleToInt64
call    2000530C     ; call DivInt64 (divide one 64-bit integer by another)
cmp     edx, 0       ; make sure upper 32 bits of quotient are 0
jne     2001C7C0
cmp     eax, 1087h   ; is quotient < 4231?
jae     2001C7E6     ; greater than or equal, jump to BadDate
jmp     2001C7C2     ; must be less than, jump to GoodDate
{% endhighlight %}

I edited the DLL in a binary editor, replacing the opcodes for the `cmp` and `jae` instructions
(seven bytes total) with `0x90`. This effectively comments out the comparison aginst 4231 and the jump if the value
is greater, letting the program fall through to the unconditional `jmp` at address `2001C7BE`:

{% highlight nasm %}
push    0            ; use the number 9 as the hardcoded parameter to DivInt64
push    9
call    20008E88     ; call GetCurrentTimeAsDouble
call    2000289C     ; call ConvertDoubleToInt64
call    2000530C     ; call DivInt64 (divide one 64-bit integer by another)
cmp     edx, 0       ; make sure upper 32 bits of quotient are 0
jne     2001C7C0
nop                  ; do nothing
nop
nop
nop
nop
nop
nop
jmp     2001C7C2     ; always jump to GoodDate
{% endhighlight %}

It's not the most elegant fix in the world; I certainly changed much more than I needed
to. A good one byte fix would have been to change the comparison against `0x1087` to `0x7f001087` (a date
centuries in the future). But why be satisfied with changing one whole byte? We could change the number to `0x40001087`,
changing just one bit. I'll try even harder next time.
