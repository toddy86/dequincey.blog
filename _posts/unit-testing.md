---
layout: post
title: Testing with PyTest
categories: resources
---

A summary of key concepts, principles and techniques for unit testing with Python and PyTest.

<!-- more -->

## Why unit test
- Testing ensures your code works as expected given a set of conditions. Helps catch syntax errors.
- Helps ensure that changes to your code (e.g. refactoring) do not break the expected functionality / end result
- Writing good testing requires modular, decoupled code. So it forces you to write good (well better) code.

## Definitions (Source: Python Testing with pytest)
- Unit test: A test that checks a small bit of code, like a function or a class, in isolation of the rest of the system. I consider the tests in Chapter 1, Getting Started with pytest, on page 1, to be unit tests run against the Tasks data structure.
- Integration test: A test that checks a larger bit of the code, maybe several classes, or a subsystem. Mostly it’s a label used for some test larger than a unit test, but smaller than a system test.
- System test (end-to-end): A test that checks all of the system under test in an environment as close to the end-user environment as possible. 
- Functional test: A test that checks a single bit of functionality of a system. A test that checks how well we add or delete or update a task item in Tasks is a functional test.
- Subcutaneous test: A test that doesn’t run against the final end-user interface, but against an interface just below the surface.
- Test discovery: The part of pytest execution where pytest goes off and finds which tests to run. Files must either start or end with test_ or _test to be found by pytest test discovery.
- Smoke test: Not all-inclusive or thorough test suite. Rather, a specifically chosen subset that can be run quickly to provide an idea of whether "at a glance" the system is still running as expected.

## Structuring Tests (Source: Python Testing with pytest)
tests
├── conftest.py 
├── pytest.ini 
├── func
│ ├── __init__.py
│ ├── test_add.py
│ └── ...
└── unit
├── __init__.py
├── test_task.py 
└── ...


## Key Files
**pytest.ini**
- Optional file
- Contains project-wide pytest configuration
- Only one file should exist

**__init__.py**
- Required in the tests folder to tell the system to go up one level for the project and/or pytest.ini file, depending on setup

**conftest.py**
- Optional file
- Considered by pytest as a local plugin
- Contains hook functions and fixtures
- Hook functions are a way to insert code into part of the pytest execution process to alter how pytest work
- Fixtures are setup and tear downs before tests
- Can have multiple conftest files (e.g. one per test directory (func and unit tests))

## Testing Exceptions
{% highlight python %}
def test_exceptions_example():
    """Make sure exceptions are raised and tested.""" 
    with pytest.raises(ValueError) as e: # If ValueError not raised, or another exception raised, the test fails
        dummy_imported.module('arg_which_causes_the_exception') 
    exception_msg = e.value.args[0]
    assert exception_msg == "Error message raised: Do they match?"
{% endhighlight %}

## Marking Tests
- Can mark (i.e. group) tests in pytest
- Tests can have 1 or more markers
- Can then use these markers to run all tests with the mark

{% highlight python %}
@pytest.mark.smoke
def test_dummy_1():
    """Dummy test ."""
    ...

@pytest.mark.api
@pytest.mark.smoke
def test_dummy_2():
    """Dummy test ."""
    ...

@pytest.mark.api
def test_dummy_3():
    """Dummy test ."""
    ...

# To run all of the smoke tests
pytest -m 'smoke'

# To run all of the smoke and api tests
pytest -m 'smoke and api'
{% endhighlight %}

## Test Pattern
- Use a GIVEN, WHEN, THEN structure for each test
- GIVEN: a specific setup condition (e.g. database is initialised)
- WHEN: a specific event then occurs
- THEN: a specific outcome is expected (i.e. assert)

## Fixtures
- Setup and teardown that can be used across multiple tests
- autouse: all tests in this file will use the fixture

{% highlight python %}
# Source: Python Testing with pytest, page 33
@pytest.fixture(autouse=True)
def initialised__db(tmpdir):
"""Connect to db before testing, disconnect after."""
    # Setup : start db
    tasks.start_tasks_db(str(tmpdir), 'tiny') 
    
    yield # this is where the testing happens
    
    # Teardown : stop db
    tasks.stop_tasks_db()
{% endhighlight %}


## Parameterising Tests
- Parameterising tests allows us to send multiple values to a test and test for all of the outcomes. Allows the tests to be more robust and encompass more scenarios.
- To parameterise a test, you pass in two objects using @pytest.market.parameterize
- First arg is a string with a comma-separated list of the variables
- Second arg is a list of the values of the variables
- Both classes and functions can be parameterized. If a class is parameterized, all of the tests in that class will use the parameters

