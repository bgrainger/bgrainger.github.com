---
layout: post
title: static in C++
---
What does the C/C++ keyword `static` mean? When used inside a function, e.g.:

{% highlight c++ %}
int f(int a)
{
    static int b = 1;
    return a + ++b;
}
{% endhighlight %}

it means that a single instance of this variable is allocated for the lifetime of the program. In terms of storage,
it's very similar to a global, but its scope is restricted to that function.

C++ inherits this sense of `static` for class definitions. If a variable is declared as `static` in a class, there is one instance of that variable (again, allocated very similarly to a global) that is shared between all
instances of that class. Declaring the variable as `protected` or `private` controls its scope.

There is another meaning of `static`. In C, it was the opposite of `extern`; it meant that a variable had internal linkage semantics and wasn't visible from any other translation unit. (That is, you could only access it from within the file in which it was defined.) This use of `static` is deprecated in C++; use an unnamed namespace instead:

{% highlight c++ %}
namespace
{
    int s_variable = 1;
}
{% endhighlight %}

Since this use is deprecated, `static` should only be used inside a class or function definition. This rule would prohibit, for example, declaring static constants (or worse yet, variables) in a header file. The problem with this is that every source file that includes that header file gets its own copy of the constant or variable. There are no errors about duplicate symbols, because the linker respects the old C meaning of internal linkage. In the case of constants, barring linker optimisation, the constant would appear multiple times in the binary, instead of just once. In the case of variables, although it might initially appear that the functions in different source files are sharing the variable, each source file has its own copy and changes to the variable won't be seen between different source files.
