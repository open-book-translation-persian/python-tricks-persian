
<h1>Patterns for Cleaner Python</h1>
  
<h3 dir='rt1'>2.1 Covering Your A** With Assertions</h3>
  
Sometimes a genuinely helpful language feature gets less attention than it deserves. For some reason, this is what happened to Python’s built-inassert statement.In this chapter I’m going to give you an introduction to using asser-tions in Python. You’ll learn how to use them to help automaticallydetect errors in your Python programs. This will make your programsmore reliable and easier to debug.At this point, you might be wondering “What are assertions and whatare they good for?” Let’s get you some answers for that.At its core, Python’s assert statement is a debugging aid that tests acondition. If the assert condition is true, nothing happens, and yourprogram continues to execute as normal. But if the condition evalu-ates to false, an Assertion Error exception is raised with an optional error message.

<h4 dir='rt1'>Assert in Python — An Example</h4>
  
Here’s a simple example so you can see where assertions might comein handy. I tried to give this some semblance of a real-world problem you might actually encounter in one of your programs.Suppose you were building an online store with Python. You’re work-ing to add a discount coupon functionality to the system, and eventu-ally you write the following apply_discount function:
  
``` python
def apply_discount(product, discount):
    price=int(product['price']*(1.0-discount))
    assert0<=price<=product['price']
    return price
```
Notice the assert statement in there? It will guarantee that, no mat-terwhat,discounted prices calculated by this function cannot be lower
$0 and they cannot be higher than the original price of the prod-uct.Let’s make sure this actually works as intended if we call this functionto apply a valid discount. In this example, products for our store willbe represented as plain dictionaries. This is probably not what you’ddo for a real application, but it’ll work nicely for demonstrating asser-tions. Let’s create an example product—a pair of nice shoes at a priceof $149.00:

``` python
>>> shoes={'name':'Fancy Shoes','price':14900}
```
  
By the way, did you notice how I avoided currency rounding issuesby using an integer to represent the price amount in cents? That’sgenerally a good idea... But I digress. Now, if we apply a 25% discountto these shoes, we would expect to arrive at a sale price of $111.75:
``` python
>>> apply_discount(shoes,0.25)
11175
```
Alright, this worked nicely. Now, let’s try to apply some invalid dis-counts. For example, a 200% “discount” that would lead to us givingmoney to the customer:
``` python
>>> apply_discount(shoes,2.0)
Traceback (most recent call last):
  File"<input>", line1,in<module>
    apply_discount(prod,2.0)
  File"<input>", line4,inapply_discount
    assert0<=price<=product['price']
  AssertionError
```
As you can see, when we try to apply this invalid discount, ourprogram halts with an AssertionError. This happens because adiscount of 200% violated the assertion condition we placed in theapply_discount function.
  
You can also see how the exception stacktrace point sout the exact line of code containing the failed assertion. If you (or another developeron your team) ever encounter one of these errors while testing theonline store, it will be easy to find out what happened just by lookingat the exception traceback.This speeds up debugging efforts considerably, and it will make your programs more maintainable in the long-run. And that, my friend, isthe power of assertions.

<h5 dir='rt1'>Why Not Just Use a Regular Exception?</h5>
  
Now, you’re probably wondering why I didn’t just use an if-statement
and an exception in the previous example…
You see, the proper use of assertions is to inform developers about
unrecoverable errors in a program. Assertions are not intended to
signal expected error conditions, like a File-Not-Found error, where
a user can take corrective actions or just try again.
Assertions are meant to be internal self-checks for your program. They
work by declaring some conditions as impossible in your code. If one
of these conditions doesn’t hold, that means there’s a bug in the program.
If your program is bug-free, these conditions will never occur. But if
they do occur, the program will crash with an assertion error telling
you exactly which “impossible” condition was triggered. This makes
it much easier to track down and fix bugs in your programs. And I like
anything that makes life easier—don’t you?
For now, keep in mind that Python’s assert statement is a debugging
aid, not a mechanism for handling run-time errors. The goal of using
assertions is to let developers find the likely root cause of a bug more
quickly. An assertion error should never be raised unless there’s a bug
in your program.
Let’s take a closer look at some other things we can do with assertions,
and then I’ll cover two common pitfalls when using them in real-world
scenarios.
  
<h5 dir='rt1'>Python’s Assert Syntax</h5>
  
It’s always a good idea to study up on how a language feature is actually implemented in Python before you start using it. So let’s take
a quick look at the syntax for the assert statement, according to the
Python docs:
  
``` python
assert_stmt ::= "assert" expression1 ["," expression2]
```
  
In this case, expression1 is the condition we test, and the optional
expression2 is an error message that’s displayed if the assertion fails.
At execution time, the Python interpreter transforms each assert statement into roughly the following sequence of statements:

``` python
if __debug__:
    if not expression1:
        raise AssertionError(expression2)
```

Two interesting things about this code snippet:
Before the assert condition is checked, there’s an additional check for
the __debug__ global variable. It’s a built-in boolean flag that’s true
under normal circumstances and false if optimizations are requested.
We’ll talk some more about later that in the “common pitfalls” section.
Also, you can use expression2 to pass an optional error message that
will be displayed with the AssertionError in the traceback. This can
simplify debugging even further. For example, I’ve seen code like this:
  
