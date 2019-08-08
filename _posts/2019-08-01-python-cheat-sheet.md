---
layout: post
title: Python Cheat Sheet
categories: resources
---

A living and ever changing library of Python notes and code snippets.

<!-- more -->

## Project goal

As I am currently not a full time developer, often I find myself searching for the right syntax to use or trying to remember the unique characteristics of certain data structures. 

So, I have created this notebook as my own personal cheat sheet which I can refer back to at any point. I know there are many other cheat sheets like this which already exist, however this is personalised to me and the knowledge areas which I personally struggle with. Further, by creating this myself, hopefully the concepts begin to sink in more. 


```python
# Import dependencies
import datetime
```

## Data Structures

### Dictionaries


```python
# Creating dictionaries
dict1 = {"key1": 1, "key2": "value2"}
dict2 = dict()
```


```python
# Adding values to a dictionary
dict2['key3'] = "value3"
dict2
```




    {'key3': 'value3'}




```python
# Deleting values from a dictionary
del dict1["key1"]
dict1
```




    {'key2': 'value2'}




```python
# Looping dictionaries
dict3 = {"key1": 1, "key2": 2, "key3": 3}

for key, value in dict3.items():
    print("This is the key:", key)
    print("This is the value:", value)
```

    This is the key: key1
    This is the value: 1
    This is the key: key2
    This is the value: 2
    This is the key: key3
    This is the value: 3



```python
# Unpacking key and values from **kwargs in a Class
class DictTest:
    def __init__(self, **kwargs):
        for key, value in kwargs.items():
            setattr(self, key, value)
            
dict4 = DictTest(name = "peter", age = 32, sex = "male")
dict4.name
```




    'peter'



## Classes


```python
class Person:
    # Use a leading underscore to indicate that this attribute should not be modified
    _dob = None 
    
    # Leading dunder hides the attribute from others accessing the class
    __security = None
    
    retirement_age = 65
 
    # Initialise the class
    def __init__(self, name, age, sex, **kwargs):
        self.name = name
        self.age = age
        self.sex = sex
        
        # Loop through any kwargs and set attributes
        for key, value in kwargs.items():
            setattr(self, key, value)
```


```python
class Student(Person):    
    # Set init and inherit from Person class
    def __init__(self, name, age, sex, student_id, **kwargs):
        
        # Call super BEFORE setting other atttributes
        super().__init__(name, age, sex, **kwargs) 
        self.student_id = student_id
        
    # Override inbuilt Python add operator for the class
    def __add__(self, other):
        return self.age + other
    
    # Method to calculate Students year of birth
    def yob(self):
        return datetime.datetime.now().year - self.age
    
    # Instead of showing as a method, show the results as a property
    @property
    def years_til_retirement(self):
        return self.retirement_age - self.age
    
    # Create a setting in case we want to set the value of the property as an attribute
    # If we try to set the value of the "years_til_retirement" attribute, we will get an error as the attr does not yet exist
    @years_til_retirement.setter
    def years_til_retirement(self, years_til_retirement):
        self.retirement_age = self.retirement_age - years_til_retirement
```


```python
# Create an instance of the Person class and access attributes
john = Person("John", 24, "male")
john.age
```




    24




```python
# Create an instance of the Student class and access attributes
sarah = Student("Sarah", 22, "Female", 16392, history_class = False)
sarah.name
```




    'Sarah'




```python
# Use overridden add operator
another_year = 1
print(sarah + another_year)

# Could have easily done the following, but classes should be self-contained and the user should not need to know all of the attribues to be able to use
print(sarah.age + another_year)
```

    23
    23



```python
# Accessed methods
sarah.yob()
```




    1997




```python
# Access properties of a class (methods, but accessed like attributes)
sarah.years_til_retirement
```




    43




```python
# Update the years_til_retirement attr using the setter 
sarah.years_til_retirement = 20
print("{} will retire in {} years when she is {} years old".format(sarah.name, sarah.years_til_retirement, sarah.retirement_age))
```

    Sarah will retire in 23 years when she is 45 years old


**Notes**  
- Name classes with CamelCase
- To inherit from another class, pass the parent in as an argument of the child class
- use \__init__ to initialise the class and set attribute values
- Variables in the class are called "attributes"
- Functions in the class are callsed "methods"
- User super().parent_func_name(args, kwargs) to initialise the child class with the parent class \__init__
- In general, the ultimate parent class, should not call on super()
- Use a leading underscore to indicate that an attribute should not be modified (e.g \_do_not_use). This will leave it visible to others though
- Use a leading double underscore to hide an attribute from others
- If you don't want to access a method of a class as a method, use the @property decorator to make the results of the method accesssible like an attribute

#### Classes which override immutable objects


```python
class ReversedString(str):
    def __new__(*args, **kwargs): # new is a class method. Therefore, does NOT take self
        self = str.__new__(*args, **kwargs)
        self = self[::-1]
        return self # Unlike __init__, we return a value with __new__
```


```python
rev_string = ReversedString("dogs")
rev_string
```




    'sgod'



#### Classes which override mutable objects


