# pytest-unordered: Test collection content, ignoring order

[![Build Status](https://travis-ci.org/utapyngo/pytest-unordered.svg?branch=master)](https://travis-ci.org/utapyngo/pytest-unordered)
[![Coverage Status](https://codecov.io/gh/utapyngo/pytest-unordered/branch/master/graph/badge.svg)](https://codecov.io/gh/utapyngo/pytest-unordered)

`pytest_unordered` allows you to write simple (pytest) assertions
to test whether collections have the same content, regardless of order.
For example:

    assert [1, 20, 300] == unordered([20, 300, 1])


## Installation

    pip install pytest-unordered


## Usage

### Basics

In most cases you just need the `unordered()` helper function:

    from pytest_unordered import unordered

Compare list or tuples by wrapping your expected value with `unordered()`:

    assert [1, 20, 300] == unordered([20, 300, 1])  # Pass
    assert (1, 20, 300) == unordered((20, 300, 1))  # Pass

Excessive/missing items will be reported by pytest:

    assert [1, 20, 300] == unordered([20, 300, 1, 300])

      E         Extra items in the right sequence:
      E         300

By default, the container type has to match too:

    assert (1, 20, 300) == unordered([20, 300, 1])

      E         Type mismatch:
      E         <class 'tuple'> != <class 'list'>



### Nesting

A seasoned developer will notice that the simple use cases above
can also be addressed with appropriate usage
of builtins like `set()`, `sorted()`, `isinstance()`, `repr()`, etc,
but these solutions scale badly (in terms of boilerplate code)
with the complexity of your data structures.
For example: naively implementing order ignoring comparison
with `set()` or `sorted()` does not work with lists of dictionaries
because dictionaries are not hashable or sortable.
`unordered()` supports this out of the box however:

    assert [{"bb": 20}, {"a": 1}] == unordered([{"a": 1}, {"bb": 20}])  # Pass


The true value of `unordered()` lies in the fact that you
can apply it inside large nested data structures to skip order checking
only in desired places with surgical precision
and without a lot of boilerplate code.
For example:

    expected = unordered([
        {"customer": "Alice", "orders": unordered([123, 456])},
        {"customer": "Bob", "orders": [789, 1000]},
    ])

    actual = [
        {"customer": "Bob", "orders": [789, 1000]},
        {"customer": "Alice", "orders": [456, 123]},
    ]

    assert actual == expected

In this example we wrapped the outer customer list and the order list of Alice
with `unordered()`, but didn't wrap Bob's order list.
With the `actual` value of above (where customer order is different
and Alice's orders are reversed), the assertion will pass.
But if the orders of Bob would be swapped in `actual`, the assertion
will fail and pytest will report:

    E         Differing items:
    E         {'orders': [1000, 789]} != {'orders': [789, 1000]}



### Container type checking

As noted, the container types should be (by default) equal to pass the
assertion. If you don't want this type check, call `unordered()`
in a variable argument fashion (instead of passing
a container as single argument):

    assert [1, 20, 300] == unordered(20, 300, 1)  # Pass
    assert (1, 20, 300) == unordered(20, 300, 1)  # Pass

This pattern also allows comparing with iterators, generators and alike:

    assert iter([1, 20, 300]) == unordered(20, 300, 1)  # Pass