``` python
>>> if cond == 'x':
        do_x()

    elif cond == 'y':
        do_y()

    else:
        assert False, (
        'This should never happen, but it does '
        'occasionally. We are currently trying to '
        'figure out why. Email dbader if you '
        'encounter this in the wild. Thanks!')
````
  
Is this ugly? Well, yes. But it’s definitely a valid and helpful technique
if you’re faced with a Heisenbug2 in one of your applications.

<h5 dir='rt1'>Common Pitfalls With Using Asserts in Python</h5>
  
Before you move on, there are two important caveats regarding the
use of assertions in Python that I’d like to call out.
The first one has to do with introducing security risks and bugs into
your applications, and the second one is about a syntax quirk that
makes it easy to write useless assertions.
This sounds (and potentially is) quite horrible, so you should probably
at least skim these two caveats below.
  
<h5 dir='rt1'>Caveat #1 – Don’t Use Asserts for Data Validation</h5>
  
The biggest caveat with using asserts in Python is that assertions canbe globally disabled3with the-Oand-OOcommand line switches, aswell as thePYTHONOPTIMIZEenvironment variable in CPython.This turns any assert statement into a null-operation: the assertionssimply get compiled away and won’t be evaluated, which means thatnone of the conditional expressions will be executed.
  
This is an intentional design decision used similarly by many other
programming languages. As a side-effect, it becomes extremely dangerous to use assert statements as a quick and easy way to validate
input data.
Let me explain—if your program uses asserts to check if a function
argument contains a “wrong” or unexpected value, this can backfire
quickly and lead to bugs or security holes.
Let’s take a look at a simple example that demonstrates this problem. Again, imagine you’re building an online store application with
Python. Somewhere in your application code there’s a function to
delete a product as per a user’s request.
Because you just learned about assertions, you’re eager to use them
in your code (hey, I know I would be!) and you write the following
implementation:
  
``` python
def delete_product(prod_id, user):
    assert user.is_admin(), 'Must be admin'
    assert store.has_product(prod_id), 'Unknown product'
    store.get_product(prod_id).delete()
```
  
Take a close look at this delete_product function. Now, what’s going
to happen if assertions are disabled?
There are two serious issues in this three-line function example, and
they’re caused by the incorrect use of assert statements:
  
1. Checking for admin privileges with an assert statement is dangerous. If assertions are disabled in the Python
interpreter, this turns into a null-op. Therefore any user can
now delete products. The privileges check doesn’t even run.
This likely introduces a security problem and opens the door
for attackers to destroy or severely damage the data in our
online store. Not good.
  
2. The has_product() check is skipped when assertions
are disabled. This means get_product() can now be called
with invalid product IDs—which could lead to more severe
bugs, depending on how our program is written. In the worst
case, this could be an avenue for someone to launch Denial of
Service attacks against our store. For example, if the store app
crashes if someone attempts to delete an unknown product,
an attacker could bombard it with invalid delete requests and
cause an outage.
How might we avoid these problems? The answer is to never use assertions to do data validation. Instead, we could do our validation
with regular if-statements and raise validation exceptions if necessary, like so:

``` python
def delete_product(product_id, user):
    if not user.is_admin():
         raise AuthError('Must be admin to delete')
    if not store.has_product(product_id):
        raise ValueError('Unknown product id')
    store.get_product(product_id).delete()
