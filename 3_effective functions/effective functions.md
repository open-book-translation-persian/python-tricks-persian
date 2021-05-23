
<h1>Effective Functions</h1>

<h3 dir='rt1'>3.1	Python’s Functions Are First-Class</h3>

Python’s functions are first-class objects. You can assign them to vari- ables, store them in data structures, pass them as arguments to other functions, and even return them as values from other functions.
Grokking these concepts intuitively will make understanding advanced features in Python  like  lambdas  and  decorators  much easier. It also puts you on a path towards functional programming techniques.
Over the next few pages I’ll guide you through a number of examples to help you develop this intuitive understanding. The examples will build on top of each other, so you might want to read them in sequence and even to try out some of them in a Python interpreter session as you go along.
Wrapping your head around the concepts we’ll be discussing here might take a little longer than you’d expect. Don’t worry—that’s completely normal. I’ve been there. You might feel like  you’re banging your head against the wall, and then suddenly things will “click” and fall into place when you’re ready.
Throughout this chapter I’ll be using this yell function for demon- stration purposes. It’s a simple toy example with easily recognizable output:

``` python
def yell(text):
return text.upper() + '!'

>>> yell('hello')
'HELLO!'
```

<h5 dir='rt1'>Functions Are Objects</h5>

All data in a Python program is represented by objects or relations between objects.1   Things like strings,  lists,  modules,  and functions are all objects. There’s nothing particularly special about functions in Python. They’re also just objects.
Because the yell function is an object in Python, you can assign it to another variable, just like any other object:

``` python
 >>> bark = yell	
```

This line doesn’t call the function. It takes the function object refer- enced by yell and creates a second name, bark, that points to it. You could now also execute the same underlying function object by calling bark:

``` python
>>> bark('woof')
'WOOF!'
```

Function objects and their names are two separate concerns. Here’s more proof: You can delete the function’s original name (yell). Since another name (bark) still points to the underlying function, you can still call the function through it:

``` python
>>> del yell

>>> yell('hello?')
NameError: "name 'yell' is not defined"

>>> bark('hey')
'HEY!'
```

By the way,  Python attaches a string identifier to every function at creation time for debugging purposes. You can access this internal identifier with the __name__ attribute:

``` python
>>> bark.  name 	
'yell'
```

Now, while the function’s  name  is still “yell,”  that doesn’t affect how you can access the function object from your code. The name identifier is merely a debugging aid. A variable pointing to a function and the function itself are really two separate concerns.

<h5 dir='rt1'>Functions Can Be Stored in Data Structures</h5>

Since functions are first-class citizens, you can store them in data structures, just like you can with other objects. For example, you can add functions to a list:

``` python
>>> funcs = [bark, str.lower, str.capitalize]
>>> funcs
[<function yell at 0x10ff96510>,
<method 'lower' of 'str' objects>,
<method 'capitalize' of 'str' objects>]
```

Accessing the function objects stored inside the list works like it would with any other type of object:

``` python
>>> for f in funcs:
...	print(f, f('hey there'))
<function yell at 0x10ff96510> 'HEY THERE!'
<method 'lower' of 'str' objects> 'hey there'
<method 'capitalize' of 'str' objects> 'Hey there'
```

You can even call a function object stored in the list without first as- signing it to a variable. You can do the lookup and then immediately call the resulting “disembodied” function object within a single expres- sion:

``` python
>>> funcs[0]('heyho')
'HEYHO!'
```

<h5 dir='rt1'>Functions Can Be Passed to Other Functions</h5>

Because functions are objects, you can pass them as arguments to other functions. Here’s a greet function that formats a greeting string using the function object passed to it and then prints it:

``` python
def greet(func):
  greeting = func('Hi, I am a Python program') 
  print(greeting)
```

You can influence the resulting greeting by passing in different func- tions. Here’s what happens if you pass the bark function to greet:

``` python
>>> greet(bark)
'HI, I AM A PYTHON PROGRAM!'
```

Of course, you could also define a new function to generate a differ- ent flavor of greeting. For example, the following whisper function might work better if you don’t want your Python programs to sound like Optimus Prime:

``` python
def whisper(text):
  return text.lower() + '...'

>>> greet(whisper)
'hi, i am a python program...'
```

The ability to pass function objects as arguments to other functions is powerful. It allows you to abstract away and pass around behavior in your programs. In this example, the greet function stays the same but you can influence its output by passing in different greeting behaviors.

Functions that can accept other functions as arguments are also called higher-order functions. They are a necessity for the functional pro- gramming style.
The classical example for higher-order functions in Python is the built- in map function. It takes a function object and an iterable, and then calls the function on each element in the iterable, yielding the results as it goes along.

Here’s how you might format a sequence of greetings all at once by
mapping the bark function to them:

``` python
>>> list(map(bark, ['hello', 'hey', 'hi']))
['HELLO!', 'HEY!', 'HI!']
```

As you saw, map went through the entire list and applied the bark func- tion to each element. As a result, we now have a new list object with modified greeting strings.

<h5>Functions Can Be Nested</h5>

