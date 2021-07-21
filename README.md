# Static type checking with Python

With Python 3.5 came [type hints](https://www.python.org/dev/peps/pep-0484/). A system within Python to *hint* what the type of a variable should be. Probably best explained by a quote from a certain [Disney classic](https://www.imdb.com/title/tt0325980/):

"The code is more what you'd call guidelines than actual rules"

In turn, in Python these hints do not do anything on their own, they are just hints. However there are programs that can take these hints and help you write better code. This is what a type hint looks like in Python:

```Py
def sum(a: int, b: int) -> int:
    return a + b
```

Admittedly the syntax takes some getting used to, but all it says is, here is a function named `add`. `add` takes two integers and will return an integer. Then there are programs such as [mypy](http://mypy-lang.org/) that can take these hints and check whether the program is free of any Type Errors. All before actually running the code.


## Background
<details>
<summary>Types and progamming languages</summary>

Different programming languages have different type systems, but why? Take a quick peek at the example below:


### Python

```Py
def sum(items):
    total = 0
    for item in items:
        total += item
    return total
```

Python's approach is simple, we'll just run the code and see if it works. If `items` can be summed, then great let's do that. This all works:

```Py
sum([1, 2])
sum([1, 2.0])
sum({1, 2, 3})
```

But this does **not**:

```Py
sum(["hello", 1])
```

And worse yet, we won't know that it does not work until this code is actually run. If the code is not properly tested, then running this function might not happen until its shipped to the client. In which case... **nightmares**.


### C

Okay, but what about other languages? Remember C?

```C
int sum(int items[], int n) {
    int total = 0;
    for (int i = 0; i < n; i++) {
        total += items[i];
    }
    return total;
}
```

C takes a different approach, put a concrete type in front of everything and check it when trying to compile. That way we'll know up front whether the code will even run. Because this:

```C
float array[] = {3.0, 4.0, 5.0};
sum(array, 3);
```

Will nicely throw a compile error. No chance that this code reaches the end user's desk. 

But wait, floats can be summed right? Well, tough luck. You'll need to write a new function for floats.

> For the curious, there are ways to escape C's type system through the use of casting and pointers. Most notably through the use of `void` pointers.


### Java

Let's find a middleground, Java. Java is a bureaucratic programming languages of sorts. Nothing is assumed, and everything has to be explicitly denoted. Here is an example: 

```java
public static <T extends Number> sum(Iterable<T> items) {
    T total = 0;
    for (T item : items) {
        total = total.doubleValue() + item.doubleValue();
    }
    return total;
}
```

Quickly jumping over `public static`, which just means this function can be called from anywhere (`public`) and is always available (`static`). You'll find `<T extends Number>`.

What is a `Number`? Well, as it turns out, [Number](https://docs.oracle.com/javase/8/docs/api/java/lang/Number.html) is a `class` of which each number (`Integer`, `Float`, `Double`, etc.) inherits. `<T extends Number>` just means any type `T` that is an extension of a `Number`. So really what `T` says is, anything that is a Number, and thus can do anything a Number can do, can be passed into this function and will be returned from it too. 

Next, `Iterable<T>`. Iterable is an abstract generic type, more on this later. For now, all this says is some collection of things, be it a list, an array, or a tuple perhaps, over which can be iterated (with a for-loop for instance), containing elements of type `T`.

`item.doubleValue()`??? Inside the function it is now unknown what the exact types of the items are. All we know is that they are `Number`s. At this point we can only assume that an item can do anything a `Number` can do, and that is not much! In fact, a `Number` in Java can only convert itself to a more concrete `Double`, `Integer`, `Float`, etc. So what this function does is convert all items to `Double`s and then sums them up. 

Through this, this all works:

```java
sum({1, 2});
sum({4.0, 5.0});
```

And even different data structures, such as linked lists:

```java
LinkedList<Number> items = new LinkedList<Number>();
items.add(1);
items.add(2.0);
sum(items);
```

And this will still nicely give a compile error:

```java
add({"hello", "bye"});
```

Problem solved... right? Well, we did end up paying a steep price. Because be honest, which one is easier to understand:

```Py
def sum(items):
    total = 0
    for item in items:
        total += item
    return total
```

```java
public static <T extends Number> sum(Iterable<T> items) {
    T total = 0;
    for (T item : items) {
        total = total.doubleValue() + item.doubleValue();
    }
    return total;
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

Static in this context just means before execution, that could be when compiling the code or through running a seperate type checker. For instance, C makes use of static type checking to ensure that all types operate with one another upon compilation. That way, there is no (technically, little) chance for any type errors while running the program. On top of this compilers can make use of the type information upon compilation to better optimize the resulting program. By for instance reserving precisely enough memory, as the data and their types is know up front.

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


## Type hints in Python

A type hint in the simplest form looks like this:

```Py
foo: int
```

All this says is, the type of the variable `foo` should be an integer. Notice how there is no initial value here. This line of code does not create a variable `foo`, all it does is add a hint that `foo`, once it exists, should be an integer. That means this will raise a `NameError`:

```
$ python
>>> foo: int
>>> foo
NameError: name 'foo' is not defined
```

It is possible to combine type hints and initialization on the same line, like so:

```Py
foo: int = 3
```

That looks somewhat redundant, doesn't it? How can the *literal* `3` be anything else than an integer? This is where type inference kicks in. Tools such as `mypy` will try to infer the types of variables from their use. It is quite safe to assume type inference is possible here, so probably best to just write:

```Py
foo = 3
```

Type inference does have its limitations, for instance `mypy` will not do any type inference in functions without type hints. To understand why, let's quickly look into function type hints. In the simplest form:

```Py
def add(a: int, b: int) -> int:
    c = a + b
    return c
```

The syntax is relatively straight forward, using the colon (`:`) for parameter type hints, and the arrow (`->`) for the return type. Notice how the type of `c` is not annotated. It can be, but it is not needed. From the types of `a` and `b` and the `+` operation, `mypy` can infer the type of `c`. But what if we did not annotate this function. Well, in that case, `a` and `b` could be anything: `str`, `float`, `list`, you name it! This is where `mypy` draws a line, if you do not annotate a function, `mypy` will not even attempt to do type inference. Instead all variables will be of type `Any`.

What is `Any`? Well, anything really. It is an escape hatch of sorts that provides no information. Once `Any` gets involved type checking becomes rather impossible. What is `Any + int`? `Any`

1. Annotate the `factorial` function below.

    ```Py
    def factorial(num):
        total = 1
        for i in range(2, num + 1):
            total *= i
        return total
    ```

    <textarea name="form[q1]" rows="5" required=""></textarea>


## Generics

Integers, floats, booleans and strings are primitive data types. Built into the language, they serve as building blocks for more complex data structures. For instance, you might need a `list` to store your data. 

```Py
numbers: list = [1, 2, 3]
number = numbers.pop()
```

Here is the catch, the type `list` does not tell *anything* about what is in the `list`. So really what we have here is a `list` containing `Any`. In this case the type of `number` would be `Any` too.

A `list` is a generic data type. It can store various types, but its operation will vary based on what you store. Simply put for a `list`, if you initially store integers in the list, you will later be able to retrieve integers from that list. This can be annotated as follows:

```Py
numbers: list[int] = [1, 2, 3]
number = numbers.pop()
```

Now `numbers` is defined as a list of integers, and through that `number` will be of type `int` too.

Let's take a quick look at `dict`. Dictionaries are generic over two types, their keys and values. This is how that can be annotated:

```Py
grades: dict[str, int] = {"Martijn": 7, "Marleen": 8}
```

Tuples are an immutable data structure, once initialized it cannot be changed. So it is known up front exactly what the type of each value in the tuple is going to be. Because of this the `tuple` type can a variable amount of generic anotations with exactly as many types as there are values. Like so:

```Py
foo: tuple[int, float] = (7, 7.2)
bar: tuple[int, float, str] = (8, 7.9, "hello world")
baz: tuple[int, int, int] = (1, 2, 3)
```

What about nested data structures?

```Py
stats: dict[str, tuple[int, float]] = {"Martijn": (7, 7.2), "Marleen": (8, 8.1)}
```

Again, in most situations `mypy` can infer the types of the variables, and it is not strictly needed to annotate each data structure for type checking. That said, especially when concerning data structures, annotations make the code easier to understand.  

2. Annotate the data structures below:

    ```Py
    foo = ["hello", "world"]
    ```

    <textarea name="form[q2.1]" rows="1" required=""></textarea>

    ```Py
    bar = [("Martijn", 1), ("Marleen", 2)]
    ```

    <textarea name="form[q2.2]" rows="1" required=""></textarea>

    ```Py
    baz = {1: {2: {3: "hello"}}}
    ```

    <textarea name="form[q2.3]" rows="1" required=""></textarea>


## Abstract types