
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
   

<h3>3.2	Lambdas Are Single-Expression Functions</h3>

The lambda keyword in Python provides a shortcut for declaring small anonymous functions. Lambda functions behave just like regular functions declared with the def keyword. They can be used whenever function objects are required.

For example, this is how you’d define a simple lambda function carry- ing out an addition:

``` python
>>> add = lambda x, y: x + y
>>> add(5, 3)
8
```

You could declare the same add function with the def keyword, but it would be slightly more verbose:

``` python
>>> def add(x, y):
...	return x + y
>>> add(5, 3)
8
```

Now you might be wondering, “Why the big fuss about lambdas? If they’re just a slightly more concise version of declaring functions with def, what’s the big deal?”
Take a look at the following example and keep the words function ex- pression in your head while you do that:

``` python
>>> (lambda x, y: x + y)(5, 3)
8
```

Okay, what happened here? I just used lambda to define an “add” func- tion inline and then immediately called it with the arguments 5 and  3.
 

Conceptually, the lambda expression lambda x, y: x + y is the same as declaring a function with def, but just written inline.  The  key difference here is that I didn’t have to bind the function object to a name before I used it. I simply stated the expression I wanted to compute as part of a lambda, and then immediately evaluated it by calling the lambda expression like a regular function.

Before you move on, you might want to play with the previous code example a little to really let the meaning of it sink in. I still remember this taking me awhile to wrap my head around. So don’t worry about spending a few minutes in an interpreter session on this. It’ll be worth it.
There’s another syntactic difference between lambdas and regular function definitions. Lambda functions are restricted to a single expression. This means a lambda function can’t use statements or annotations—not even a return statement.
How do you return values from lambdas then? Executing a lambda function evaluates its expression  and  then  automatically  returns  the expression’s result, so there’s always an implicit return state- ment. That’s why some people refer to lambdas as single expression functions.

<h5>Lambdas You Can Use</h5>

When should you use lambda functions in your code? Technically, any time you’re expected to supply a function object you can use a lambda expression. And because lambdas can be anonymous, you don’t even need to assign them to a name first.
This can provide a handy and “unbureaucratic” shortcut to defining a function in Python. My most frequent use case for lambdas is writing short and concise key funcs for sorting iterables by an alternate key:

``` python
>>> tuples = [(1, 'd'), (2, 'b'), (4, 'a'), (3, 'c')]
>>> sorted(tuples, key=lambda x: x[1]) 
[(4, 'a'), (2, 'b'), (3, 'c'), (1, 'd')]
```

In the above example, we’re sorting a list of tuples by the second value in each tuple. In this case, the lambda function provides a quick way to modify the sort order. Here’s another sorting example you can play with:

``` python
>>> sorted(range(-5, 6), key=lambda x: x * x)
[0, -1, 1, -2, 2, -3, 3, -4, 4, -5, 5]
```

Both examples I showed you have more concise implementations in Python using the built-in operator.itemgetter() and abs() func- tions. But I hope you can see how using a lambda gives you much more flexibility. Want to sort a sequence by some arbitrary computed key? No problem. Now you know how to do it.

Here’s another interesting thing about lambdas: Just like regular nested functions, lambdas also work as lexical closures.
What’s a lexical closure? It’s just a fancy name for a function that remembers the values from the enclosing lexical scope even when the program flow is no longer in that scope. Here’s a (fairly academic) example to illustrate the idea:

``` python
>>> def make_adder(n):
...	return lambda x: x + n

>>> plus_3 = make_adder(3)
>>> plus_5 = make_adder(5)

>>> plus_3(4) 7
>>> plus_5(4)
9
```

In the above example, the x + n lambda can still access the value of n even though it was defined in the make_adder function (the enclosing scope).

Sometimes, using a lambda function instead of a nested function declared with the def keyword can express the programmer’s intent more clearly. But to be honest, this isn’t a common occurrence—at least not in the kind of code that I like to write. So let’s talk a little more about that.

<h5>But Maybe You Shouldn’t…</h5>

On the one hand, I’m hoping this chapter got you interested in explor- ing Python’s lambda functions. On the other hand, I feel like it’s time to put up another caveat: Lambda functions should be used sparingly and with extraordinary care.
I know I’ve written my fair share of code using lambdas that looked “cool” but were actually a liability for me and my coworkers. If you’re tempted to use a lambda,  spend a few seconds (or minutes) to think  if it is really the cleanest and most maintainable way to achieve the desired result.
For example, doing something like this to save two lines of code is just silly. Sure, technically it works and it’s a nice enough “trick.” But it’s also going to confuse the next gal or guy that has to ship a bugfix under a tight deadline:

