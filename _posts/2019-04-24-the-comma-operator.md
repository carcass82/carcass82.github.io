---
title: "The Comma Operator"
tags:
  - Rants
  - C
  - C++
---

It's easy to get _lost in parentheses_ <!-- more --> when you have _complex_ functions that takes a lot of input parameters, like a `foo` function which takes an object which based on some condition will be scaled by some factor and will be appended to a processing list based on some other operation depending on its visibility:

```c++
bool foo(Object* object, float scale, bool append_to_list = true);
```

You may end up with code like this, where you want to execute some code depending on the result of `foo`:

```c++
if (foo(object, (condition? 2.f : 1.f)), bar(object->IsVisible()))
{
    ...
}
```

If you were not paying enough attention you could have missed that this code is equivalent to:

```c++
if (bar(object->IsVisible()))
{
    ...
}
```

because of _parentheses_. Instead of `if (foo(x, y, z))` I wrote `if (foo(x, y), z)`. The default parameter of `foo()` and the _comma operator_ make the code perfectly legal. It compiles fine. It does not generate any warning. What happens here though is that `foo(x, y)` gets executed, its return value is ignored and then `z` is executed and its return value is evaluated by the `if()`.

A formal description of the comma operator can be found i.e. on [cppreference](https://en.cppreference.com/w/cpp/language/operator_other):
> E1, E2
>
> In a comma expression E1, E2, the expression E1 is evaluated, **its result is discarded** [...] and its side effects are completed before evaluation of the expression E2 begins

This whole post is just a _gentle self-reminder_ of the existence of this _beautiful operator_ since an _impossible_ bug survived a good month in production code because _"Ehy, the `if` is there! The problem must be elsewhere!"_.

I think a warning would be _really_ helpful for the unused value in conditional statement. Pretty much like the one you can see for code like

```c++
for (...); // <-- :/
{
    do_something();
}
```

While both _gcc_ and _clang_ actually emit a warning with `-Wunused-value` (enabled with `-Wall`), _msvc_ does not. Well, this may change [soon](https://developercommunity.visualstudio.com/idea/601039/add-wunused-value-warning-in-vs.html).

_Oh, and for extra fun the operator `T2& operator,(const T1&, T2&)` could of course be overloaded!_