```

This updated example also has the benefit that instead of raising unspecific AssertionError exceptions, it now raises semantically correct exceptions like ValueError or AuthError (which we’d have to
define ourselves.)

<h5 dir='rt1'>Caveat #2 – Asserts That Never Fail</h5>
  
It’s surprisingly easy to accidentally write Python assert statements
that always evaluate to true. I’ve been bitten by this myself in the past.
Here’s the problem, in a nutshell:
When you pass a tuple as the first argument in an assert statement,
the assertion always evaluates as true and therefore never fails.

For example, this assertion will never fail:

``` python
assert(1 == 2, 'This should fail')
```

This has to do with non-empty tuples always being truthy in Python. If
you pass a tuple to an assert statement, it leads to the assert condition
always being true—which in turn leads to the above assert statement
being useless because it can never fail and trigger an exception.
It’s relatively easy to accidentally write bad multi-line asserts due to
this, well, unintuitive behavior. For example, I merrily wrote a bunch
of broken test cases that gave a false sense of security in one of my test
suites. Imagine you had this assertion in one of your unit tests:
  
``` python
assert (
counter == 10,
'It should have counted all the items'
)
```

Upon first inspection, this test case looks completely fine. However, it
would never catch an incorrect result: the assertion always evaluates
to True, regardless of the state of the counter variable. And why is
that? Because it asserts the truth value of a tuple object.
Like I said, it’s rather easy to shoot yourself in the foot with this (mine
still hurts). A good countermeasure you can apply to prevent this syntax quirk from causing trouble is to use a code linter.4 Newer versions
of Python 3 will also show a syntax warning for these dubious asserts.
By the way, that’s also why you should always do a quick smoke test
with your unit test cases. Make sure they can actually fail before you
move on to writing the next one.

  
<h5 dir='rt1'>Python Assertions — Summary</h5>
  
Despite these caveats I believe that Python’s assertions are a powerful
debugging tool that’s frequently underused by Python developers.
Understanding how assertions work and when to apply them can help
you write Python programs that are more maintainable and easier to
debug.
It’s a great skill to learn that will help bring your Python knowledge to
the next level and make you a more well-rounded Pythonista. I know
it has saved me hours upon hours of debugging.

<h5 dir='rt1'>Key Takeaways</h5>
  
   • Python’s assert statement is a debugging aid that tests a condition as an internal self-check in your program.
  
   • Asserts should only be used to help developers identify bugs.
   They’re not a mechanism for handling run-time errors.
  
   • Asserts can be globally disabled with an interpreter setting.


<h3 dir='rt1'>2.2 Complacent Comma Placement</h3>

Here’s a handy tip for when you’re adding and removing items from
a list, dict, or set constant in Python: Just end all of your lines with a
comma.
Not sure what I’m talking about? Let me give you a quick example.
Imagine you’ve got this list of names in your code:

``` python
>>> names = ['Alice', 'Bob', 'Dilbert']
```

Whenever you make a change to this list of names, it’ll be hard to tell
what was modified by looking at a Git diff, for example. Most source
control systems are line-based and have a hard time highlighting multiple changes to a single line.
A quick fix for that is to adopt a code style where you spread out list,
dict, or set constants across multiple lines, like so:

``` python
>>> names = [
...'Alice',
...'Bob',
...'Dilbert'
... ]
```

That way there’s one item per line, making it perfectly clear which one
was added, removed, or modified when you view a diff in your source
control system. It’s a small change but I found it helped me avoid silly
mistakes. It also made it easier for my teammates to review my code
changes.
Now, there are two editing cases that can still cause some confusion.
Whenever you add a new item at the end of a list, or you remove the
last item, you’ll have to update the comma placement manually to get
consistent formatting.

Let’s say you’d like to add another name (Jane) to that list. If you add
Jane, you’ll need to fix the comma placement after the Dilbert line to
avoid a nasty error:

``` python
>>> names = [...'Alice',
...'Bob',
...'Dilbert' # <- Missing comma!
...'Jane']
```

When you inspect the contents of that list, brace yourself for a sur-prise:

``` python
>>> names 
['Alice','Bob','DilbertJane']
```

As you can see, Python merged the strings Dilbert and Jane into DilbertJane. This so-called “string literal concatenation” is intentional
and documented behavior. And it’s also a fantastic way to shoot yourself in the foot by introducing hard-to-catch bugs into your programs:

     "Multiple adjacent string or bytes literals (delimited by
      whitespace), possibly using different quoting conventions, are allowed, and their meaning is the same as
      their concatenation.”
      
Still, string literal concatenation is a useful feature in some cases. For
example, you can use it to reduce the number of backslashes needed
to split long string constants across multiple lines:

```python
my_str = ('This is a super long string constant '
'spread out across multiple lines. '
'And look, no backslash characters needed!')
```

On the other hand, we’ve just seen how the same feature can quickly
turn into a liability. Now, how do we fix this situation?
Adding the missing comma after Dilbert prevents the two strings from
getting merged into one:

``` python
>>> names = [...'Alice',
...'Bob',
...'Dilbert',
...'Jane']
```

But now we’ve come full circle and returned to the original problem.
I had to modify two lines in order to add a new name to the list. This
makes it harder to see what was modified in the Git diff again… Did
someone add a new name? Did someone change Dilbert’s name?
Luckily, Python’s syntax allows for some leeway to solve this comma
placement issue once and for all. You just need to train yourself to
adopt a code style that avoids it in the first place. Let me show you
how.
In Python, you can place a comma after every item in a list, dict, or set
constant, including the last item. That way, you can just remember
to always end your lines with a comma and thus avoid the comma
placement juggling that would otherwise be required.
Here’s what the final example looks like:

``` python
>>>names=[...'Alice',
...'Bob',
...'Dilbert',
... ]
```

Did you spot the comma after Dilbert? That’ll make it easy to add or
remove new items without having to update the comma placement. It
keeps your lines consistent, your source control diffs clean, and your
code reviewers happy. Hey, sometimes the magic is in the little things,
right?

<h5 dir='rt1'>Key Takeaways</h5>

   • Smart formatting and comma placement can make your list,
   dict, or set constants easier to maintain.

   • Python’s string literal concatenation feature can work to your
   benefit, or introduce hard-to-catch bugs.


<h3 dir='rt1'>2.3 Context Managers and the with Statement</h3>

The with statement in Python is regarded as an obscure feature by
some. But when you peek behind the scenes, you’ll see that there’s no
magic involved, and it’s actually a highly useful feature that can help
you write cleaner and more readable Python code.
So what’s the with statement good for? It helps simplify some common resource management patterns by abstracting their functionality
and allowing them to be factored out and reused.
A good way to see this feature used effectively is by looking at examples in the Python standard library. The built-in open() function provides us with an excellent use case:

``` python
with open('hello.txt', 'w') as f:
    f.write('hello, world!')
```

Opening files using the with statement is generally recommended because it ensures that open file descriptors are closed automatically after program execution leaves the context of the with statement. Internally, the above code sample translates to something like this:

``` python
f = open('hello.txt', 'w')
try:
    f.write('hello, world')
finally:
    f.close()
```

You can already tell that this is quite a bit more verbose. Note that
the try...finally statement is significant. It wouldn’t be enough to
just write something like this:

```python
f = open('hello.txt', 'w')
f.write('hello, world')
f.close()
```

This implementation won’t guarantee the file is closed if there’s an exception during the f.write() call—and therefore our program might
leak a file descriptor. That’s why the with statement is so useful. It
makes properly acquiring and releasing resources a breeze.
Another good example where the with statement is used effectively in
the Python standard library is the threading.Lock class:

``` python
some_lock = threading.Lock()


# Harmful:
some_lock.acquire()
try:
    # Do something...
finally:
    some_lock.release()
    
# Better:
with some_lock:
    # Do something...
```

In both cases, using a with statement allows you to abstract away most
of the resource handling logic. Instead of having to write an explicit
try...finally statement each time, using the with statement takes
care of that for us.
The with statement can make code that deals with system resources
more readable. It also helps you avoid bugs or leaks by making it practically impossible to forget to clean up or release a resource when it’s
no longer needed.

<h5 dir='rt1'>Supporting with in Your Own Objects</h5>

Now, there’s nothing special or magical about the open() function or
the threading.Lock class and the fact that they can be used with a
with statement. You can provide the same functionality in your own
classes and functions by implementing so-called context managers.6
What’s a context manager? It’s a simple “protocol” (or interface) that
your object needs to follow in order to support the with statement.
Basically, all you need to do is add __enter__ and __exit__ methods
to an object if you want it to function as a context manager. Python
will call these two methods at the appropriate times in the resource
management cycle.
Let’s take a look at what this would look like in practical terms. Here’s
what a simple implementation of the open() context manager might
look like:

``` python
class ManagedFile:
    def __init__(self, name):
        self.name = name
    def __enter__(self):
        self.file = open(self.name, 'w')
        return self.file
    def __exit__(self, exc_type, exc_val, exc_tb):
        if self.file:
            self.file.close()