``` python
# Harmful:
>>> class Car:
...    rev = lambda self: print('Wroom!')
...    crash = lambda self: print('Boom!')

>>> my_car = Car()
>>> my_car.crash()
'Boom!'
```

I have similar feelings about complicated map() or filter() con- structs using lambdas. Usually it’s much cleaner to go with a list comprehension or generator expression:

``` python
# Harmful:
>>> list(filter(lambda x: x % 2 == 0, range(16))) 
[0, 2, 4, 6, 8, 10, 12, 14]

# Better:
>>> [x for x in range(16) if x % 2 == 0] 
[0, 2, 4, 6, 8, 10, 12, 14]
```

If you find yourself doing anything remotely complex with lambda expressions, consider defining a standalone function with a proper name instead.
Saving a few keystrokes won’t matter in the long run, but your col- leagues (and your future self) will appreciate clean and readable code more than terse wizardry.

<h5>Key Takeaways</h5>

   •	Lambda functions are single-expression functions that are not necessarily bound to a name (anonymous).
   
   •	Lambda functions can’t use regular Python statements and al- ways include an implicit return statement.
   
   •	Always ask yourself: Would using a regular (named) function or a list comprehension offer more clarity?
   


<h3>3.3	The Power of Decorators</h3>

At their core, Python’s decorators allow you to extend and modify the behavior of a callable (functions, methods, and classes) without per- manently modifying the callable itself.
Any sufficiently generic functionality you can tack on to an existing class or function’s behavior makes a great use case for decoration. This includes the following:

   •	logging
   •	enforcing access control and authentication
   •	instrumentation and timing functions
   •	rate-limiting
   •	caching, and more


Now, why should you master the use of decorators in Python? After all, what I just mentioned sounded quite abstract, and it might be diffi- cult to see how decorators can benefit you in your day-to-day work as a Python developer. Let me try to bring some clarity to this question by giving you a somewhat real-world example:
Imagine you’ve got 30 functions with business logic in your report- generating program. One rainy Monday  morning  your  boss  walks up to your desk and says: “Happy Monday! Remember those TPS reports? I need you to add input/output logging to each step in the report generator. XYZ Corp needs it for auditing purposes. Oh, and I told them we can ship this by Wednesday.”

Depending on whether or not you’ve got a solid grasp on Python’s dec- orators, this request will either send your blood pressure spiking or leave you relatively calm.
Without decorators you might be spending the next three days scram- bling to modify each of those 30 functions and clutter them up with manual logging calls. Fun times, right?
 

If you do know your decorators however, you’ll calmly smile at your boss and say: “Don’t worry Jim, I’ll get it done by 2pm today.”
Right after that you’ll type the code for a generic @audit_log decora- tor (that’s only about 10 lines long) and quickly paste it in front of each function definition.  Then you’ll commit your code and grab another cup of coffee…

I’m dramatizing here, but only a little. Decorators can be that power- ful. I’d go as far as to say that understanding decorators is a milestone for any serious Python programmer. They require a solid grasp of sev- eral advanced concepts in the language, including the properties of first-class functions.

<h5>I believe that the payoff for understanding how decorators work in Python can be enormous.</h5>

Sure, decorators are relatively complicated to wrap your head around for the first time, but they’re a highly useful feature that you’ll often encounter in third-party frameworks and the Python standard library. Explaining decorators is also a make or break moment for any good Python tutorial. I’ll do my best here to introduce you to them step by step.

Before you dive in however, now would be an excellent moment to re- fresh your memory on the properties of first-class functions in Python. There’s a chapter on them in this book, and I would encourage you to take a few minutes to review it. The most important “first-class func- tions” takeaways for understanding decorators are:

   •	<h5>Functions are objects</h5> —they can be assigned to variables and passed to and returned from other functions
   •	<h5>Functions can be defined inside other functions</h5> —and a child function can capture the parent function’s local state (lex- ical closures)

Alright, are you ready to do this? Let’s get started.
 

<h5>Python Decorator Basics</h5>

