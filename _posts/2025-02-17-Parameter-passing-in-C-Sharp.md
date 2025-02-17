---
title: Parameter passing in C#
date: 2025-02-17 09:05:54 +0200
---

Source: [jonskeet.uk](https://jonskeet.uk/csharp/parameters.html)

Microsoft also has a [good page](http://msdn.microsoft.com/en-us/library/8f1hz171.aspx) about this topic (which I believe uses exactly the same terminology as this page - let me know if they appear to disagree).

Note: Lee Richardson has written a [complementary article](https://rapidapplicationdevelopment.blogspot.com/2007/01/parameter-passing-in-c.html) to this one, particularly for those who learn well with pictures. Basically it illustrates the same points, but using pretty diagrams to show what's going on.

Table of contents
-----------------

*   [Preamble: what is a reference type?](#preamble1)
*   [Further preamble: what is a value type?](#preamble2)
*   [Checking you understand the preamble...](#check)
*   [The different kinds of parameters](#paramTypes)
*   [Value parameters](#value)
*   [Reference parameters](#ref)
*   [Output parameters](#out)
*   [Parameter arrays](#params)
*   [Mini-glossary](#gloss)

Preamble: what is a reference type?
-----------------------------------

Note: I have a [whole article about reference types and value types](references.html) which you may find useful, particularly if you're quite new to .NET. This preamble is a shorter version of the same information, but if you find anything confusing it's probably worth reading both.

In .NET (and therefore C#) there are two main sorts of type: _reference types_ and _value types_. Classes, delegates and interfaces are reference types; structs and enums are value types. They act differently, and a lot of confusion about parameter passing is really down to people not properly understanding the difference between them. Here's a quick explanation:

A _reference type_ is a type which has as its value a _reference_ to the appropriate data rather than the data itself. For instance, consider the following code:

```c-sharp
StringBuilder sb = new StringBuilder();
```

(I have used `StringBuilder` as a random example of a reference type - there's nothing special about it.) Here, we declare a variable `sb`, create a new `StringBuilder` object, and assign to `sb` a reference to the object. The value of `sb` is _not_ the object itself, it's the reference. Assignment involving reference types is simple - the value which is assigned is the value of the expression/variable - i.e. the reference:

```c-sharp
StringBuilder first = new StringBuilder();
first.Append("hello");
StringBuilder second = first;
Console.WriteLine(second); // Prints hello
```

Here we declare a variable `first`, create a new `StringBuilder` object, and assign to `first` a reference to the object. We then assign to `second` the value of `first`. This means that they both refer to the same object. If we modify the contents of the object via another call to `first.Append`, that change is visible via `second`:

```c-sharp
StringBuilder first = new StringBuilder();
first.Append("hello");
StringBuilder second = first;
first.Append(" world");
Console.WriteLine(second); // Prints hello world
```

They are still, however, independent variables themselves. Changing the value of `first` to refer to a completely different object (or indeed changing the value to a null reference) doesn't affect `second` at all, or the object it refers to:

```c-sharp
StringBuilder first = new StringBuilder();
first.Append("hello");
StringBuilder second = first;
first.Append(" world");
first = new StringBuilder("goodbye");
Console.WriteLine(first); // Prints goodbye
Console.WriteLine(second); // Still prints hello world
```

To reiterate: class types, interface types, delegate types and array types are all reference types.

Further preamble: what is a value type?
---------------------------------------

While reference types have a layer of indirection between the variable and the real data, _value types_ don't. Variables of a value type directly contain the data. Assignment of a value type involves the actual data being copied. Take a simple struct, for example:

```c-sharp
public struct IntHolder
{
    public int i;
}
```

Wherever there is a variable of type `IntHolder`, the value of that variable contains all the data - in this case, the single integer value. An assignment copies the value, as demonstrated here:

```c-sharp
IntHolder first = new IntHolder();
first.i = 5;
IntHolder second = first;
first.i = 6;
Console.WriteLine (second.i);
```

[(Download sample code)](Example2.cs) Output:

5

Here, `second.i` has the value 5, because that's the value `first.i` has when the assignment `second = first` occurs - the values in `second` are independent of the values in `first` apart from when the assignment takes place.

Simple types (such as `float`, `int`, `char`), enum types and struct types are all value types.

Note that many types (such as `string`) appear in some ways to be value types, but in fact are reference types. These are known as _immutable_ types. This means that once an instance has been constructed, it can't be changed. This allows a reference type to act _similarly_ to a value type in some ways - in particular, if you hold a reference to an immutable object, you can feel comfortable in returning it from a method or passing it to another method, safe in the knowledge that it won't be changed behind your back. This is why, for instance, the `string.Replace` doesn't change the string it is called on, but returns a new instance with the new string data in - if the original string were changed, any other variables holding a reference to the string would see the change, which is very rarely what is desired.

Contrast this with a _mutable_ (changeable) type such as `ArrayList` - if a method returns the `ArrayList` reference stored in an instance variable, the calling code could then add items to the list without the instance having any say about it, which is usually a problem. Having said that immutable reference types act _like_ value types, they are _not_ value types, and shouldn't be thought of as actually being value types.

For more information about value types, reference types, and where the data for each is stored in memory, please see my [other article about the subject](memory.html).

Checking you understand the preamble...
---------------------------------------

What would you expect to see from the code above if the declaration of the `IntHolder` type was as a class instead of a struct? If you don't understand why the output would be `6`, please re-read both preambles and [mail me](/cdn-cgi/l/email-protection#394a525c5c4d7949565b5641175a5654) if it's still not clear - if you don't get it, it's my fault, not yours, and I need to improve this page. If you _do_ understand it, parameter passing becomes very easy to understand - read on.

The different kinds of parameters
---------------------------------

There are four different kinds of parameters in C#: value parameters (the default), reference parameters (which use the `ref` modifier), output parameters (which use the `out` modifier), and parameter arrays (which use the `params` modifier). _You can use any of them with both value and reference types._ When you hear the words "reference" or "value" used (or use them yourself) you should be very clear in your own mind whether you mean that a parameter is a reference or value parameter, or whether you mean that the type involved is a reference or value type. If you can keep the two ideas separated, they're very simple.

Value parameters
----------------

By default, parameters are value parameters. This means that a new storage location is created for the variable in the function member declaration, and it starts off with the value that you specify in the function member invocation. If you change that value, that doesn't alter any variables involved in the invocation. For instance, if we have:

```c-sharp
void Foo (StringBuilder x)
{
    x = null;
}

...

StringBuilder y = new StringBuilder();
y.Append ("hello");
Foo (y);
Console.WriteLine (y==null);
```
[(Download sample code)](Example3.cs) Output:

False

The value of `y` isn't changed just because `x` is set to `null`. Remember though that the value of a reference type variable is the reference - if two reference type variables refer to the same object, then changes to the data in that _object_ will be seen via both variables. For example:

```c-sharp
void Foo (StringBuilder x)
{
    x.Append (" world");
}

...

StringBuilder y = new StringBuilder();
y.Append ("hello");
Foo (y);
Console.WriteLine (y);
```

[(Download sample code)](Example4.cs) Output:

hello world

After calling `Foo`, the StringBuilder object that `y` refers to contains "hello world", as in `Foo` the data " world" was appended to that object via the reference held in `x`.

Now consider what happens when value types are passed by value. As I said before, the value of a value type variable is the data itself. Using the previous definition of the struct `IntHolder`, let's write some code similar to the above:

```c-sharp
void Foo (IntHolder x)
{
    x.i=10;
}

...

IntHolder y = new IntHolder();
y.i=5;
Foo (y);
Console.WriteLine (y.i);
```

[(Download sample code)](Example5.cs) Output:

5

When `Foo` is called, `x` starts off as a struct with value `i=5`. Its `i` value is then changed to 10. `Foo` knows nothing about the variable `y`, and after the method completes, the value in `y` will be exactly the same as it was before (i.e. 5).

As we did earlier, check that you understand what would happen if `IntHolder` was declared as a class instead of a struct. You should understand why `y.i` would be 10 after calling `Foo` in that case.

Reference parameters
--------------------

Reference parameters don't pass the values of the variables used in the function member invocation - they use the variables themselves. Rather than creating a new storage location for the variable in the function member declaration, the same storage location is used, so the value of the variable in the function member and the value of the reference parameter will always be the same. Reference parameters need the `ref` modifier as part of both the declaration and the invocation - that means it's always clear when you're passing something by reference. Let's look at our previous examples, just changing the parameter to be a reference parameter:

```c-sharp
void Foo (ref StringBuilder x)
{
    x = null;
}

...

StringBuilder y = new StringBuilder();
y.Append ("hello");
Foo (ref y);
Console.WriteLine (y==null);
```

[(Download sample code)](Example6.cs) Output:

True

Here, because a reference to `y` is passed rather than its value, changes to the value of parameter `x` are immediately reflected in `y`. In the above example, `y` ends up being `null`. Compare this with the result of the same code without the `ref` modifiers.

Now consider the struct code we had earlier, but using reference parameters:

```c-sharp
void Foo (ref IntHolder x)
{
    x.i=10;
}

...

IntHolder y = new IntHolder();
y.i=5;
Foo (ref y);
Console.WriteLine (y.i);
```

[(Download sample code)](Example7.cs) Output:

10

The two variables are sharing a storage location, so changes to `x` are also visible through `y`, so `y.i` has the value 10 at the end of this code.

### Sidenote: what is the difference between passing a value object by reference and a reference object by value?

You may have noticed that the last example, passing a struct by reference, had the same effect in this code as passing a class by value. This doesn't mean that they're the same thing, however. Consider the following code:

```c-sharp
void Foo (??? IntHolder x)
{
    x = new IntHolder();
}

...

IntHolder y = new IntHolder();
y.i=5;
Foo (??? y);
```

Consider the case where `IntHolder` is a struct (i.e. a value type) and the parameter is a reference parameter (i.e. replace `???` with `ref` above). After the call to `Foo(ref y)`, the value of `y` is a new `IntHolder` value - i.e. `y.i` is 0.

In the case where `IntHolder` is a class (i.e. a reference type) and the parameter is a value parameter (i.e. remove `???` above), the value of `y` isn't changed - it's a reference to the same object it was before the function member call. **This difference is absolutely crucial to understanding parameter passing in C#, and is why I believe it is highly confusing to say that objects are passed by reference by default instead of the correct statement that object references are passed by value by default.**

Output parameters
-----------------

Like reference parameters, output parameters don't create a new storage location, but use the storage location of the variable specified on the invocation. Output parameters need the `out` modifier as part of both the declaration and the invocation - that means it's always clear when you're passing something as an output parameter.

Output parameters are very similar to reference parameters. The only differences are:

*   The variable specified on the invocation doesn't need to have been assigned a value before it is passed to the function member. If the function member completes normally, the variable is considered to be assigned afterwards (so you can then "read" it).
*   The parameter is considered initially unassigned (in other words, you must assign it a value before you can "read" it in the function member).
*   The parameter _must_ be assigned a value before the function member completes normally.

Here is some example code showing this, with an `int` parameter (`int` is a value type, but if you understood reference parameters properly, you should be able to see what the behaviour for reference types is):

```c-sharp
void Foo (out int x)
{
    // Can't read x here - it's considered unassigned

    // Assignment - this must happen before the method can complete normally
    x = 10;

    // The value of x can now be read:
    int a = x;
}

...

// Declare a variable but don't assign a value to it
int y;

// Pass it in as an output parameter, even though its value is unassigned
Foo (out y);

// It's now assigned a value, so we can write it out:
Console.WriteLine (y);
```

[(Download sample code)](Example8.cs) Output:

10

Parameter arrays
----------------

Parameter arrays allow a variable number of arguments to be passed into a function member. The definition of the parameter has to include the `params` modifier, but the use of the parameter has no such keyword. A parameter array has to come at the end of the list of parameters, and must be a single-dimensional array. When using the function member, any number of parameters (including none) may appear in the invocation, so long as the parameters are each compatible with the type of the parameter array. Alternatively, a single array may be passed, in which case the parameter acts just as a normal value parameter. For example:

```c-sharp
void ShowNumbers (params int\[\] numbers)
{
    foreach (int x in numbers)
    {
        Console.Write (x+" ");
    }
    Console.WriteLine();
}

...

int\[\] x = {1, 2, 3};
ShowNumbers (x);
ShowNumbers (4, 5);
```

[(Download sample code)](Example9.cs) Output:

1 2 3 
4 5 

In the first invocation, the value of the variable `x` is passed by value, as the type of `x` is already an array. In the second invocation, a new array of ints is created containing the two values specified, and a reference to this array is passed (still by value).

Mini-glossary
-------------

Some informal definitions and summaries of terms:

Function member

A function member is a method, property, event, indexer, user-defined operator, instance constructor, static constructor, or destructor.

Output parameter

A parameter very similar to a reference parameter, but with different definite assignment rules.

Reference parameter (pass-by-reference semantics)

A parameter which shares the storage location of the variable used in the function member invocation. As they share the same storage location, they always have the same value (so changing the parameter value changes the invocation variable value).

Reference type

Type where the value of a variable/expression of that type is a reference to an object rather than the object itself.

Storage location

A portion of memory holding the value of a variable.

Value parameter (the default semantics, which are pass-by-value)

A value parameter that has its own storage location, and thus its own value. The initial value is the value of the expression used in the function member invocation.

Value type

Type where the value of a variable/expression of that type is the object data itself.

Variable

Name associated with a storage location and type. (Usually a single variable is associated with a storage location. The exceptions are for reference and output parameters.)