```python
import copy
class FilledList(list):
    def __init__(self, count, value, *args, **kwargs):
        super().__init__()
        for _ in range(count): # Underscore ignores the number coming out of range
            self.append(copy.copy(value))
```


```python
fl = FilledList(2,[1,2,3])
```


```python
fl
```




    [[1, 2, 3], [1, 2, 3]]



**Notes**  
- If we want to change immutable objects (e.g. strings), then we should use \__new__ to change the object at run time. 
- If we want to change mutable objects (e.g. lists), then we change them with \__init__

### Python Conventions

**General**  
- 4 space indentation
- CamelCase for class naming
- lower_case_underscore for function and variable names
- CAPS for constants
- imports at the top and all on separate lines
- Use "flake8" from the terminal or add to sublime to check scripts for adherance to pep8


```python
# For top line functions (i.e. not in a class etc), put 2 lines between them
def my_func1():
    return 1


def my_func2(arg1, arg2, arg3): # Spaces between commas
    return 2
```


```python
# Classes are named with CamelCase
class MyClass:
    # Functions inside a class only have one space between them
    def class_func1():
        return 1
    
    def class_func2():
        return 2
    
    
class MyClass2:    # 2 spaces between classes
    pass
```


```python
# Variable assignment has a space either side of the equals sign
var1 = "my variable"
```


```python
# Keyword args do NOT have spaces between the equals signs
# Space after each comma
def my_func("key1"=None, "key2"=1)
```


```python
# Dictionaries have a space after the colon, but not before
my_dict = {"key1": "value after a space", "key2": "another example"}
```


```python
# Constants are in caps
CONSTANT_VAR = 10
```


```python
# Importing modules must be on separate lines
# Python imports vs external imports are separated with a line
from collections import OrderedDict
from datetime import datetime

import pandas as pd
```

#### Docstrings
Source: https://github.com/google/styleguide/blob/gh-pages/pyguide.md