```

Our ManagedFile class follows the context manager protocol and now
supports the with statement, just like the original open() example
did:

``` python
>>> withManagedFile('hello.txt') as f:
...    f.write('hello, world!')
...    f.write('bye now')
```

Python calls __enter__ when execution enters the context of the

with statement and it’s time to acquire the resource. When execu-

tion leaves the context again, Python calls __exit__ to free up the
resource.

Writing a class-based context manager isn’t the only way to support
the with statement in Python. The contextlib7 utility module in the
standard library provides a few more abstractions built on top of the
basic context manager protocol. This can make your life a little easier
if your use cases match what’s offered by contextlib.
For example, you can use the contextlib.contextmanager decorator to define a generator-based factory function for a resource that will
then automatically support the with statement. Here’s what rewriting
our ManagedFile context manager example with this technique looks
like:

``` python
from contextlib import contextmanager
@contextmanager
def managed_file(name):
    try:
        f = open(name, 'w')
        yield f
    finally:
        f.close()
>>> withmanaged_file('hello.txt') as f:
...     f.write('hello, world!')
...     f.write('bye now')
```

In this case, managed_file() is a generator that first acquires the
resource. After that, it temporarily suspends its own execution and
yields the resource so it can be used by the caller. When the caller
leaves the with context, the generator continues to execute so that any
remaining clean-up steps can occur and the resource can get released
back to the system.
The class-based implementation and the generator-based one are essentially equivalent. You might prefer one over the other, depending
on which approach you find more readable.
A downside of the @contextmanager-based implementation might be
that it requires some understanding of advanced Python concepts like
decorators and generators. If you need to get up to speed with those,
feel free to take a detour to the relevant chapters here in this book.
Once again, making the right implementation choice here comes
down to what you and your team are comfortable using and what you
find the most readable.

<h5 dir='rt1'>Writing Pretty APIs With Context Managers</h5>

Context managers are quite flexible, and if you use the with statement creatively, you can define convenient APIs for your modules and
classes.
For example, what if the “resource” we wanted to manage was text
indentation levels in some kind of report generator program? What if
we could write code like this to do it:
```python
with Indenter() as indent:
    indent.print('hi!')
    with indent:
        indent.print('hello')
        with indent:
            indent.print('bonjour')
    indent.print('hey')
```

This almost reads like a domain-specific language (DSL) for indenting text. Also, notice how this code enters and leaves the same context manager multiple times to change indentation levels. Running
this code snippet should lead to the following output and print neatly
formatted text to the console:

hi!
    hello
        bonjour
hey

So, how would you implement a context manager to support this functionality?
By the way, this could be a great exercise for you to understand exactly
how context managers work. So before you check out my implementation below, you might want to take some time and try to implement
this yourself as a learning exercise.
If you’re ready to check out my implementation, here’s how you might
implement this functionality using a class-based context manager:

``` python
class Indenter:
    def __init__(self):
        self.level = 0
    def __enter__(self):
        self.level += 1
        return self
    def __exit__(self, exc_type, exc_val, exc_tb):
        self.level -= 1
    def print(self, text):
        print('    ' * self.level + text)
```

That wasn’t so bad, was it? I hope that by now you’re already feeling
more comfortable using   in
your own Python programs. They’re an excellent feature that will allow you to deal with resource management in a much more Pythonic
and maintainable way.
If you’re looking for another exercise to deepen your understanding,
try implementing a context manager that measures the execution time
of a code block using the time.time function. Be sure to try out writing both a decorator-based and a class-based variant to drive home
the difference between the two.

<h5 dir='rt1'>Key Takeaways</h5>

   • The with statement simplifies exception handling by encapsulating standard uses of try/finally statements in so-called
   context managers.
   
   • Most commonly it is used to manage the safe acquisition and
   release of system resources. Resources are acquired by the
   with statement and released automatically when execution
   leaves the with context.
   
   • Using with effectively can help you avoid resource leaks and
   make your code easier to read.


<h3 dir='rt1'>2.4 Underscores, Dunders, and More</h3>

Single and double underscores have a meaning in Python variable and
method names. Some of that meaning is merely by convention and
intended as a hint to the programmer—and some of it is enforced by
the Python interpreter.
If you’re wondering, “What’s the meaning of single and double underscores in Python variable and method names?” I’ll do my best to get
you the answer here. In this chapter we’ll discuss the following five
underscore patterns and naming conventions, and how they affect the
behavior of your Python programs:

   Single Leading Underscore: _var
   
   Single Trailing Underscore: var_
   
   Double Leading Underscore: __var
   
   Double Leading and Trailing Underscore: __var__
   
   Single Underscore: _
   
   
<h5 dir='rt1'>1. Single Leading Underscore: “_var”</h5>

When it comes to variable and method names, the single underscore prefix has a meaning by convention only. It’s a hint to the
programmer—it means what the Python community agrees it should
mean, but it does not affect the behavior of your programs.
The underscore prefix is meant as a hint to tell another programmer
that a variable or method starting with a single underscore is intended
for internal use. This convention is defined in PEP 8, the most commonly used Python code style guide.8
However, this convention isn’t enforced by the Python interpreter.
Python does not have strong distinctions between “private” and
“public” variables like Java does. Adding a single underscore in front
of a variable name is more like someone putting up a tiny underscore
warning sign that says: “Hey, this isn’t really meant to be a part of the
public interface of this class. Best to leave it alone.”

Take a look at the following example:

``` python
class Test:
    def __init__(self):
        self.foo = 11
        self._bar = 23
