# Static type checking with Python

With Python 3.5 came [type hints](https://www.python.org/dev/peps/pep-0484/). A system within Python to *hint* what the type of a variable should be. Probably best explained by a quote from a certain [Disney classic](https://www.imdb.com/title/tt0325980/):

"The code is more what you'd call guidelines than actual rules"

In turn, in Python these hints do not do anything on their own, they are just hints. However there are programs that can take these hints and help you write better code. This is what a type hint looks like in Python:

```Py
def add(a: int, b: int) -> int:
    return a + b
```

Admittedly the syntax takes some getting used to, but all it says is, here is a function named `add`. `add` takes two integers and will return an integer. Then there are programs such as [mypy](http://mypy-lang.org/) that can take these hints and check whether the program is free of any Type Errors. All before actually running the code.


## Background
<details>
<summary>Types and progamming languages</summary>

Different programming languages have different type systems, but why? Take a quick peek at the example below:


### Python

```Py
def add(a, b):
    return a + b
```

Python's approach is simple, we'll just run the code and see if it works. If `a` and `b` can be added together, then great let's add them together. This all works:

```Py
add(1, 2)
add(1, 2.0)
add([1, 2], [3, 4])
```

But this does **not**:

```Py
add([1, 2], 3)
```

And worse yet, we won't know that it does not work until this code is actually run. If the code is not properly tested, then running this function might not happen until its shipped to the client. In which case... **nightmares**.


### C

Okay, but what about other languages? Remember C?

```C
// C
int add(int a, int b) {
    return a + b;
}
```

C takes a wildly different approach, put a concrete type in front of everything and check it when trying to compile. That way we'll know up front whether the code will even run. Because this:

```
add(1.0, 2.0);
```

Will nicely throw a compile error. No chance that this code reaches the client's desk! But wait, surely adding floats together is fine, right? Easy fix, write more code:

```C
float add(float a, float b) {
    return a + b;
}
```

> For the curious, there are ways to escape C's type system through the use of casting and pointers. Most notably through the use of `void` pointers.


### Java

Let's find a middleground, Java. Java is a bureaucratic programming languages of sorts. Nothing is assumed, and everything has to be explicitly denoted. Here is an example: 

```java
public static Number add(Number a, Number b) {
    return a.doubleValue() + two.doubleValue();
}
```

Quickly jumping over `public static`, which just means this function can be called from anywhere (`public`) and is always available (`static`). You'll find the word `Number`.

What is a `Number`? Well, as it turns out, [Number](https://docs.oracle.com/javase/8/docs/api/java/lang/Number.html) is a `class` of which each number (`Integer`, `Float`, `Double`, etc.) inherits. So really what this function says is, anything that inherits from Number, and thus can do anything a Number can do, can be passed into this function.

`a.doubleValue()`??? Inside the function it is now unknown what types `a` and `b` are. All we know is that they are both `Number`s. At this point we can only assume that `a` and `b` can do anything a `Number` can do and that is not much! In fact, a `Number` in Java can only convert itself to a more concrete `Double`, `Integer`, `Float`, etc. So what this function does is convert both `a` and `b` to `Double`s and then adds them together. Luckily the resulting `Double` is also a `Number` so we can immediately return it.

Through this, this all works:

```java
add(1, 2);
add(4.0, 5);
add(4.0, 5.0);
```

And this will still nicely give a compile error:

```java
add("hello", "bye");
```

Problem solved... right? Well, we did end up paying a steep price. Because be honest, which one is easier to understand:

```Py
def add(a, b):
    return a + b
```

```java
public static Number add(Number a, Number b) {
    return a.doubleValue() + two.doubleValue();
}
```

Suprisingly perhaps, is that the answer to that question depends on who you'd ask. If you are used to dealing with millions of lines of code written by many others before you, you might strongly prefer the `Java` implementation. Simply because it gives you all information you need to know.
</details>

<details>
<summary>No type system</summary>

All your computer has is `1`-s and `0`-s, and all it can do is operate on these ones and zeroes. So at the root there are effectively no types. Instead the program just has to treat certain ones and zeroes differently than others. This is true in machine code (the ones and zeros), but also one level higher in [an assembly language](https://cs.lmu.edu/~ray/notes/x86assembly/). These type of languages are often just the instructions an operating system can execute, but in a more convenient text form. Assembly languages do contain data instructions for storing the length and allignment of data, but not much more than that. Naturally this way of working is error prone and undesirable. 
</details>

<details>
<summary>Dynamic vs Static</summary>

All programming languages have some form of type system, but when and what they do with that system varies. First, let's talk about when. There are two main forms, **static** and **dynamic**, and they are not exclusive from one another. 

#### Static

Static in this context just means before execution, that could be when compiling the code or through running a seperate type checker. For instance, C makes use of static type checking to ensure that all types operate with one another upon compilation. That way, there is no (technically, little) chance for any type errors while running the program. On top of this C compilers also make use of the type information upon compilation to better optimize the resulting program. By for instance reserving precisely enough memory, as the data and their types is know up front.

#### Dynamic

Dynamic means during execution of a program, or in runtime. A good example of a dynamic type system is Python. Values in Python do have types, there are `int`s, `list`s, `string`s, you name it. Misuse of these types will often result in an error, for instance this code:

```Py
"hello" + [1,2,3]
```

Will raise a `TypeError` upon execution. But only during execution. So the information is there, and Python will protect you from weird and unexpected results, but a little late perhaps.

That said, dynamic type systems are often flexible and easy to use. As a programmer you don't have to worry about declaring types, and that means writing less code and probably easier to read code. This is a big reason as to why scripting languages such as Python, JavaScript and Bash tend to favor dynamic type systems. The flexibility in turn makes it possible to do extensive introspection, allowing the program itself to reason about types too. For instance in Python you can check the type of a variable through:

```Py
isinstance(a, int)
```
</details>

<details>
<summary>Type Checking</summary>

Type information can be used for different things, such as optimizing programs, ensuring enough memory is available, but perhaps most important to us programmers: type checking. Ensuring that the program is free of any type errors. 

Type checking can be done both dynamically and statically. But, dynamic type checking happens while running the program and will inevitably impact performance. That is why dynamic type checking is usually only done upon execution of a line of code, to ensure no unnecessary checks are done. That execution of a line of code might be very late, and if you are not testing properly, it might just be in the hands of the end user by the time that buggy line of code finally runs.

In contrast static type checking does not need to worry about performance (as much). Afterall, this style of type checking happens in the developer's time (hence the "as much"), and not the end user's time. This enables a static type checker to do more complex type of checks and give better hints as to what is going wrong. However, to perform static type checking the information on types needs to be available before running the code. That often means that you as a programmer need to add this information. That is both a blessing and a curse. You will have to write more code, but the added information will make it easier for your co-workers or your future self to understand.

Static type checking is often preferred, and so much so that languages such as JavaScript (in the form of TypeScript) and Python have started to adopt type information to enable static type checking. 
</details>