Now, what are decorators really? They “decorate” or “wrap” another function and let you execute code before and after the wrapped func- tion runs.
Decorators allow you to define reusable building blocks that can change or extend the behavior of other functions. And,  they let you  do that without permanently modifying the wrapped function itself. The function’s behavior changes only when it’s decorated.
What might the implementation of a simple decorator look like? In basic terms, a decorator is a callable that takes a callable as input and returns another callable.
The following function has that property and could be considered the simplest decorator you could possibly write:

``` python
def null_decorator(func):
  return func
```

As you can see, null_decorator is a callable (it’s a function), it takes another callable as its input, and it returns the same input callable without modifying it.

Let’s use it to decorate (or wrap) another function:

``` python
def greet():
    return 'Hello!'

greet = null_decorator(greet)

>>> greet()
'Hello!'
```

In this example, I’ve defined a greet function and then immediately decorated it by running it through the null_decorator function. I
 

know this doesn’t look very useful yet. I mean, we specifically de- signed the null decorator to be useless, right? But in a moment this ex- ample will clarify how Python’s special-case decorator syntax works.
Instead of explicitly calling null_decorator on greet and then reas- signing the greet variable, you can use Python’s @ syntax for decorat- ing a function more conveniently:

``` python
@null_decorator
def greet():
    return 'Hello!'

>>> greet()
'Hello!'
```

Putting an @null_decorator line in front of the function definition is the same as defining the function first and then running through the decorator. Using the @ syntax is just syntactic sugar and a shortcut for this commonly used pattern.

Note that using the @ syntax decorates the function immediately at definition time. This makes it difficult to access the undecorated orig- inal without brittle hacks. Therefore you might choose to decorate some functions manually in order to retain the ability to call the un- decorated function as well.

<h5>Decorators Can Modify Behavior</h5>

Now that you’re a little more familiar with the decorator syntax, let’s write another decorator that actually does something and modifies the behavior of the decorated function.
Here’s a slightly more complex decorator which converts the result of the decorated function to uppercase letters:

``` python
def uppercase(func):
    def wrapper(): original_result = func()
        modified_result = original_result.upper()
        return modified_result
    return wrapper
```

Instead of simply returning the input function like the null decora-  tor did, this uppercase decorator defines a new function on the fly (a closure) and uses it to wrap the input function in order to modify its behavior at call time.

The wrapper closure has access to the undecorated input function and it is free to execute additional code before and after calling the input function. (Technically, it doesn’t even need to call the input function at all.)

Note how, up until now, the decorated function has never been exe- cuted. Actually calling the input function at this point wouldn’t make any sense—you’ll want the decorator to be able to modify the behavior of its input function when it eventually gets called.
You might want to let that sink in for a minute or two. I know how complicated this stuff can seem, but we’ll get it sorted out together, I promise.
Time to see the uppercase decorator in action. What happens if you decorate the original greet function with it?

``` python
@uppercase
def greet():
    return 'Hello!'

>>> greet()
'HELLO!'
```

I hope this was the result you expected. Let’s take a closer look at what just happened here. Unlike null_decorator, our uppercase decora- tor returns a different function object when it decorates a function:

``` python
>>> greet
<function greet at 0x10e9f0950>

>>> null_decorator(greet)
<function greet at 0x10e9f0950>

>>> uppercase(greet)
<function uppercase.<locals>.wrapper at 0x76da02f28>
```

And as you saw earlier, it needs to do that in order to modify the behavior of the decorated function when it finally gets called. The uppercase decorator is a function itself. And the only way to influence the “future behavior” of an input function it decorates is to replace (or wrap) the input function with a closure.
That’s why uppercase defines and returns another function (the clo- sure) that can then be called at a later time, run the original input function, and modify its result.

Decorators modify the behavior of a callable through a wrapper clo- sure so you don’t have to permanently modify the original. The orig- inal callable isn’t permanently modified—its behavior changes only when decorated.
This let’s you tack on reusable building blocks, like logging and other instrumentation, to existing functions and classes. It makes decora- tors such a powerful feature in Python that it’s frequently used in the standard library and in third-party packages.

<h5>A Quick Intermission</h5>

By the way, if you feel like you need a quick coffee break or a walk around the block at this point—that’s totally normal. In my opinion
closures and decorators are some of the most difficult concepts to un- derstand in Python.
Please, take your time and don’t worry about figuring this out imme- diately. Playing through the code examples in an interpreter session one by one often helps make things sink in.
I know you can do it!

