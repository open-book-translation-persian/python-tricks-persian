
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