```

What’s going to happen if you instantiate this class and try to access
the foo and _bar attributes defined in its __init__ constructor?

Let’s find out:

``` python
>>> t=Test()
>>>t.foo
11
>>>t._bar
23
```

As you can see, the leading single underscore in _bar did not prevent
us from “reaching into” the class and accessing the value of that variable.
That’s because the single underscore prefix in Python is merely an
agreed-upon convention—at least when it comes to variable and
method names. However, leading underscores do impact how names
get imported from modules. Imagine you had the following code in a
module called my_module:

``` python
# my_module.py:

def external_func():
    return 23
def _internal_func():
    return 42
```

Now, if you use a wildcard import to import all the names from the
module, Python will not import names with a leading underscore (unless the module defines an __all__ list that overrides this behavior9 ):

``` python
>>> from my_module import*
>>>external_func()
23
>>_internal_func()
NameError:"name '_internal_func' is not defined"
```

By the way, wildcard imports should be avoided as they make it unclear which names are present in the namespace. It’s better to stick
to regular imports for the sake of clarity. Unlike wildcard imports, regular imports are not affected by the leading single underscore naming
convention:

``` python
>>> import my_module
>>> my_module.external_func()
23
>>> my_module._internal_func()
42
```

I know this might be a little confusing at this point. If you stick to
the PEP 8 recommendation that wildcard imports should be avoided,
then all you really need to remember is this:
Single underscores are a Python naming convention that indicates a
name is meant for internal use. It is generally not enforced by the
Python interpreter and is only meant as a hint to the programmer.

<h5 dir='rt1'>2. Single Trailing Underscore: “var_”</h5>

Sometimes the most fitting name for a variable is already taken by a
keyword in the Python language. Therefore, names like class or def
cannot be used as variable names in Python. In this case, you can
append a single underscore to break the naming conflict:

``` python
>>> defmake_object(name,class):
SyntaxError:"invalid syntax"
>>> defmake_object(name, class_):
...pass
```

In summary, a single trailing underscore (postfix) is used by convention to avoid naming conflicts with Python keywords. This convention
is defined and explained in PEP 8.

<h5 dir='rt1'>3. Double Leading Underscore: “__var”</h5>

The naming patterns we’ve covered so far receive their meaning from
agreed-upon conventions only. With Python class attributes (variables and methods) that start with double underscores, things are a
little different.
A double underscore prefix causes the Python interpreter to rewrite
the attribute name in order to avoid naming conflicts in subclasses.
This is also called name mangling—the interpreter changes the name
of the variable in a way that makes it harder to create collisions when
the class is extended later.
I know this sounds rather abstract. That’s why I put together this little
code example we can use for experimentation:

```python
class Test:
    def __init__(self):
        self.foo = 11
        self._bar = 23
        self.__baz = 23
```

Let’s take a look at the attributes on this object using the built-in dir()
function:

``` python
>>> t = Test()
>>> dir(t)
['_Test__baz', '__class__', '__delattr__', '__dict__',
'__dir__', '__doc__', '__eq__', '__format__', '__ge__',
'__getattribute__', '__gt__', '__hash__', '__init__',
'__le__', '__lt__', '__module__', '__ne__', '__new__',
'__reduce__', '__reduce_ex__', '__repr__',
'__setattr__', '__sizeof__', '__str__',
'__subclasshook__', '__weakref__', '_bar', 'foo']
```

This gives us a list with the object’s attributes. Let’s take this list and
look for our original variable names foo, _bar, and __baz. I promise
you’ll notice some interesting changes.

First of all, the self.foo variable appears unmodified as foo in the
attribute list.

Next up, self._bar behaves the same way—it shows up on the class
as _bar. Like I said before, the leading underscore is just a convention
in this case—a hint for the programmer.

However, with self.__baz things look a little different. When you
search for __baz in that list, you’ll see that there is no variable with
that name.

So what happened to __baz?

If you look closely, you’ll see there’s an attribute called _Test__baz
on this object. This is the name mangling that the Python interpreter
applies. It does this to protect the variable from getting overridden in
subclasses.

Let’s create another class that extends the Test class and attempts to
override its existing attributes added in the constructor:

``` python
class ExtendedTest(Test):
    def __init__(self):
        super().__init__()
        self.foo = 'overridden'
        self._bar = 'overridden'
        self.__baz = 'overridden'
```

Now, what do you think the values of foo, _bar, and __baz will be on
instances of this ExtendedTest class? Let’s take a look:

``` python
>>> t2 = ExtendedTest()
>>> t2.foo
'overridden'
>>> t2._bar
'overridden'
>>> t2.__baz
AttributeError:
"'ExtendedTest' object has no attribute '__baz'"
```

Wait, why did we get that AttributeError when we tried to inspect
the value of t2.__baz? Name mangling strikes again! It turns out
this object doesn’t even have a __baz attribute:

``` python
>>> dir(t2)
['_ExtendedTest__baz', '_Test__baz', '__class__',
'__delattr__', '__dict__', '__dir__', '__doc__',
'__eq__', '__format__', '__ge__', '__getattribute__',
'__gt__', '__hash__', '__init__', '__le__', '__lt__',
'__module__', '__ne__', '__new__', '__reduce__',
'__reduce_ex__', '__repr__', '__setattr__',
'__sizeof__', '__str__', '__subclasshook__',
'__weakref__', '_bar', 'foo', 'get_vars']
```

As you can see, __baz got turned into _ExtendedTest__baz to prevent accidental modification. But the original _Test__baz is also still
around:

``` python
>>> t2._ExtendedTest__baz
'overridden'
>>> t2._Test__baz
42
```

Double underscore name mangling is fully transparent to the programmer. Take a look at the following example that will confirm
this:

``` python
class ManglingTest:
    def __init__(self):
        self.__mangled = 'hello'
    def get_mangled(self):
        return self.__mangled
        