<h5>Applying Multiple Decorators to a Function</h5>

Perhaps not surprisingly, you can apply more than one decorator to a function. This accumulates their effects and it’s what makes decora- tors so helpful as reusable building blocks.
Here’s an example. The following two decorators wrap the output string of the decorated function in HTML tags. By looking at how the tags are nested, you can see which order Python uses to apply multiple decorators:

``` python
def strong(func):
    def wrapper():
        return '<strong>' + func() + '</strong>'
    return wrapper

def emphasis(func):
    def wrapper():
        return '<em>' + func() + '</em>'
return wrapper
```

Now let’s take these two decorators and apply them to our greet func- tion at the same time. You can use the regular @ syntax for that and just “stack” multiple decorators on top of a single function:

``` python
@strong
@emphasis

def greet():
    return 'Hello!'
```

What output do you expect to see if you run the decorated function? Will the @emphasis decorator add its <em> tag first, or does @ strong have precedence? Here’s what happens when you call the decorated function:

``` python
>>> greet()
'<strong><em>Hello!</em></strong>'
```
 
This clearly shows in what order the decorators were applied: from bottom to top. First, the input function was wrapped by the @emphasis decorator, and then the resulting (decorated) function got wrapped again by the @strong decorator.
To help me remember this bottom to top order, I like to call this be- havior decorator stacking. You start building the stack at the bottom and then keep adding new blocks on top to work your way upwards.
If you break down the above example and avoid the @ syntax to apply the decorators, the chain of decorator function calls looks like this:

``` python
decorated_greet = strong(emphasis(greet))	
```

Again, you can see that the emphasis decorator is applied first and then the resulting wrapped function is wrapped again by the strong decorator.

This also means that deep levels of decorator stacking will evenutally have an effect on performance because they keep adding nested function calls.  In  practice,  this  usually  won’t  be  a  problem,  but it’s something to keep in mind if you’re working on performance- intensive code that frequently uses decoration.
 

<h5>Decorating Functions That Accept Arguments</h5>
 
All examples so far only decorated a simple nullary greet function that didn’t take any arguments whatsoever. Up until now, the deco- rators you saw here didn’t have to deal with forwarding arguments to the input function.

If you try to apply one of these decorators to a function that takes ar- guments, it will not work correctly. How do you decorate a function that takes arbitrary arguments?
This is where Python’s *args and **kwargs feature3 for dealing with variable numbers of arguments comes in handy. The following proxy decorator takes advantage of that:

``` python
def proxy(func):
    def wrapper(*args, **kwargs): 
        return func(*args, **kwargs)
return wrapper
```
 
There are two notable things going on with this decorator:

   •	It uses the   and   operators in the wrapper closure definition to collect all positional and keyword arguments and stores them in variables (args and kwargs).
   •	The wrapper closure then forwards the collected arguments to the original input function using the and “argument un- packing” operators.

It’s a bit unfortunate that the meaning of the star and double-star op- erators is overloaded and changes depending on the context they’re used in, but I hope you get the idea.

Let’s expand the technique laid out by the proxy decorator into a more useful practical example. Here’s a trace decorator that logs function arguments and results during execution time:
 
``` python
def trace(func):
    def wrapper(*args, **kwargs):
        print(f'TRACE: calling {func. name }() ' 
            f'with {args}, {kwargs}')

        original_result = func(*args, **kwargs)

        print(f'TRACE: {func. name }() ' 
            f'returned {original_result!r}')

        return original_result
    return wrapper
```
 
Decorating a function with trace and then calling it will print the ar- guments passed to the decorated function and its return value.  This  is still somewhat of a “toy” example—but in a pinch it makes a great debugging aid:
 
``` python
@trace
def say(name, line):
    return f'{name}: {line}'

>>> say('Jane', 'Hello, World')
'TRACE: calling say() with ("Jane", "Hello, World"), {}' 'TRACE: say() returned "Jane: Hello, World"'
'Jane: Hello, World'
```
 
Speaking of debugging, there are some things you should keep in mind when debugging decorators:

<h5>How to Write “Debuggable” Decorators</h5>
 
When you use a decorator, really what you’re doing is replacing one function with another. One downside of this process is that it “hides” some of the metadata attached to the original (undecorated) function.
For example, the original function name, its docstring, and parameter list are hidden by the wrapper closure:

``` python
 def greet():
    """Return a friendly greeting."""
    return 'Hello!'

decorated_greet = uppercase(greet)
```
 
If you try to access any of that function metadata, you’ll see the wrap- per closure’s metadata instead:
 
``` python
>>> greet.  name 	
'greet'
>>> greet.  doc 	
'Return a friendly greeting.'

>>> decorated_greet.  name 	
'wrapper'
>>> decorated_greet.  doc 	 None
```
 
This makes debugging and working with the Python interpreter awkward and challenging. Thankfully there’s a quick fix for this: the functools.wraps decorator included in Python’s standard library.4
You can use functools.wraps in your own decorators to copy over the lost metadata from the undecorated function to the decorator closure. Here’s an example:

``` python
import functools

def uppercase(func): 
    @functools.wraps(func) 
    def wrapper():
        return func().upper()
    return wrapper
```
 
Applying functools.wraps to the wrapper closure returned by the decorator carries over the docstring and other metadata of the input function:
 
``` python
@uppercase
def greet():
    """Return a friendly greeting."""
    return 'Hello!'

>>> greet.  name 	
'greet'
>>> greet.  doc 	
'Return a friendly greeting.'
```
 
As a best practice, I’d recommend that you use functools.wraps in all of the decorators you write yourself. It doesn’t take much time and it will save you (and others) debugging headaches down the road.

Oh,  and congratulations—you’ve made it all the way to the end of  this complicated chapter and learned a whole lot about decorators in Python. Great job!

<h5>Key Takeaways</h5>
 
   •	Decorators define reusable building blocks you can apply to a callable to modify its behavior without permanently modifying the callable itself.
 
   •	The @ syntax is just a shorthand for calling the decorator on  an input function. Multiple decorators on a single function are applied bottom to top (decorator stacking).
 
   •	As a debugging best practice, use the functools.wraps helper in your own decorators to carry over metadata from the undec- orated callable to the decorated one.
 
   •	Just like any other tool in the software development toolbox, decorators are not a cure-all and they should not be overused. It’s important to balance the need to “get   stuff done” with the goal of “not getting tangled up in a horrible, unmaintainable mess of a code base.”

 
<h3>3.4	Fun With *args and **kwargs</h3>
 
I once pair-programmed with a smart Pythonista who would exclaim “argh!” and “kwargh!” every time he typed out a function definition with optional or keyword parameters.  We got along great otherwise.   I guess that’s what programming in academia does to people eventu- ally.
Now, while easily mocked, *args and **kwargs parameters are nev- ertheless a highly useful feature in Python. And understanding their potency will make you a more effective developer.

So what are *args and **kwargs parameters used for? They allow a function to accept optional arguments, so you can create flexible APIs in your modules and classes:

``` python
def foo(required, *args, **kwargs): 
    print(required)
    if args:
        print(args)
    if kwargs:
        print(kwargs)
```
 
The above function requires at least one argument called “required,” but it can accept extra positional and keyword arguments as well.

If we call the function with additional arguments, args will collect extra positional arguments as a tuple because the parameter name has a * prefix.

Likewise, kwargs will collect extra keyword arguments as a dictionary because the parameter name has a ** prefix.

Both args and kwargs can be empty if no extra arguments are passed to the function.

As we call the function with various combinations of arguments, you’ll see how Python collects them inside the args and kwargs parameters
according to whether they’re positional or keyword arguments:
 
``` python
>>> foo()
TypeError:
"foo() missing 1 required positional arg: 'required'"

>>> foo('hello') hello

>>> foo('hello', 1, 2, 3) hello
(1, 2, 3)

>>> foo('hello', 1, 2, 3, key1='value', key2=999) hello
(1, 2, 3)
{'key1': 'value', 'key2': 999}
```
 
I want to make it clear that calling the parameters args and kwargs is simply a naming convention. The previous example would work just as well if you called them  parms and  argv.  The actual syntax is   just the asterisk ( ) or double asterisk ( ), respectively.

However, I recommend that you stick with the accepted naming con- vention to avoid confusion. (And to get a chance to yell “argh!” and “kwargh!” every once in a while.)

<h5>Forwarding Optional or Keyword Arguments</h5>
 
It’s possible to pass optional or keyword parameters from one func- tion to another. You can do so by using the argument-unpacking op- erators and when calling the function you want to forward argu- ments to.