Always use the three double-quote """ format for docstrings (per PEP 257).

A function must have a docstring, unless it meets all of the following criteria:

not externally visible
very short
obvious
A docstring should give enough information to write a call to the function without reading the function's code. The docstring should be descriptive ("""Fetches rows from a Bigtable.""") rather than imperative ("""Fetch rows from a Bigtable."""). A docstring should describe the function's calling syntax and its semantics, not its implementation. For tricky code, comments alongside the code are more appropriate than using docstrings.

A method that overrides a method from a base class may have a simple docstring sending the reader to its overridden method's docstring, such as """See base class.""". The rationale is that there is no need to repeat in many places documentation that is already present in the base method's docstring. However, if the overriding method's behavior is substantially different from the overridden method, or details need to be provided (e.g., documenting additional side effects), a docstring with at least those differences is required on the overriding method.

Certain aspects of a function should be documented in special sections, listed below. Each section begins with a heading line, which ends with a colon. Sections should be indented two spaces, except for the heading.

Args: : List each parameter by name. A description should follow the name, and be separated by a colon and a space. If the description is too long to fit on a single 80-character line, use a hanging indent of 2 or 4 spaces (be consistent with the rest of the file).
The description should include required type(s) if the code does not contain a corresponding type annotation.
If a function accepts *foo (variable length argument lists) and/or **bar (arbitrary keyword arguments), they should be listed as *foo and **bar.

Returns: (or Yields: for generators) : Describe the type and semantics of the return value. If the function only returns None, this section is not required. It may also be omitted if the docstring starts with Returns or Yields (e.g. """Returns row from Bigtable as a tuple of strings.""") and the opening sentence is sufficient to describe return value.

Raises: : List all exceptions that are relevant to the interface.


```python
def fetch_bigtable_rows(big_table, keys, other_silly_variable=None):
    """Fetches rows from a Bigtable.

    Retrieves rows pertaining to the given keys from the Table instance
    represented by big_table.  Silly things may happen if
    other_silly_variable is not None.

    Args:
        big_table: An open Bigtable Table instance.
        keys: A sequence of strings representing the key of each table row
            to fetch.
        other_silly_variable: Another optional variable, that has a much
            longer name than the other args, and which does nothing.

    Returns:
        A dict mapping keys to the corresponding table row data
        fetched. Each row is represented as a tuple of strings. For
        example:

        {'Serak': ('Rigel VII', 'Preparer'),
         'Zim': ('Irk', 'Invader'),
         'Lrrr': ('Omicron Persei 8', 'Emperor')}

        If a key from the keys argument is missing from the dictionary,
        then that row was not found in the table.

    Raises:
        IOError: An error occurred accessing the bigtable.Table object.
    """
```

### Logging


```python
import logging
```


```python
# logs aren't printed to the end user
print(logging.info("This isn't seen by the user"))
```

    None



```python
# Creates a log file to save to
logging.basicConfig(filename="test.log", level=logging.DEBUG)
```

Levels of logging are:
* Critical 
* Error
* Warning
* Info
* Debug 
* Notset


```python
# Log the initial state of an object as an INFO item
test = "This is what the string was in the beginning"
logging.info('Start string: {}'.format(test))

# Change the object and then log the new state as a DEBUG
test = test.upper()
logging.debug('Updated string: {}'.format(test))
```

Output in the log file is:  
  
INFO:root:Start string: This is what the string was in the beginning
INFO:root:Updated string: THIS IS WHAT THE STRING WAS IN THE BEGINNING

Note: The above does not work in Jupyter notebooks.

### Python Debugger (PDB)


```python
# Terribly created list - we only want the numbers
bad_list = [0, 2, "cat", 4, False, 6]
```


```python
# Set trace for where we want to start debugging from
import pdb; pdb.set_trace()

# Delete non-numeric items (deliberately wrong)
del bad_list[3]
del bad_list[5]
```

When trace set for pdb, you are provided with a pdb prompt in the terminal. Enter "next" or just "n" to move from line to line in the code to identify any errors. 

**PDB Commands**
- "n" or "next" to move to the next line of code
- "q" to quit PDB
- "c" to continue through the rest of the script without PDB running line-by-line (i.e. run as if PDB weren't there)
- "var_name" to display the state of a variable at any stage in the PDB process


## Test Driven Development (TDD)

### Types of Tests

**Unit Tests**  
Units are single features or aspects of your code. 

For example, if you have a class called Car. Car will have attributes (e.g. top_speed, max_milage, make, model etc), as well as methods (e.g. start, stop etc). 

A unit test focuses on a single aspect (e.g. top_speed which must be numeric) and deliberately sets the attribute to a false value (e.g. str of "fast") and then raises an exception. 

**Regression Tests**  
Regression tests ensure that previously made mistakes do not happen again. 

**Integration Tests**  
Makes sure that for a given process that each unit works from beginning to end. 

### Doctests
Somewhat unique to Python and are written in a docstring. 

Documentation: https://docs.python.org/3/library/doctest.html

To run from the terminal:  
python -m doctest your_script.py


```python
import doctest
```


```python
# Dummy function to test
# Two tests. Test one will pass, test two will deliberately fail
def multiply_nums(number, multiplier):
    """ Insert the docstring like normal - Returns multiplication of two numbers. 
    
    >>> multiply_nums(2, 2)
    4
    >>> multiply_nums(3, 2)
    999

    """
    result = number * multiplier
    return result
```

Notes:  
- Important to leave a space before the end of the docstring block, otherwise Python sometimes misses the doctest
- Best to run the tests from the terminal using the python -m doctest your_script.py command
- Doctests use string comparison, so they can cause problems with floats, as well as not being very reusable as they are tightly bound to the function they are written for

### Unittests

To run from the terminal:  
python -m unittest test_script_name.py

Save all of the tests in a separate test script. Not to be included in the main code.

Documentation: https://docs.python.org/3/library/unittest.html

Notes: 
- All tests must start with the keyword of "test_"
- There are many types of assertions which can be used (assert, assertEqual(x, y), assertGreater(x, y) etc)
- unittest module preceeds PEP8, so many functions are in camel case

**Assertions**  
- assertTrue(x)
- assertFalse(x)
- assertEqual(a, b)
- assertNotEqual(a, b)
- assertis(a, b)
- assertIsNot(a, b)
- assertIsNone(x)
- assertIsNotNone(a, b)
- assertIn(a, b)
- assertNotIn(a, b)
- assertIsInstance(a, b)
- assertNotIsInstance(a, b)
- assertAlmostEqual(a, b)  # checks if 2x rounded values = 0
- assertNotAlmostEqual(a, b)
- assertGreater(a, b)
- assertGreaterEqual(a, b)
- assertLess(a, b)
- assertLessEqual(a, b)
- assertRegex(string, regex)  # r.search(s)
- assertNotRegex(string, regex)
- assertCountEqual(a, b)
- assertMultiLineEqual(a, b)  # compares that two strings are equal
- assertListEqual(a, b)  # compares two lists
- assertTupleEqual(a, b)
- assertSetEqual(a, b)
- assertDictEqual(a, b)


```python
# Add the following to a script
import unittest

# Create a class which extends TestCase
class NumericTests(unittest.TestCase):
    # Create variables to re-use throughout the test cases
    def setUp(self):
        self.var1 = "xyz"
    
    def test_five_plus_1(self):
        assert 5 + 1 == 6
    
    def test_five_equals_five(self):
        assertEqual(5, 5)
    
    def test_five_not_equal_six(self):
        assertNotEqual(5, 6)
        
    def tearDown(self):
        self.var1.dispose()

    # Tests if an assertion is correctly raised when an invalid value is passed
    # The test fails if the assertion is NOT raised
    def test_exception(self):
        with self.assertRaises(ValueError):
            int('a')

# Runs the unit tests
if __name__ == "__main__":
    unittest.main()


#### Coverage
To determine how much of your code is covered by your unit tests, we can use the coverage library.

- pip install coverage

To get coverage to run over your test script, simply run the following from commands terminal:

- coverage run test_script_name.py  
- coverage report

Once we have the coverage report, to identify which code lines are NOT covered by the tests, run the following command:

- coverage report -m

To generate an interactive HTML version of the coverage report which highlights the uncovered lines of code, run the following commands instead: 

- coverage html
- python -m http.server  #treehouse only??
