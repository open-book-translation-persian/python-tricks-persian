
<h1 dir="rtl">Patterns for Cleaner Python</h1>
<div dir='rt1'>
  
<h3 dir='rt1'>1.2 Covering Your A** With Assertions</h3>
  
Sometimes a genuinely helpful language feature gets less attentionthan it deserves. For some reason, this is what happened to Python’sbuilt-inassertstatement.In this chapter I’m going to give you an introduction to using asser-tions in Python. You’ll learn how to use them to help automaticallydetect errors in your Python programs. This will make your programsmore reliable and easier to debug.At this point, you might be wondering “What are assertions and whatare they good for?” Let’s get you some answers for that.At its core, Python’s assert statement is a debugging aid that tests acondition. If the assert condition is true, nothing happens, and yourprogram continues to execute as normal. But if the condition evalu-ates to false, anAssertionErrorexception is raised with an optionalerror message.