This also gives you an opportunity to modify the arguments before you pass them along. Here’s an example:

``` python
def foo(x, *args, **kwargs): 
    kwargs['name'] = 'Alice' 
    new_args = args + ('extra', ) 
    bar(x, *new_args, **kwargs)
```
 
This technique can be useful for subclassing and writing wrapper func- tions. For example, you can use it to extend the behavior of a parent class without having to replicate the  full signature of  its constructor in the child class. This can be quite convenient if you’re working with an API that might change outside of your control:
 
``` python
class Car:
    def init (self, color, mileage): 
        self.color = color
        self.mileage = mileage

class AlwaysBlueCar(Car):
    def init (self, *args, **kwargs): 
        super(). init (*args, **kwargs) 
        self.color = 'blue'

>>> AlwaysBlueCar('green', 48392).color
'blue'
```
 
The AlwaysBlueCar constructor simply passes on all arguments to its parent class and then overrides an internal attribute. This means  if the parent class constructor changes, there’s a good chance that AlwaysBlueCar would still function as intended.
The downside here is that the AlwaysBlueCar constructor now has a rather unhelpful signature—we don’t know what arguments it expects without looking up the parent class.
 

Typically you wouldn’t use this technique with your own class hierar- chies. The more likely scenario would be that you’ll want to modify or override behavior in some external class which you don’t control.
But this is always dangerous territory, so best be careful (or you might soon have yet another reason to scream “argh!”).
One more scenario where this technique is potentially helpful is writing wrapper functions such as decorators. There you typically  also want to accept arbitrary arguments to be passed through to the wrapped function.
And, if we can do it without having to copy and paste the original func- tion’s signature, that might be more maintainable:
 
``` python
def trace(f):
    @functools.wraps(f)
    def decorated_function(*args, **kwargs): 
        print(f, args, kwargs)
        result = f(*args, **kwargs) print(result)
    return decorated_function

@trace
def greet(greeting, name):
    return '{}, {}!'.format(greeting, name)

>>> greet('Hello', 'Bob')
<function greet at 0x1031c9158> ('Hello', 'Bob') {}
'Hello, Bob!'
```

With techniques like this one, it’s sometimes difficult to balance the idea of making your code explicit enough and yet adhere to the Don’t Repeat Yourself (DRY) principle. This will always be a tough choice to make. If you can get a second opinion from a colleague, I’d encourage you to ask for one.
 

 <h5>Key Takeaways</h5>
 
   •		args and	kwargs let you write functions with a variable number of arguments in Python
 
   •	args collects extra positional arguments as a tuple.	kwargs
   collects the extra keyword arguments as a dictionary.
 
   •	The actual syntax is	and	. Calling them args and kwargs is just a convention (and one you should stick to).

 
<h3>3.5	Function Argument Unpacking</h3>
 
A really cool but slightly arcane feature is the ability to “unpack” func- tion arguments from sequences and dictionaries with the	* and ** operators.
Let’s define a simple function to work with as an example:

``` python
def print_vector(x, y, z):
    print('<%s, %s, %s>' % (x, y, z))
```
 
As you can see, this function takes three arguments (x, y, and z) and prints them in a nicely formatted way. We might use this function to pretty-print 3-dimensional vectors in our program:
 
``` python
>>> print_vector(0, 1, 0)
<0, 1, 0>
```
 
Now depending on which data structure we choose to represent 3D vectors with, printing them with our print_vector function might feel a little awkward. For example, if our vectors are represented as tuples or lists we must explicitly specify the index for each component when printing them:
 
``` python
>>> tuple_vec = (1, 0, 1)
>>> list_vec = [1, 0, 1]
>>> print_vector(tuple_vec[0],
                 tuple_vec[1], 
                 tuple_vec[2])
<1, 0, 1>
```
 
Using a normal function call with separate arguments seems unneces- sarily verbose and cumbersome. Wouldn’t it be much nicer if we could just “explode” a vector object into its three components and pass ev- erything to the print_vector function all at once?
 

(Of course, you could simply redefine  print_vector so that it takes a single parameter representing a vector object—but for the sake of having a simple example, we’ll ignore that option for now.)

Thankfully, there’s a better way to handle this situation in Python with
Function Argument Unpacking using the * operator:

``` python
>>> print_vector(*tuple_vec)
<1, 0, 1>
>>> print_vector(*list_vec)
<1, 0, 1>
```
 
