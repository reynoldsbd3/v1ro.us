---
author:
  description: Software Engineer
  email: reynoldsbd3@hotmail.com
  github: https://github.com/reynoldsbd3
  image: /images/avatar-64x64.png
  name: Bobby Reynolds
cardbackground: '#263238'
cardheaderimage: /images/default.jpg
cardthumbimage: /images/default.jpg
categories:
- presentation
date: 2017-02-15T19:00:00Z
description: Introduction to modern Python for intermediate student programmers
tags:
- meta
- python
title: Intermediate Introduction to Python
---

# Intermediate Introduction to Python

#### Bobby Reynolds

---

# Python

* General purpose programming language
* Interpreted, not compiled
* Completely cross-platform

Follow along:

* `sudo apt install python3 python3-pip`
* [https://www.python.org/downloads/release/python-360/](https://www.python.org/downloads/release/python-360/) for
  Windows users

???

Python is yet another programming language. For the most part, it has the same capabilities as any other general use
language like C or Java, but there are a number of differences that make Python particularly handy for the kinds of
problems you see in CTFs.

The first part of this presentation is an overview of Python's syntax and how that differs from other languages. I'll
use C as an example, because it's something we should all be familiar with. In the second section, we'll take an
interactive deep-dive into some advanced features that make Python a great language for hackers.

I highly encourage you to follow along. If you don't already have Python installed, you can get it from python.org or
using a package manager.

---

# Hello World

In C:

```c
#include <stdio.h>

int main() {

  printf("hello, world\n");
}
```

???

This is the standard C "hello world" program. Nothing particularly interesting is happening here.

--

To run:

```bash
$ gcc -o ex1 ex1.c
$ ./ex1
hello, world
$
```

???

C is a compiled language, meaning you must use a compiler like gcc to translate from source code into an executable
program.

---

# Hello World

In Python:

```python
print('hello, world')
```

???

And this is the exact same program, written in Python. I want to point out a few notable differences:

* There is no main function. Your program can do all its work at the top level without using any functions at all
* There is no semicolon. Python assumes that every statement in your program is on a separate line; more on this in a
  second
* There is no "`include`" directive. "`print`" is built in to the language itself

--

To run:

```bash
$ python3 ex1.py
hello, world
$
```

???

Running a Python program is also much easier: you simply use the "`python3`" command and give it the file you want to
run. In fact, if you're on a Mac or Linux system we can make things even easier by adding a line that looks like this to
the top of a Python program:

--

```python
#!/usr/bin/env python3
```

???

Then, we can directly run the script.

--

To run:

```bash
$ ./ex1.py
hello, world
$
```

???

Boom! I already like Python better than C. Let's keep looking at some of the differences.

---

# Variables

In C:

```c
int main() {

  int a = 1;
  int b = 2;
  printf("a = %d and b = %d\n", a, b);
}
```

In Python:

```python
a = 1
b = 2

print('a = %d and b = %d' % (a, b))
```

???

Here's an example that creates variables and prints them to the screen. Notice a few more differences:

* Variables have no type signatures
* Printing variables here uses the same `%d` syntax, but we have to use this weird `%` in order to insert our variables
  into the string being printed

--

Both Output:

```
a = 1 and b = 2
```

???

Both give the exact same output.

---

# Printing

Old C-style string formatting:

```python
print(
  'a = %d and b = %d' % (a, b)
)
```

Modern string formatting:

```python
print(
  'a = {} and b = {}'.format(a, b)
)
```

Direct string interpolation (Python 3.6+):

```python
print(
  f'a = {a} and b = {b}'
)
```

???

In fact, there are tons of ways to print your data to the screen besides what I used in the last example. We can still
use the printf format strings from C, or we can use the `format` method which uses this curly brace syntax.

I'm personally a fan of this new feature, direct interpolation. Under the hood, it works exactly the same as
`str.format`, but I think the syntax is a lot more intuitive.

--

  
Resources:

* [PyFormat](https://pyformat.info/) - an online reference for Python string formatting
* [PEP 498](https://www.python.org/dev/peps/pep-0498/) - official specification for new style direct interpolation

???

Here are some references for string formatting, for the curious

---

# Control Flow

???

Now let's move on to control flow.

Python provides the same control flow structures as any other language. For the most part, they work the same and look
almost the same.

--

`if` branches in C:

```c
if (a != 0) {
  printf("a is not zero\n");
  printf("duh\n");
}
```

???

This is an `if` statement in C.

--

`if` branches in Python:

```python
if a != 0:
    print('a is not zero')
    print('duh')
```

???

And here's the same `if` construct in Python. Ultimately, this code behaves *identically* to the above C version, but
there are a few syntactic differences worth noting:

* Python is smart and allows us to omit the parenthesis around our `if` condition
* Instead of curly braces, we have a colon and a few lines of indented code. Unlike other languages, Python actually
  cares about indentation and uses it to recognize blocks of code

---

# Control Flow

`while` loop in C:

```c
int counter = 0;
while (counter < 5) {
  printf("counting: %d\n", counter);
  counter++;
}
```

???

Here we have a while loop in C

--

`while` loop in Python:

```python
counter = 0
while counter < 5:
    print(f'counting: {counter}')
    counter += 1
```

???

And here's the same code translated to Python. You'll notice a lot of similarities to Python's `if` such as lack of
parenthesis and indented code block.

---

# Control Flow

`for` loop in C:

```c
for (int i = 0; i < 5; i++) {
  printf("iterating: %d\n", i);
}
```

???

This is what a `for` loop in C looks like.

--

`for` loop in Python:

```python
for i in range(5):
    print(f'iterating: {i}')
```

???

And here's the same for loop in Python. This is where things start to get interesting, because this looks pretty
different from the C version.

In fact, what's happening under the hood *is* completely different. A `for` loop in C is really just a fancy way of
combining your initialization, condition, and update expressions in the same line.

However, Python's `for` loop is much closer to what some languages call a `foreach` loop. It takes a list of items and
performs the loop body for each item on the list. In this case, the loop is iterating over the list of numbers 1, 2, 3,
4, and 5.

I'm going to come back to this example in a little bit, because `for` is one of the most useful things in Python.

---

# Control Flow

Functions in C:

```c
int add_them(int a, int b) {
  
  int c = a + b;
  return c;
}
```

???

Functions in C look like this. This is a simple function that adds two numbers and returns the result.

--

Functions on Python:

```python
def add_them(a, b):
    c = a + b
    return c
```

???

This is the same function in Python. No need to write out the return type or the types of arguments; Python doesn't
really care!

---

# Data Types

???

Just like other languages, Python has a few fundamental data types that we can use to build our programs.

---

# Data Types

Numbers in C:

```c
int a = 1;
float b = 2.7;
long long unsigned c = 9223372036854775807;
```

???

Let's say we have a C program with data that looks like this. Here we have an integer, a floating-point number, and a
really really big integer.

--

Nubers in Python:

```python
a = 1
print(type(a)) # <class 'int'>

b = 2.7
print(type(b)) # <class 'float'>

c = 9223372036854775807
print(type(c)) # <class 'int'>
```

???

Here is how we would represent that data in a Python program. I've added some print statements to show you explicitly
which data types are being used under the hood.

The numeric data types `int` and `float` are used to represent numbers. Notice that we don't have to designate any type
signatures; Python infers which type to use based on the literal assignment.

Also notice that `int` is used to represent small and large numbers. The  `int` datatype in Python can store an
infinitely large number, whereas in C we have to use a different type if we want to store a larger number, and we are
still limited by the machine's hardware.

---

# Data Types

Strings in C:

```c
char *msg = "hello, world";
```

Strings in Python:

```python
msg = 'hello, world'
print(type(msg)) # <class 'str'>
```

???

Python uses the `str` type to represent textual data much in the same way that C uses `char *`.

---

# Data Types

Arrays in C:

```c
int numbers[] = {1, 2, 3};
char characters[] = {'a', 'b', 'c'};
```

???

In C, there are arrays which hold a collection of items of the same type.

--

Lists in Python:

```python
numbers = [1, 2, 3]
characters = ['a', 'b', 'c']
```

???

In Python, these are called `list` objects, and they are created using the square bracket syntax shown here.

---

# Data Types

???

Pretty much anything you can do with an array in C, you can also do with a list in Python.

--

Creating a list:

```python
a = 3
my_list = [1, 2, a]
```

???

You can create lists from literal values or by stringing together variables. You can even do a combination of the two as
shown here.

--

Accessing elements:

```python
print(my_list[0]) # 1
print(my_list[1]) # 2
```

???

Accessing elements from the list works exactly the same as C, using a zero-based index and the square-bracket notation.

But lists in Python have a few tricks up their sleeves.

--

```python
print(my_list[-1]) # 3
print(my_list[-2]) # 2
```

???

We can also index backwards, starting from the end of the list. This example gets the last element using -1 and the
second-to-last element using -2.

--

Slicing lists:

```python
print(my_list[1:2]) # [2]
print(my_list[:2])  # [1, 2]
print(my_list[-2:]) # [2, 3]
```

???

Finally, we can slice and dice lists into smaller sub-lists using indices and a `:`

---

# Data Types

Dictionaries:

```python
my_dict = {
  'a': 1,
  'b': 2,
  'c': 3,
}
```

???

There's one more data type that doesn't really have a parallel in C, and that's the dictionary. Dictionaries give you
the ability to associate keys and values. In this case, we're associating the strings 'a', 'b', and 'c' with the
integers 1, 2, and 3.

--

Accessing dictionaries:

```python
print(my_dict['a']) # 1
print(my_dict['b']) # 2
```

???

Accessing a dictionary in Python looks a lot like accessing a list; that is, it uses the same square-bracket notation,
but instead of providing an integer index you pass in a key and you get back the value.

---

# Type System

???

Types are super important in programming languages, because they help us reason about the correctness of our programs.

--

Illegal in C:

```c
char *msg = "the number is: ";
int num = 5;

char *full_message = msg + num;
```

???

For example, C will not let you add a number to a string. Given our knowledge about data types in the C language, this
code just doesn't make any sense, and the compiler will yell at us if we write it.

--

Legal in Python:

```python
msg = 'the number is: '
num = 5

full_message = msg + num
```

???

However, the story is quite different in Python! This code is perfectly legal, but its results aren't always what you'd
expect. In general, Python does not care about types at all. Instead of telling you that your code is invalid and
refusing to run it, Python will happily attempt to add an integer to a string.

This is where things get hairy. Instead of getting a warning ahead of time in the form of a compiler error, Python's
philosophy is to crash at run time when you try to do an unsupported operation like this.

At first, this might seem stupid. If it's possible to know ahead of time that things are going to go wrong, why wouldn't
I want to stop that from happening?

The reasoning behind this philosophy is complicated, but it turns out that this is a really useful feature. To fully
understand why this is a good thing, we need to learn more about Python's object-oriented features. Later, we'll come
back to this philosophy and I'll show you and example of why this is a feature, not a bug.

---

# Objects

The `Apple` object:

```python
class Apple:

    def __init__(self, color, flavor):
        self.color = color
        self.flavor = flavor
    
    def get_color(self):
        return self.color
    
    def get_flavor(self):
        return self.flavor
```

???

Python is an object-oriented language, meaning it allows us to define our own types and their behaviors. In this example
we have the `Apple` type. An `Apple` has a color and a flavor which we can get using these "getter" methods. The color
and flavor of an apple are set in the constructor, which is named `__init__`.

There are a few peculiar things about this syntax:

* In some other languages, you can use the `this` keyword automatically. In Python, `this` is called `self`, and it must
  be explicitly named as a method's first parameter.
* Note the double-underscores around `init`. This is Python's way of reserving special function names. If you create a
  function called `__init__`, it will be automatically used as the constructor.

---

# Objects

Creating and using an `Apple`:

```python
a = Apple('red', 'sweet')
print(a.get_color()) # red
print(a.get_flavor()) # sweet
```

???

To create a new instance of this Apple object, we use its name as if it were a function. Under the hood, this calls the
`__init__` function with the given arguments.

We can then call `get_color` and `get_flavor` on the new instance.

--

Directly accessing fields:

```python
print(a.color) # red
print(a.flavor) # sweet
```

???

We can also just directly access the `color` and `flavor` fields of the `Apple` without using the getters. For this
reason, you don't usually see getters or setters in Python.

---

# Objects

???

Now let's say I want to do something with the apple, such as eat it.

--

How to eat an `Apple`:

```python
def eat(apple):
    print(f'You take a bite of the {apple.color} apple.')
    print(f'It tastes {apple.flavor}.')
```

???

We can write a function for eating apples that looks like this.

--

Eating some `Apple`s:

```python
apple1 = Apple('green', 'sour')
apple2 = Apple('red', 'delicious')

eat(apple1)
eat(apple2)
```

Output:

```bash
You take a bite of the green apple.
It tastes sour.
You take a bite of the red apple.
It tastes delicious.
```

???

Using this function looks like this.

Simple enough, right? We've created a new object type called `Apple`, created a few instances of this type, and eaten
them using the `eat` function.

---

# Objects

```python
class Candy:
    
    def __init__(self, color):
        self.color = color
        self.flavor = 'sweet'
```

???

Now suppose we have another object type called Candy that looks like this. Pretty similar to the `Apple`, except flavor
is always sweet.

But what if we want to eat the Candy like we ate the `Apple`?

--

Eating the `Candy`:

```python
def eat_candy(candy):
    print(f'You take a bite of the {candy.color} candy.')
    print(f'It tastes {candy.flavor}.')

chocolate = Candy('brown')
eat_candy(chocolate)
```

Output:

```bash
You take a bite of the brown candy.
It tastes sweet.
```

???

We could make an `eat_candy` function just like we did for the `Apple`, but it looks almost exactly the same as before.

---

# Objects

Re-using eat:

```python
chocolate = Candy('brown')
eat(chocolate)
```

???

Here's where not having types starts coming in handy. Since functions in Python don't require any specific type, I can
take a piece of `Candy` and eat it exactly as if it were an `Apple`.

Of course, since the eat function tries to access `apple.color` and `apple.flavor`, whatever I pass in must have those
two properties, or things are going to break when I try to do this.

There's one problem though...

--

Output:

```bash
You take a bite of the brown apple.
It tastes sweet.
```

Gross!

???

We didn't eat an apple, we ate chocolate. The eat function still assumes that we're eating an apple.

Any ideas on how to fix this?

---

# Objects

To string:

```python
class Apple:
    
    ...

    def __str__(self):
        return self.color + ' apple'

class Candy:
    
    ...

    def __str__(self):
        return self.color + 'candy'
```

???

I think I want to fix this by standardizing the way in which I display my foods. Other languages call this method
`toString`, but Python just calls it `__str__`.

Again, we see the double-underscore nomenclature used for built-in features.

---

# Objects

???

Now I can rewrite my `eat` function to accept any type of food.

--

Revised `eat` function:

```python
def eat(food):
    print(f'You take a bite of the {food}.')
    print(f'It tastes {food.flavor}.')
```

???

As long as the object we pass in has a `flavor` and a sensible implementation of "to string", we can `eat` it and get
output that makes sense.

For example:

--

Putting it all together:

```python
apple = Apple('red', 'delicious')
chocolate = Candy('brown')

eat(apple)
eat(chocolate)
```

Output:

```bash
You take a bite of the red apple.
It tastes delicious.
You take a bite of the brown candy.
It tastes sweet.
```

???

Now we can pass candies and apples into eat, and things just end up working! This is just a trivial example of why not
having types can be a good thing. It's a source of flexibility.

Any questions so far?

Cool, now we're going to dip into some intermediate features that make Python extra cool.

---

# Functions

???

In Python, functions are first-class objects. We can assign them to variables and pass them around like data.

--

Example:

```python
def do_something(action, item):
    action(item)

what_to_do = eat
what_to_eat = Apple('green', 'sour')

do_something(what_to_do, what_to_eat)
```

???

In this example, we take the eat function and assign it to a variable. We also create an apple, which is eventually
going to get eaten.

Then, instead of manually calling `eat`, we pass the action and the item into this `do_something` method, which calls
the action for us.

Questions? It really is that simple.

---

# Lambdas

Example:

```python
def do_something(action):
    action()
```

???

Here's a slightly different example. Let's say we want `do_something` to eat for us. But, we can no longer specify the
argument like we could before.

Python provides a construct called a `lambda` that allows us to dynamically change function signatures on the fly.

--

Using a lambda:

```python
apple = Apple('yellow', 'tart')
snack = lambda: eat(apple)

snack()

do_something(snack)
```

???

Using a lambda, we create `snack`, which is a new function that takes no arguments and does the same thing as eating the
apple.

---

# Lambdas

Example:

```python
def do_something(action, arg1, arg2):
    action(arg1, arg2)
```

???

We can also go the other way with lambdas and create a function which accepts *more* arguments.

--

Using a lambda:

```python
feast = lambda food, _: eat(food)

apple = Apple('brown', 'gross')
feast(apple, None)

do_something(feast, apple, None)
```
