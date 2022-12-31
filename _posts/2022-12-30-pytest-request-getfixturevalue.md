---
layout: post
title:  "[pytest] Use fixtures in the test parameter set in `parametrize`"
date:   2022-12-31 18:19:00 +0100
categories: pytest
---
### HOW TO: Use fixtures in the parameter set in `parametrize`.

> TL; DR: Call fixture with `request.getfixturevalue("fixture_name")`


Sometimes it is useful to use fixtures as parameters in `parametrize`. Let's say we want to run a test for two different test environments. Before the test is executed, we want to call the respective setup fixture. So, we make two fixtures:

```
@pytest.fixture
def setup_env1():
    print("Using environment 1")
    return 1


@pytest.fixture
def setup_env2():
    print("Using environment 2")
    return 2
```

And try to parametrize the test:

```
@pytest.mark.parametrize("env", [setup_env1, setup_env2])
def test_with_parametrize_fixture(env):
    print("Testing in environment", env)
```

However, this won't work as intended:

```
test_fixtures_basic.py::test_with_parametrize_fixture[setup_env1] Testing in environment <function setup_env1 at 0x7fcbebc4ad30>
PASSED
test_fixtures_basic.py::test_with_parametrize_fixture[setup_env2] Testing in environment <function setup_env2 at 0x7fcbebc4ae50>
PASSED
```

### SOLUTION

There's an alternative method to invoke a fixture: `request.getfixturevalue("fixture_name")`, which runs a fixture with a name (string) provided as an input.

With this method, we provide a fixture name just as any other string test parameter.

```
@pytest.mark.parametrize("env", ["setup_env1", "setup_env2"])
def test_with_parametrize_fixture_ok(env, request):
    print("Testing in environment", request.getfixturevalue(env))
```

This time it works as expected:

```
test_fixtures_basic.py::test_with_parametrize_fixture_ok[setup_env1] Using environment 1
Testing in environment 1
PASSED
test_fixtures_basic.py::test_with_parametrize_fixture_ok[setup_env2] Using environment 2
Testing in environment 2
PASSED
```