{% highlight python %}
# Source: Python Testing with pytest, page 44
@pytest.mark.parametrize('summary, owner, done', 
                        [('sleep', None, False),
                        ('wake', 'brian', False), 
                        ('breathe', 'BRIAN', True), 
                        ('eat eggs', 'BrIaN', False)
                        ])
def test_add_3(summary, owner, done):
    """Demonstrate parametrize with multiple parameters.""" 
    task = Task(summary, owner, done)
    task_id = tasks.add(task)
    t_from_db = tasks.get(task_id)
    assert equivalent(t_from_db, task)

# Can also setup a variable and pass that into the wrapper
# If doing this, you should also set task_ids to ensure the output is readable (i.e. you can see the input vars in the pytest output)
tasks_to_try = (Task('sleep', done=True), 
                Task('wake', 'brian'), 
                Task('wake', 'brian'),
                Task('breathe', 'BRIAN', True), 
                Task('exercise', 'BrIaN', False))

task_ids = ['Task({},{},{})'.format(t.summary, t.owner, t.done) for t in tasks_to_try] # For readability of output
@pytest.mark.parametrize('task', tasks_to_try, ids=task_ids) 
def test_add_5(task):
    """Demonstrate ids."""
    task_id = tasks.add(task) 
    t_from_db = tasks.get(task_id) 
    assert equivalent(t_from_db, task)

# Example Output
# Test input displayed since we setup the task_id. Otherwise would be "task0" 
test_add_variety.py::test_add_5[Task(sleep,None,True)] PASSED  
test_add_variety.py::test_add_5[Task(wake,brian,False)0] PASSED
...

{% endhighlight %}

## X
- X

## X
- X

## Misc Syntax and Tidbits
**Named Tuples**
{% highlight python %}
# Create a simple "fixture" style object to test against by using named tuples (Source: Python Testing with pytest, page xiii)
from collections import namedtuple
Task = namedtuple('Task', ['summary', 'owner', 'done', 'id']) 
Task.__new__.__defaults__ = (None, None, False, None)

def test_defaults():
    """Using no parameters should invoke defaults.""" 
    t1 = Task()
    t2 = Task(None, None, False, None)
    assert t1 == t2

# And use of the asdict conversion of a NamedTuple
def test_asdict():
    """_asdict() should return a dictionary.""" 
    t_task = Task('do something', 'okken', True, 21)
    t_dict = t_task._asdict()
    expected = {'summary': 'do something', 'owner': 'okken', 'done': True, 'id': 21}
    assert t_dict == expected
{% endhighlight %}

**CLI**
{% highlight python %}
# Run a single test in verbose mode
pytest -v tasks/test_four.py::test_asdict

# Collect, but don't run the tests. Useful for seeing which tests will be executed under a specific set of options
pytest --collect-only

# Search for the name of tests (functions) which match a specific pattern
# Below will collect all tests with the word pretend or default in the test name
pytest -k "pretend or default" --collect-only

# Use markers to group tests together and run them
@pytest.mark.group_1 
def test_number_1(): pass

@pytest.mark.group_1
def test_number_2(): pass

@pytest.mark.group_2
def test_number_3(): pass

pytest -m group_1 # Will run tests 1 and 2
pytest -m "group_1 and group_2" # Will run tests 1,2 and 3
pytest -m "group_1 and not group_2" # Will run tests 1 and 2

# Exit pytest as soon as a failing test is found. Useful for debugging
pytest -x

# Display the output (e.g. print statements) inside a test. Especially useful when building tests and you want to "log" and see the output
def test_with_print():
    print("Yes this printed to the terminal when the test ran")

pytest test_with_print -s

# Can also display all local variables in a failing test
pytest -l

# Re-run only the last tests which failed. Very useful for debugging
pytest –lf

# Skipping tests (e.g. misunderstood an API and the test does not make sense at all)
@pytest.mark.skip(reason="Insert a good reason for ppl to understand")
def test_with_print():
    print("Yes this printed to the terminal when the test ran")

# Skip tests under conditions (e.g. only applied to certain versions of the API)
@pytest.mark.skipif(tasks.__version__ < '0.2.0', reason='not supported until version 0.2.0')
def test_with_print():
    print("Yes this printed to the terminal when the test ran")

# Run tests based on pattern in test name
pytest -v -k pattern_goes_here
pytest -v -k "matched_pattern and not unmatched_pattern"
{% endhighlight %}

{% highlight python %}
# Title
code
{% endhighlight %}

{% highlight python %}
# Title
code
{% endhighlight %}

{% highlight python %}
# Title
code
{% endhighlight %}


## References
The above notes are from a variety of sources, including, but not limited to:
* [Improve Your Understanding of Unit Testing ](https://jeffknupp.com/blog/2013/12/09/improve-your-python-understanding-unit-testing/)
* Python Testing with pytest by Brian Okken


The intellectual property of the below notes and resources belong to the sources which they were derived. 