>>> ManglingTest().get_mangled()
'hello'
>>> ManglingTest().__mangled
AttributeError:
"'ManglingTest' object has no attribute '__mangled'"
```

Does name mangling also apply to method names? It sure does!
Name mangling affects all names that start with two underscore
characters (“dunders”) in a class context:

``` python
class MangledMethod:
    def __method(self):
        return 42
    def call_it(self):
        return self.__method()

>>> MangledMethod().__method()
AttributeError:
"'MangledMethod' object has no attribute '__method'"
>>> MangledMethod().call_it()
42
```

Here’s another, perhaps surprising, example of name mangling in action:

``` python
_MangledGlobal__mangled = 23

class MangledGlobal:
    def test(self):
        return __mangled
>>> MangledGlobal().test()
23
```

In this example, I declared _MangledGlobal__mangled as a global
variable. Then I accessed the variable inside the context of a class
named MangledGlobal. Because of name mangling, I was able to
reference the _MangledGlobal__mangled global variable as just
__mangled inside the test() method on the class.
The Python interpreter automatically expanded the name __mangled
to _MangledGlobal__mangled because it begins with two underscore
characters. This demonstrates that name mangling isn’t tied to class
attributes specifically. It applies to any name starting with two underscore characters that is used in a class context.
Whew! That was a lot to absorb.
To be honest with you, I didn’t write down these examples and explanations off the top of my head. It took me some research and editing
to do it. I’ve been using Python for years but rules and special cases
like that aren’t constantly on my mind.
Sometimes the most important skills for a programmer are “pattern
recognition” and knowing where to look things up. If you feel a little
overwhelmed at this point, don’t worry. Take your time and play with
some of the examples in this chapter.
Let these concepts sink in enough so that you’ll recognize the general
idea of name mangling and some of the other behaviors I’ve shown
you. If you encounter them “in the wild” one day, you’ll know what to
look for in the documentation.

<h5 dir='rt1'>Sidebar: What are dunders?</h5>

If you’ve heard some experienced Pythonistas talk about Python or
watched a few conference talks you may have heard the term dunder.
If you’re wondering what that is, well, here’s your answer:
Double underscores are often referred to as “dunders” in the Python
community. The reason is that double underscores appear quite often
in Python code, and to avoid fatiguing their jaw muscles, Pythonistas
often shorten “double underscore” to “dunder.”
For example, you’d pronounce __baz as “dunder baz.” Likewise,
__init__ would be pronounced as “dunder init,” even though one
might think it should be “dunder init dunder.”
But that’s just yet another quirk in the naming convention. It’s like a
secret handshake for Python developers.

<h5 dir='rt1'>4. Double Leading and Trailing Underscore: “__var__”</h5>

Perhaps surprisingly, name mangling is not applied if a name starts
and ends with double underscores. Variables surrounded by a double
underscore prefix and postfix are left unscathed by the Python interpreter:

``` python
class PrefixPostfixTest:
    def __init__(self):
        self.__bam__ = 42
        
