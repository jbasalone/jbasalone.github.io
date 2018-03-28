# What Are Python Decorators?

That little @ symbol with a function name after it

    @somefunctionname
    def someotherfunctionname(parameter):
       return 
# What Are They Not?
![](https://d2mxuefqeaa7sj.cloudfront.net/s_C508EA5D229A366574E86D8CAA2DECC2AEAF4CEC911614815FC011CFD59FD7BB_1522202444611_image.png)



# The Purest Definition

**“Decorators** provide a simple syntax for calling higher-order functions. By definition, a **decorator** is a function that takes another function and extends the behavior of the latter function without explicitly modifying it. ..”
-- www.realpython.com 


# A Simpler Definition

Decorators extend and modify the behavior of (callables) functions, methods, and classes **WITHOUT** modifying the functions, methods, or classes themselves. 


# When would I use a Decorator?
- adding logging ability 
- enforcing access controls and authentication 
- Where you need timing, rate-limiting, and acquisition/release of locks 
- entering database transactions; commit if successful or rollback if failed


# When I Shouldn’t Use a Decorator?
- Decorators should not alter the semantics of its argument function 
- Decorators should not require a specific position when stacking
- If you need to call the function without its decorated form (reusable library style code) theres no simple easy way to do that. When you apply a decorator to a function, the function itself is lost from the namespace


# This Is Painful, Is Painful Logic Worth It?

Yes, and its big money 🤑 . This can be really easy for some and for many, really hard to wrap your head around. I’ve read some really painful instructions and introductions to decorators. As someone who is allergic to complexity, Im going to try to make this as simple as possible. 

I do need to make one assumption here, or rather come to an agreement with you. You know what first-class objects are.  That functions are objects, functions can be defined inside other functions, and functions can capture the parents local state.  Functions can be passed around, used as arguments same as other values like ‘int’, ‘float’, and ‘string’


## The Key Points You Should Get From Decorators
- Decorators are reusable chunks of code you can apply to a function, method, or class to modify the behavior without changing the function, method, or class itself. 
- The ‘@’ symble is just python short hand for calling the decorator on an input function and they are stackable 
- Debug yoru decorators with [functools.wraps](https://docs.python.org/3/library/functools.html#functools.wraps) helper


# Reminder on Functions

You can assign variables to functions 

    def muppet_name(name):
        return 'Your Muppet Name Is: ' + name 
    
    get_your_name = muppet_name 
    print get_your_name('cookie monster') 
    
    # Your Muppet Name Is: cookie monster 

Define Functions within Functions

    def muppet_name(name):
       def the_message():
           return 'Hello '
        
       result = the_message() + name  
       return result 
    
    print muppet_name('cookie monster') # <-- Try to call the_message() What happens?
    # Hello cookie monster 

Functions can be passed as parameters to other functions 

    def muppet_name(name):
        return 'Hello ' + name 
    
    def name_call(func):
       other_name = 'gonzo' 
       return func(other_name)
    
    print name_call(muppet_name)
    # Hello gonzo 

Functions can generate other functions

    def compose_a_greeting():
       def the_message():
          return 'Hidy Ho ' 
       return the_message():
    
    greet = compose_a_greeting('cookie monster')
    print greet() 
    
    # Hidy Ho cookie monster 

Inside functions can access the enclosing scope or ‘closure’

    def compose_a_greeting():
       def the_message():
          return 'Hello there ' + name + '!'
        
        return the_message 
    
    greet = compose_a_greeting('cookie monster') 
    print greet() 
# Decorating with Functions Looks Like
    def xmltag(fn):
       def func(*args):
            return '<' +fn(*args) + '/>'
       return func
    
    @xmltag 
    def render_tag(name):
       return name 
       
    render_tag('llama') 
    
    # <llama/>


# Lets Create a Function Decorator! 

a function decorator is a wrapper to existing functions. Lets build a function that wraps the string output of another function and bolds it with ‘b’ tag. 

    def getting_the_text(name):
       return '{0}, is the best!'.format(name)
    
    def b_decorate(func):
        def the_wrapper(name):
           return '<b>{0}</b>'.format(func(name))
        return the_wrapper
    
    bold_that_text = b_decorate(getting_the_text)
    
    print bold_that_text('cookie monster')
    # <b>cookie monster<b>, is the best! 


# The Python Decorator Way! 
    def b_decorate(func):
       def the_wrapper(name):
           return '<b>{0}</b>'.format(func(name))
        return the_wrapper
    
    @b_decorate 
    def getting_the_text(name):
       return '{0}, is the best!'.format(name)
       
    print getting_the_text('cookie monster') 
    
    # <b>cookie monster</b>, is the best! 

.. hint.. decorators are stackable. So make one for the html tag, or the p tag and stack it up like

    @b_decorate
    @p_decorate
    @html_decorate  
    def getting_the_text(name):
       return '{0}, is the best!'.format(name)

However,  those three are common.. could we just make a ‘@html_tags’ decorator that takes the tag to wrap the text with? 🤔 

# Decorators for Methods 


## What is a method?

methods are functions that expect their first parameter to be referenced within the current object. So we can build a decorator for methods in the same way, taking self into consideration in the wrapper function


# Write our Method Decorator
    def b_decorate(func):
       def the_wrapper(self):
          return '<b>{0}</b>'.format(func(name))
       return the_wrapper
    
    class Muppet(ojbect):
       def __init__(self):
          self.name = 'Sid'
          self.nickname = 'cookie monster'
       
       @b_decorate
       def get_allnames(self):
          return self.name + ', ' + self.nickname
    
    my_muppet = Muppet()
    print my_muppet.get_allnames()
    # Sid, cookie monster


# An Improvement Opens The Decorator
    def b_decorate(func):
       def the_wrapper(*args, **kwargs): # <--- CHANGE
          return '<b>{0}</b>'.format(func(name))
       return the_wrapper
    
    class Muppet(ojbect):
       def __init__(self):
          self.name = 'Sid'
          self.nickname = 'cookie monster'
       
       @b_decorate
       def get_allnames(self):
          return self.name + ', ' + self.nickname
    
    my_muppet = Muppet()
    print my_muppet.get_allnames()
    # Sid, cookie monster

adding args and kwargs allows us to add any arbitrary number of arguments and keyword arguments and forwards it to the function call. 


# Class Based Decorators
    class ClassBasedDecorator(object):
    
       def __init__(self, func_to_decorate):
          print 'INIT ClassBasedDecorator'
          self.func_to_decorate = func_to_decorate
          
       def __call__(self, *args, **kwargs):
        '''The only constraint upon the object returned by the decorator is 
        that it can be used as a function; meaning, it must be callable. 
        Any classes we use as decorators must implement __call__. '''
          print 'CALL ClassBasedDecorator'
          return self.func_to_decorate(*args, **kwargs)
    
    @ClassBasedDecorator
    def print_all_the_things(*args):
       for arg in args:
          print arg 
    
    print_all_the_things(1,3,5,7,9)
    
    '''
    Side Note: In Python 3 
    __call__ is fundementally changed, as only a single function can be passed to __call__
    '''

This is for people who want to maintain state or just like to make it a little more complicated. These operate at a higher level and decorate the entire class. Their effect takes place at the class definition time. You can use the decorators to add or remove methods of any decorated class, or apply function decorators to an entire set of methods. 

Examples like this is keeping track of various states of a program or various exceptions that were raised for a program. 

# Examples
# A Working Example of a Timer 
    from time import sleep
    
    def sleep_decorator(function):
        '''We are limiting how fast the function is called'''
        def wrapper(*args, **kwargs):
           sleep(2)
           return function(*args, **kwargs)
        return wrapper 
    
    def print_number(num):
        return num
    
    print print_number(222)
    
    for num in range(1, 6):
       print print_number(num)
       
    # thanks to realpython.com for the example


# An Example of Logging
    import functools 
    
    def log_this(logger, level='info'):
       def log_decorator(fn):
          @functools.wraps(fn)
          def wrapper(*args, **kwargs):
             getattr(logger, level)(fn.__name__)
             return fn(*args, **kwargs)
          return wrapper
       return log_decorator 
    
    @log_this(logging.getLogger('main'), level='warning')
    def some_really_dangerous_weird_thing_im_sure_i_did(times):
       for _ in xrange(times): bad_code.get_code(crashes=True).execute() 
    
    


# An Example of When To Use a Decorator
    def OG():
       print 'Im an OG'
    OG()
    
    OG_variable = OG 
    
    def decorated_OG():
       print 'Your Decorator Functions'
       OG_variable 
    
    OG = decorated_OG
    OG()

Thats a lot of work to  do. Its completely polluted the name space with stuff like OG_variable and decorator_OG, and requires repetition if you want to use it again. 


## The Decorated Way
    def do_i_work(func): 
       def decorated(*args, **kwargs):
          print 'Your Decorator Functions'
          return func(*args, **kwargs)
       return decorated
    
    @do_i_work
    def function_1():
       print 'I am function 1'
    
    @do_i_work
    def function_2():
       print 'I am function 2'
       
    '''
    >> function_1()
    Your Decorator Functions
    I am function 1
    
    >> function_2()
    Your Decorator Functions
    I am function 2
    '''

Here we can see function_1() and function_2() prints ‘Your Decorator Functions’ and the invoked functions



# Wrap Up


## Few Notes
- decorators only affect the functions they are attached to
- When you apply a decorator to a function, the function itself is lost from the namespace but its lost attributes can be saved by using functools.wraps! :mind blown: 
  - These Function attributes are lost when decorated 
    - __name__ (name of the function),
    - __doc__ (the docstring) and
    - __module__ (The module in which the function is defined)
- The object returned by the decorator must be callable


Python affords us much flexibility and Decorators stretch the lengths of that flexibility. You can package reusable decorators and apply them to functions, methods, and a whole class. Discovering decorators and the ability to chain them has been really fun and I hope you enjoy learning them too! 

by: Jennifer Basalone @pennyblackio
