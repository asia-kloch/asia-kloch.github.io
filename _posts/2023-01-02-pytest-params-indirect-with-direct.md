---
layout: post
title:  "[pytest] Indirect and direct together in parametrize"
date:   2023-01-02 16:42:00 +0100
categories: pytest
---
### HOW TO: Use direct test parametrization together with the indirect fixture parametrization.

> TL; DR: `@pytest.mark.parametrize("my_fixture, ...", ..., indirect=["my_fixture"])`

It is possible to mix direct and indirect parametrization, i.e. to parametrize the test in a way that some parameters are passed to it directly and some are passed to the fixture(s) used by this test.

First, let's make two dummy fixtures, which will receive parameters from the test.

```
@pytest.fixture
def my_fixture_1(request):
    a, b = request.param
    print(f"In my_fixture_1: params {a}, {b}")
    return a, b, a * b


@pytest.fixture
def my_fixture_2(request):
    print(f"In my_fixture_2: {request.param}")
```

Note that the fixture `my_fixture_1` returns the input arguments as well. Sometimes in my tests I need to access the parameters in both fixture and the test function and this is how I do it. I don't know if there's another way to access the parameters passed as `indirect` from the test level. Maybe using `request` could work[^note].
 
[^note]: There is actually something in `request._fixture_defs["my_fixture_1"].cached_result`, but this definitely doesn't look like an easy way :D


Now let's parametrize the test: first parameter is a tuple for the fixture `my_fixture_1`, second one is the parameter used by the test directly and the third one is the indirect parameter for `my_fixture_2`. As we see, there's no rule such as "all direct parameters must come before indirect ones", but I typically keep them ordered. Side note: the order of the names in the `indirect` list doesn't have to match the order of the indirect parameters in the arguments list - but it is more intuitive to use the same order ;)

```
@pytest.mark.parametrize(
    "my_fixture_1, expected, my_fixture_2",
    [[(1, 2), 2, "NOPE"], [(3, 4), 12, "STILL NOPE"]],
    indirect=["my_fixture_1", "my_fixture_2"],
)
def test_indirect_and_direct(my_fixture_1, expected, my_fixture_2):
    a, b, result = my_fixture_1
    print(
        f"In test: test input (from fixture) {a}, {b}, {result}, "
        f"expected (direct param) {expected}"
    )
    assert result == expected
```

This results in:
```
test_fixtures_indirect.py::test_indirect_and_direct[my_fixture_10-2-NOPE] In my_fixture_1: params 1, 2
In my_fixture_2: NOPE
In test: test input (from fixture) 1, 2, 2, expected (direct param) 2
PASSED
test_fixtures_indirect.py::test_indirect_and_direct[my_fixture_11-12-STILL NOPE] In my_fixture_1: params 3, 4
In my_fixture_2: STILL NOPE
In test: test input (from fixture) 3, 4, 12, expected (direct param) 12
PASSED

```
