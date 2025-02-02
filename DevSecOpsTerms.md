# DevSecOps 

## understanding Dev
### Coding Terms
## Table of Contents
  - [Arguments in python].(#Arguments) in python)
In Python, arguments are the values passed to a function when it is called. They are the actual data that the function operates on. 
Here's a breakdown of the different types of arguments:
  -(#Positional Arguments):
These are the most common type of arguments. They are passed to the function in the order that the parameters are defined in the function definition.
Keyword Arguments:
These arguments are passed with their corresponding parameter names. This allows you to pass arguments out of order or skip some arguments.
Default Arguments:
These arguments have a default value assigned in the function definition. If you don't provide a value for these arguments when calling the function, the default value is used.
Variable-length Arguments:
These arguments allow you to pass a variable number of arguments to a function. They are defined using *args (for non-keyword arguments) and **kwargs (for keyword arguments).

def greet(name, age=18):
  print("Hello,", name)
  print("You are", age, "years old.")

greet("Alice")  # Using positional argument
greet("Bob", 20)  # Using positional arguments
greet(name="Charlie", age=25)  # Using keyword arguments

