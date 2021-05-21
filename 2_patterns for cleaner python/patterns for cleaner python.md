
<h1 dir="rtl">Patterns for Cleaner Python</h1>
<div dir='rt1'>
  
<h3 dir='rt1'>2.1 Covering Your A** With Assertions</h3>
  
Sometimes a genuinely helpful language feature gets less attention than it deserves. For some reason, this is what happened to Python’s built-inassert statement.In this chapter I’m going to give you an introduction to using asser-tions in Python. You’ll learn how to use them to help automaticallydetect errors in your Python programs. This will make your programsmore reliable and easier to debug.At this point, you might be wondering “What are assertions and whatare they good for?” Let’s get you some answers for that.At its core, Python’s assert statement is a debugging aid that tests acondition. If the assert condition is true, nothing happens, and yourprogram continues to execute as normal. But if the condition evalu-ates to false, an Assertion Error exception is raised with an optional error message.

<h4 dir='rt1'>Assert in Python — An Example</h4>
  
Here’s a simple example so you can see where assertions might comein handy. I tried to give this some semblance of a real-world problem you might actually encounter in one of your programs.Suppose you were building an online store with Python. You’re work-ing to add a discount coupon functionality to the system, and eventu-ally you write the following apply_discount function:
  
``` python
defapply_discount(product, discount):
    price=int(product['price']*(1.0-discount))
    assert0<=price<=product['price']
    return price
```