Perhaps surprisingly, Python allows functions to be defined inside other functions. These are often called nested functions or inner func- tions. Here’s an example:

``` python
def speak(text):
  def whisper(t):
    return t.lower() + '...'
  return whisper(text)

>>> speak('Hello, World')
'hello, world...'
```

Now, what’s going on here? Every time you call speak, it defines a new inner function whisper and then calls it immediately after. My brain’s starting to itch just a little here but, all in all, that’s still rela- tively straightforward stuff.

Here’s the kicker though—whisper does not exist outside speak:

``` python
>>> whisper('Yo')
NameError:
"name 'whisper' is not defined"

>>> speak.whisper
AttributeError:
"'function' object has no attribute 'whisper'"
```

But what if you really wanted to access that nested whisper function from outside speak? Well, functions are objects—you can return the inner function to the caller of the parent function.

For example, here’s a function defining two inner functions. Depend- ing on the argument passed to top-level function, it selects and returns one of the inner functions to the caller:

``` python
def get_speak_func(volume):
  def whisper(text):
    return text.lower() + '...'
  def yell(text):
    return text.upper() + '!'
  if volume > 0.5:
    return yell
  else:
    return whisper
```

Notice how get_speak_func doesn’t actually call any of its inner functions—it simply selects the appropriate inner function based on the volume argument and then returns the function object:

``` python
>>> get_speak_func(0.3)
<function get_speak_func.<locals>.whisper at 0x10ae18>

>>> get_speak_func(0.7)
<function get_speak_func.<locals>.yell at 0x1008c8>
```

Of course, you could then go on and call the returned function, either directly or by assigning it to a variable name first:

``` python
>>> speak_func = get_speak_func(0.7)
>>> speak_func('Hello')
'HELLO!'
```

Let that sink in for a second here… This means not only can functions accept behaviors through arguments but they can also return behav- iors. How cool is that?
You know what, things are starting to get a little loopy here. I’m going to take a quick coffee break before I continue writing (and I suggest you do the same).

<h5>Functions Can Capture Local State</h5>

You just saw how functions can contain inner functions, and that it’s even possible to return these (otherwise hidden) inner functions from the parent function.
Best put on your seat belt now because it’s going to get a little crazier still—we’re about to enter even deeper functional programming terri- tory. (You had that coffee break, right?)
Not only can functions return other functions, these inner functions can also capture and carry some of the parent function’s state with them. Well, what does that mean?

I’m going to slightly rewrite the previous get_speak_func example to illustrate this. The new version takes a “volume” and a “text” argu- ment right away to make the returned function immediately callable:

``` python
def get_speak_func(text, volume):
  def whisper():
    return text.lower() + '...'
  def yell():
    return text.upper() + '!'
  if volume > 0.5:
    return yell
  else:
    return whisper

>>> get_speak_func('Hello, World', 0.7)()
'HELLO, WORLD!'
```

Take a good look at the inner functions whisper and yell now. Notice how they no longer have a text parameter? But somehow they can still access the text parameter defined in the parent function. In fact, they seem to capture and “remember” the value of that argument.

Functions that do this are called lexical closures (or just closures, for short). A closure remembers the values from its enclosing lexical scope even when the program flow is no longer in that scope.
In practical terms, this means not only can functions return behaviors but they can also pre-configure those behaviors. Here’s another bare- bones example to illustrate this idea:

``` python
def make_adder(n):
def add(x):
return x + n
return add

>>> plus_3 = make_adder(3)

>>> plus_5 = make_adder(5)

>>> plus_3(4) 
7
>>> plus_5(4)
9
```

In this example, make_adder serves as a factory to create and config- ure “adder” functions. Notice how the “adder” functions can still ac- cess the n argument of the make_adder function (the enclosing scope).

<h5>Objects Can Behave Like Functions</h5>

While all functions are objects in Python, the reverse isn’t true. Ob- jects aren’t functions. But they can be made callable, which allows  you to treat them like functions in many cases.
If an object is callable it means you can use the round parentheses function call syntax on it and even pass in function call arguments. This is all powered by the _call__ dunder method. Here’s an example of class defining a callable object:

``` python
class Adder:
  def init (self, n):
    self.n = n
  def call (self, x):
    return self.n + x

>>> plus_3 = Adder(3)
>>> plus_3(4) 
7
```

Behind the scenes, “calling” an object instance as a function attempts to execute the object’s __call__	method.

Of course, not all objects will be callable. That’s why there’s a built-in callable function to check whether an object appears to be callable or not:

``` python
>>> callable(plus_3) True
>>> callable(yell) True
>>> callable('hello')
False
```

<h5>Key Takeaways</h5>

   •	Everything in Python is an object, including functions. You can assign them to variables, store them in data structures, and pass or return them to and from other functions (first-class func- tions.)
   •	First-class functions allow you to abstract away and pass around behavior in your programs.
   •	Functions can be nested and they can capture and carry some of the parent function’s state with them. Functions that do this are called closures.
   •	Objects can be made callable. In many cases this allows you to treat them like functions.