Putting a before an iterable in a function call will unpack it and pass its elements as separate positional arguments to the called function.
This technique works for any iterable, including generator expres- sions.  Using the  operator on a generator consumes all elements   from the generator and passes them to the function:

``` python
>>> genexpr = (x * x for x in range(3))
>>> print_vector(*genexpr)
```
 
Besides the * operator for unpacking sequences like tuples, lists, and generators into positional arguments, there’s also the ** operator for unpacking keyword arguments from dictionaries. Imagine our vector was represented as the following dict object:
 
``` python
>>> dict_vec = {'y': 0, 'z': 1, 'x': 1}
```
 
We could pass this dict to print_vector in much the same way using the	operator for unpacking:

``` python
>>> print_vector(**dict_vec)
<1, 0, 1>
```
 
Because dictionaries are unordered, this matches up dictionary values and function arguments based on the dictionary keys: the x argument receives the value associated with the 'x' key in the dictionary.
If you were to use the single asterisk ( ) operator to unpack the dictio- nary, keys would be passed to the function in random order instead:

``` python
>>> print_vector(*dict_vec)
<y, x, z>
```
 
Python’s function argument unpacking feature gives you a lot of flex- ibility for free. Often this means you won’t have to implement a class for a data type needed by your program. As a result, using simple built-in data structures like tuples or lists will suffice and help reduce the complexity of your code.

 <h5>Key Takeaways</h5>
 
   •	The	and	operators can be used to “unpack” function argu- ments from sequences and dictionaries.
 
   •	Using argument unpacking effectively can help you write more flexible interfaces for your modules and functions.

 
<h3>3.6	Nothing to Return Here</h3>
 
Python adds an implicit return None statement to the end of any function. Therefore, if a function doesn’t specify a return value, it re- turns None by default.
This means you can replace return None statements with bare return statements or even leave them out completely and still get the same result:

``` python
def foo1(value):
    if value:
        return value
    else:
        return None

def foo2(value):
    """Bare return statement implies `return None`"""
    if value:
        return value
    else:
        return

def foo3(value):
    """Missing return statement implies `return None`"""
    if value:
        return value
```
 
All three functions properly return None if you pass them a falsy value as the sole argument:
 
``` python
>>> type(foo1(0))
<class 'NoneType'>

>>> type(foo2(0))
<class 'NoneType'>

>>> type(foo3(0))
<class 'NoneType'>
```
 
Now, when is it a good idea to use this feature in your own Python code?
My rule of thumb is that if a function doesn’t have a return value (other languages would call this a procedure), then I will leave out the return statement. Adding one would just be superfluous and confusing. An example for a procedure would be Python’s built-in print function which is only called for its side-effects (printing text) and never for its return value.

Let’s take a function like Python’s built-in sum. It clearly has a logi-  cal return value, and typically sum wouldn’t get called only for its side- effects. Its purpose is to add a sequence of numbers together and then deliver the result. Now, if a function does have a return value from a logical point of view, then you need to decide whether to use an im- plicit return or not.

On the one hand, you could argue that omitting an explicit return None statement makes the code more concise and therefore easier to read and understand. Subjectively, you might also say it makes the code “prettier.”

On the other hand, it might surprise some programmers that Python behaves this way. When it comes to writing clean and maintainable code, surprising behavior is rarely a good sign.
For example, I’ve been using an “implicit return statement” in one of the code samples in an earlier revision of the book. I didn’t mention what I was doing—I just wanted a nice short code sample to explain some other feature in Python.
Eventually I started getting a steady stream of emails pointing me to “the missing return statement” in that code example. Python’s implicit return behavior was clearly not obvious to everybody and was a dis-
 

traction in this case. I added a note to make it clear what was going on, and the emails stopped.
Don’t get me wrong—I love writing clean and “beautiful” code as much as anyone. And I also used to feel strongly that programmers should know the ins and outs of the language they’re working with.
But when you consider the maintenance impact of even such a simple misunderstanding, it might make sense to lean towards writing more explicit and clear code. After all, code is communication.

 <h5>Key Takeaways</h5>
 
   •	If a function doesn’t specify a return value, it returns None. Whether to explicitly return None is a stylistic decision.
 
   •	This is a core Python feature but your code might communicate its intent more clearly with an explicit return None statement.
