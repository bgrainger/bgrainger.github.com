---
layout: post
title: Profiling JScript, Part 0
---

As is my wont, I recently started looking at JScript.dll disassembly again and wondering about the possibility of implementing a JScript
profiler. It seems that with XP SP2 I now have symbols for JScript.dll on my system, which makes it a lot easier to understand the code.
(Symbols provide function names and parameter lists for those functions.) While reading the assembly listings, I noticed that
most functions look like this:

{% highlight nasm %}
mov     edi,edi
push    ebp
mov     ebp,esp
   ...
ret
nop
nop
nop
nop
nop
{% endhighlight %}

The second and third lines are standard x86 function prologue (setting up the stack), but I couldn't understand the apparently-useless `mov edi, edi` (or `edi = edi` in C++) instruction at the begining
of every function. There's also a five-byte gap between functions (that seems to be always five bytes, and not a variable
number, e.g., to force functions to align on 16-byte boundaries). A quick Google search turned up [Ishai's](http://blogs.msdn.com/ishai/) blog post
"[Why does the compiler generate a MOV EDI, EDI instruction at the beginning of functions?](http://blogs.msdn.com/ishai/archive/2004/06/24/165143.aspx)".
In short, this extraneous two-byte instruction (and the five bytes of `nop` (no operation, i.e., do nothing) at the end)
are inserted to enable hot-patching of the application. The five `nop` bytes can be replaced with a "long jump"
instruction (the `jmp` opcode, and a 32-bit address to jump to), while the two-byte `mov edi, edi` instruction can
be replaced with a "short jump" instruction (the `jmp` opcode, and an 8-bit offset) that jumps to the long jump.
But why use `mov edi, edi` instead of two `nop` instructions? If you are patching the application
while it's running, it's theoretically possible that one thread could have executed the first `nop` but not the second. If your
patch thread then changes both bytes (to the short jump), the thread that's about to read the second `nop` instruction
will instead read the offset your patch just inserted, and attempt to interpret that offset as a valid x86 opcode. Bad things would
almost certainly happen. `mov edi, edi` takes two bytes, and so it's impossible for the instruction pointer to be
pointing at the second byte.
This should enable us to hook certain functions in JScript.dll very easily. Since JScript.dll is loaded into our process's address
space, we can change the bytes making up its code easily. We first get the address of our profiling function and overwrite the five `nop` bytes
with a jump to that address. We then change the `mov edi, edi` instruction to jump to our long jump. Now, whenever
that function is called, execution will quickly pass to our profiling function. We just have to make sure we return to the right
instruction with the stack <i>exactly</i> as we found it. How hard could that be?