>>> PrefixPostfixTest().__bam__
42
```

However, names that have both leading and trailing double underscores are reserved for special use in the language. This rule covers
things like __init__ for object constructors, or __call__ to make objects callable.
These dunder methods are often referred to as magic methods—but
many people in the Python community, including myself, don’t like
that word. It implies that the use of dunder methods is discouraged,
which is entirely not the case. They’re a core feature in Python and
should be used as needed. There’s nothing “magical” or arcane about
them.
However, as far as naming conventions go, it’s best to stay away from
using names that start and end with double underscores in your own
programs to avoid collisions with future changes to the Python language.

<h5 dir='rt1'>5. Single Underscore: “_”</h5>

Per convention, a single stand-alone underscore is sometimes used as
a name to indicate that a variable is temporary or insignificant.
For example, in the following loop we don’t need access to the running
index and we can use “_” to indicate that it is just a temporary value:

``` python
>>> for_inrange(32):
...     print('Hello, World.')
```

You can also use single underscores in unpacking expressions as a
“don’t care” variable to ignore particular values. Again, this meaning
is per convention only and it doesn’t trigger any special behaviors in
the Python parser. The single underscore is simply a valid variable
name that’s sometimes used for this purpose.
In the following code example, I’m unpacking a tuple into separate
variables but I’m only interested in the values for the color and
mileage fields. However, in order for the unpacking expression to
succeed, I need to assign all values contained in the tuple to variables.
That’s where “_” is useful as a placeholder variable:

``` python
>>> car = ('red', 'auto', 12, 3812.4)
>>> color, _, _, mileage = car
>>> color
'red'
>>> mileage
3812.4
>>> _
12
```

Besides its use as a temporary variable, “_” is a special variable in most
Python REPLs that represents the result of the last expression evaluated by the interpreter.
This is handy if you’re working in an interpreter session and you’d like
to access the result of a previous calculation:

``` python
>>> 20 + 3
23
>>> _
23
>>> print(_)
23
```

It’s also handy if you’re constructing objects on the fly and want to
interact with them without assigning them a name first:

``` python
>>> list()
[]
>>> _.append(1)
>>> _.append(2)
>>> _.append(3)
>>> _
[1, 2, 3]
```

<h5 dir='rt1'>Key Takeaways</h5>

   • Single Leading Underscore “_var”: Naming convention indicating a name is meant for internal use. Generally not enforced by the Python interpreter (except in    wildcard imports)
   and meant as a hint to the programmer only.
   
   • Single Trailing Underscore “var_”: Used by convention to
   avoid naming conflicts with Python keywords.
   
   • Double Leading Underscore “__var”: Triggers name mangling when used in a class context. Enforced by the Python interpreter.
   
   • Double Leading and Trailing Underscore “__var__”: Indicates special methods defined by the Python language. Avoid
   this naming scheme for your own attributes.
   
   • Single Underscore “_”: Sometimes used as a name for temporary or insignificant variables (“don’t care”). Also, it represents the result of the last      expression in a Python REPL session.


<h3 dir='rt1'>2.5 A Shocking Truth About String</h3>

Remember the Zen of Python and how there should be “one obvious
way to do something?” You might scratch your head when you find
out that there are four major ways to do string formatting in Python.
In this chapter I’ll demonstrate how these four string formatting approaches work and what their respective strengths and weaknesses
are. I’ll also give you my simple “rule of thumb” for how I pick the
best general-purpose string formatting approach.
Let’s jump right in, as we’ve got a lot to cover. In order to have a simple
toy example for experimentation, let’s assume we’ve got the following
variables (or constants, really) to work with:

``` python
>>> errno = 50159747054
>>> name = 'Bob'
```

And based on these variables we’d like to generate an output string
with the following error message:

``` python
'Hey Bob, there is a 0xbadc0ffee error!'
```

Now, that error could really spoil a dev’s Monday morning! But we’re
here to discuss string formatting today. So let’s get to work.

<h5 dir='rt1'>#1 – “Old Style” String Formatting</h5>

Strings in Python have a unique built-in operation that can be
accessed with the %-operator. It’s a shortcut that lets you do simple
positional formatting very easily. If you’ve ever worked with a
printf-style function in C, you’ll instantly recognize how this works.
Here’s a simple example:

``` python
>>> 'Hello, %s' % name
'Hello, Bob'
```

I’m using the %s format specifier here to tell Python where to substitute the value of name, represented as a string. This is called “old style”
string formatting.
In old style string formatting there are also other format specifiers
available that let you control the output string. For example, it’s possible to convert numbers to hexadecimal notation or to add whitespace
padding to generate nicely formatted tables and reports.11
Here, I’m using the %x format specifier to convert an int value to a
string and to represent it as a hexadecimal number:

``` python
>>> '%x' % errno
'badc0ffee'
```

The “old style” string formatting syntax changes slightly if you want to
make multiple substitutions in a single string. Because the %-operator
only takes one argument, you need to wrap the right-hand side in a
tuple, like so:

``` python
>>> 'Hey %s, there is a 0x%x error!' % (name, errno)
'Hey Bob, there is a 0xbadc0ffee error!'
```

It’s also possible to refer to variable substitutions by name in your
format string, if you pass a mapping to the %-operator:

``` python
>>> 'Hey%(name)s, there is a 0x%(errno)xerror!'%{
...    "name": name,"errno": errno }
'Hey Bob, there is a 0xbadc0ffee error!'
```

This makes your format strings easier to maintain and easier to modify
in the future. You don’t have to worry about making sure the order
you’re passing in the values matches up with the order the values are
referenced in the format string. Of course, the downside is that this
technique requires a little more typing.
I’m sure you’ve been wondering why this printf-style formatting is
called “old style” string formatting. Well, let me tell you. It was technically superseded by “new style” formatting, which we’re going to
talk about in a minute. But while “old style” formatting has been deemphasized, it hasn’t been deprecated. It is still supported in the latest versions of Python.

<h5 dir='rt1'>#2 – “New Style” String Formatting</h5>

Python 3 introduced a new way to do string formatting that was also
later back-ported to Python 2.7. This “new style” string formatting
gets rid of the %-operator special syntax and makes the syntax for
string formatting more regular. Formatting is now handled by calling a format() function on a string object.12
You can use the format() function to do simple positional formatting,
just like you could with “old style” formatting:

``` python
>>> 'Hello, {}'.format(name)
'Hello, Bob'
```

Or, you can refer to your variable substitutions by name and use them
in any order you want. This is quite a powerful feature as it allows
for re-arranging the order of display without changing the arguments
passed to the format function:

``` python
>>> 'Hey {name}, there is a 0x{errno:x} error!'.format(
...    name=name, errno=errno)
'Hey Bob, there is a 0xbadc0ffee error!'
```

This also shows that the syntax to format an int variable as a hexadecimal string has changed. Now we need to pass a format spec by adding
a “:x” suffix after the variable name.
Overall, the format string syntax has become more powerful without
complicating the simpler use cases. It pays off to read up on this string
formatting mini-language in the Python documentation.13
In Python 3, this “new style” string formatting is preferred over %-style
formatting. However, starting with Python 3.6 there’s an even better
way to format your strings. I’ll tell you all about it in the next section.

<h5 dir='rt1'>#3 – Literal String Interpolation (Python 3.6+)</h5>

Python 3.6 adds yet another way to format strings, called Formatted
String Literals. This new way of formatting strings lets you use embedded Python expressions inside string constants. Here’s a simple
example to give you a feel for the feature:

``` python
>>> f'Hello, {name}!'
'Hello, Bob!'
```

This new formatting syntax is powerful. Because you can embed arbitrary Python expressions, you can even do inline arithmetic with it,
like this:
``` python
>>> a = 5
>>> b = 10
>>> f'Five plus ten is {a + b} and not {2 * (a + b)}.'

'Five plus ten is 15 and not 30.'
```

Behind the scenes, formatted string literals are a Python parser feature that converts f-strings into a series of string constants and expressions. They then get joined up to build the final string.

Imagine we had the following greet() function that contains an fstring:

``` python
>>> def greet(name, question):
...    return f"Hello, {name}! How's it {question}?"
...

