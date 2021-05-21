
<h1 dir="rtl">Patterns for Cleaner Python</h1>
<div dir='rt1'>
  
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
