---
layout: post
title: Profiling JScript, Part 1
---

Finishing my rewrite of the Logos Bible Software Home Page wasn't sufficiently appealing, so I started looking at JScript.dll in more detail.
I decided to get a feel for the architecture of jscript.dll by examining the call stack for a simple property access on one of our COM objects.
Placing a breakpoint in `CLbxScriptUtil::get_Date` (the function invoked by `Application.ScriptUtil.Date` in JScript) gave the following call stack:

    ScriptUt.dll!CLbxScriptUtil::get_Date()  Line 158	C++
    oleaut32.dll!_DispCallFunc@32()  + 0xc3
    oleaut32.dll!CTypeInfo2::Invoke()  + 0x20c
    oleaut32.dll!CTypeInfo2::Invoke()  + 0xc4
    oleaut32.dll!CTypeInfo2::Invoke()  + 0xc4
    ScriptUt.dll!ATL::IDispatchImpl::Invoke()  Line 4412	C++
    jscript.dll!IDispatchInvoke()  + 0x6f
    jscript.dll!InvokeDispatch()  + 0x72
    jscript.dll!VAR::InvokeByName()  + 0x10c19
    jscript.dll!CScriptRuntime::Run()  + 0xcd5d
    jscript.dll!ScrFncObj::Call()  + 0x6a
    jscript.dll!JsEval()  + 0xe2
    ...

jscript.dll!IDispatchInvoke() looked like an interesting function. Getting its decorated name out of jscript.dll and running it through
[UnDecorateSymbolName](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/undecoratesymbolname.asp)
gave the following method signature: _(Note: parameter types and names cleaned up by referring to the IDispatch::Invoke() documentation)_

{% highlight c++ %}
long __stdcall IDispatchInvoke(
  class CSession * pSession,
  struct IDispatch * pDispatch,
  DISPID dispIdMember,
  REFIID riid,
  LCID lcid,
  WORD wFlags,
  DISPPARAMS * pDispParams
  VARIANT * pVarResult,
  EXCEPINFO * pExcepInfo,
  unsigned int  * puArgErr
)
{% endhighlight %}

Although I can't prove it, I'm going to assume that this is a helper function used by JScript internally whenever IDispatch::Invoke
needs to be called. A simple profiling exercise would be to log how many times this function is called. You can see that when it's
calling our code, it immediately invokes ATL::IDispatchImpl (in the common header file atlcom.h). We could log all calls to Invoke
by changing atlcom.h and recompiling almost every source file in our application. Or we could try to intercept the function at runtime
and log accesses that way. Not only is the latter method a lot quicker (no recompilation necessary), but it greatly increases the
chances of GPFs (since this would involve programatically changing the bytes of a running application). It seemed like the obvious choice.

I decided to write three functions: one to hook the IDispatchInvoke function, one to unhook it, and the hook function itself (which would
just increment a counter). For ease of testing, I'd invoke these functions from a JScript-accessible COM method. I chose to make
`ScriptUtil.Application` install the hook, `ScriptUtil.Parent` remove it, and `ScriptUtil.Date` log the current
counter value.

While a &ldquo;production&rdquo; version of this code would need to handle different base addresses for JScript, it's more expedient
to just hard-code the addresses of the code I want to change. The `nop` bytes that need to be changed to a long jump are at
`0x75C6672E`; the `mov edi, edi` that needs to be changed to a short jump is at `0x75C66733`. We also need
the address of our hook function (which is easy to get at runtime by using a function pointer). The fourth address needed is that of
the five `nop` bytes in our hook function that we'll overwrite with a jump back to the third byte of JScript's IDispatchInvoke function.

The blog post about `mov edi, edi` that I mentioned in the last installment had a cryptic reference to something called
&ldquo;Detours&rdquo;. A little googling showed that [Detours](http://research.microsoft.com/sn/detours/) is a Microsoft Research
project that enables interception of arbitrary functions at runtime. It seems quite sophisticated, but assumes that the function being
&ldquo;detoured&rdquo; doesn't have the `mov edi, edi` instruction at the beginning. As a result, Detours needs to copy the
first bytes of the function to a temporary location, since it overwrites code that's essential for the function's correct operation. Secondly,
while you can call the original function (through a &ldquo;trampoline&rdquo;), you need to supply all the original parameters (just like a
normal function call). While this makes sense conceptually, it could lead to subtle bugs if the calling convention or parameter types
weren't declared identically. I'm not currently using Detours, but did take a few ideas from their description of its implementation.

My implementation is fairly simple. I have a [`__declspec(naked)`](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/vclang/html/_pluslang_the_naked_attribute.asp) hook function that increments a global counter
and has five `nop` bytes that are reserved for replacement with a long jump. A C++ helper class, `CEnablePageWrite`,
calls [FlushInstructionCache](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/debug/base/flushinstructioncache.asp)
and [VirtualProtect](http://msdn.microsoft.com/library/default.asp?url=/library/en-us/memory/base/virtualprotect.asp) (in
its constructor) to enable writing to the page containing the code to be modified; the destructor restores the original protection to the page.
Consultation of the [Intel Architecture Software Developer's Manual](http://developer.intel.com/design/pentium4/manuals/index_new.htm),
Volume 2: Instruction Set Reference Manual gave the necessary bytes
for the `jmp` opcodes. The hook installation function creates the short and long jumps in jscript.dll, and creates the long
jump inside the hook function. The hook uninstallation function restores the jscript.dll bytes to what they were originally. `CLbxScriptUtil::get_Date`
just logs the current value of the counter to the logfile.

All this code is running well on my machine; you're welcome to come over and take a look at details of the implementation or at its
output.