>>> greet('Bob', 'going')
"Hello, Bob! How's it going?"
```

When we disassemble the function and inspect what’s going on behind the scenes, we can see that the f-string in the function gets transformed into something similar to the following:

``` python
>>> def greet(name, question):
...    return ("Hello, " + name + "! How's it " +
            question + "?")
```

The real implementation is slightly faster than that because it uses the
BUILD_STRING opcode as an optimization.14 But functionally they’re
the same:

``` python
>>> import dis
>>> dis.dis(greet)
  2    0 LOAD_CONST     1 ('Hello, ')
       2 LOAD_FAST      0 (name)
       4 FORMAT_VALUE   0
       6 LOAD_CONST     2 ("! How's it ")
       8 LOAD_FAST      1 (question)
       10 FORMAT_VALUE  0
       12 LOAD_CONST    3 ('?')
       14 BUILD_STRING  5
       16 RETURN_VALUE
```

String literals also support the existing format string syntax of the
str.format() method. That allows you to solve the same formatting
problems we’ve discussed in the previous two sections:

``` python
>>> f"Hey {name}, there's a {errno:#x} error!"
"Hey Bob, there's a 0xbadc0ffee error!"
```

Python’s new Formatted String Literals are similar to the JavaScript
Template Literals added in ES2015. I think they’re quite a nice addition to the language, and I’ve already started using them in my dayto-day Python 3 work. You can learn more about Formatted String
Literals in the official Python documentation.

<h5 dir='rt1'>#4 – Template Strings</h5>

One more technique for string formatting in Python is Template
Strings. It’s a simpler and less powerful mechanism, but in some
cases this might be exactly what you’re looking for.
Let’s take a look at a simple greeting example:

``` python
>>> from string import Template
>>> t = Template('Hey, $name!')
>>> t.substitute(name=name)
'Hey, Bob!'
```

You see here that we need to import the Template class from Python’s
built-in string module. Template strings are not a core language feature but they’re supplied by a module in the standard library.
Another difference is that template strings don’t allow format specifiers. So in order to get our error string example to work, we need to
transform our int error number into a hex-string ourselves:

``` python
>>> templ_string = 'Hey $name, there is a $error error!'
>>> Template(templ_string).substitute(
...    name=name, error=hex(errno))
'Hey Bob, there is a 0xbadc0ffee error!'
```

That worked great but you’re probably wondering when you use template strings in your Python programs. In my opinion, the best use
case for template strings is when you’re handling format strings generated by users of your program. Due to their reduced complexity,
template strings are a safer choice.
The more complex formatting mini-languages of other string formatting techniques might introduce security vulnerabilities to your programs. For example, it’s possible for format strings to access arbitrary
variables in your program.
That means, if a malicious user can supply a format string they can
also potentially leak secret keys and other sensible information!
Here’s a simple proof of concept of how this attack might be used:

``` python
>>> SECRET = 'this-is-a-secret'
>>> class Error:
...    def __init__(self):
           pass
>>> err = Error()
>>> user_input = '{error.__init__.__globals__[SECRET]}'

# Uh-oh...
>>> user_input.format(error=err)
'this-is-a-secret'
```

See how the hypothetical attacker was able to extract our secret string
by accessing the __globals__ dictionary from the format string?
Scary, huh! Template Strings close this attack vector, and this makes
them a safer choice if you’re handling format strings generated from
user input:

``` python
>>> user_input = '${error.__init__.__globals__[SECRET]}'
>>> Template(user_input).substitute(error=err)
ValueError:
"Invalid placeholder in string: line 1, col 1"
```

<h5 dir='rt1'>Which String Formatting Method Should I Use?</h5>

I totally get that having so much choice for how to format your strings
in Python can feel very confusing. This would be a good time to bust
out some flowchart infographic…
But I’m not going to do that. Instead, I’ll try to boil it down to the
simple rule of thumb that I apply when I’m writing Python.
Here we go—you can use this rule of thumb any time you’re having
difficulty deciding which string formatting method to use, depending
on the circumstances:

<h6 dir='rt1'>Dan’s Python String Formatting Rule of Thumb:</h6>

    If your format strings are user-supplied, use Template
    Strings to avoid security issues. Otherwise, use Literal
    String Interpolation if you’re on Python 3.6+, and “New
    Style” String Formatting if you’re not.

<h5 dir='rt1'>Key Takeaways</h5>

   • Perhaps surprisingly, there’s more than one way to handle
   string formatting in Python.
   
   • Each method has its individual pros and cons. Your use case
   will influence which method you should use.
   
   • If you’re having trouble deciding which string formatting
   method to use, try my String Formatting Rule of Thumb.
   
   
<h3 dir='rt1'>2.6 “The Zen of Python” Easter Egg</h3>

I know what follows is a common sight as far as Python books go. But
there’s really no way around Tim Peters’ Zen of Python. I’ve benefited
from revisiting it over the years, and I think Tim’s words made me a
better coder. Hopefully they can do the same for you.
Also, you can tell the Zen of Python is a big deal because it’s included as
an Easter egg in the language. Just enter a Python interpreter session
and run the following:

``` python
>>> import this
```

<h5 dir='rt1'>The Zen of Python, by Tim Peters</h5>

Beautiful is better than ugly.

Explicit is better than implicit.

Simple is better than complex.

Complex is better than complicated.

Flat is better than nested.

Sparse is better than dense.

Readability counts.

Special cases aren’t special enough to break the rules.

Although practicality beats purity.

Errors should never pass silently.

Unless explicitly silenced.

In the face of ambiguity, refuse the temptation to guess.

There should be one—and preferably only one—obvious way to do it.

Although that way may not be obvious at first unless you’re Dutch.

Now is better than never.

Although never is often better than right now.

If the implementation is hard to explain, it’s a bad idea.

If the implementation is easy to explain, it may be a good idea.

Namespaces are one honking great idea—let’s do more of those!

