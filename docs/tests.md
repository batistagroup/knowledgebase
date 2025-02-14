# Testing Scientific Code with Pytest

This guide introduces best practices for testing scientific Python code using the `pytest` framework. We'll cover various testing strategies, focusing on writing clear, concise, and maintainable tests.

## Why Test Scientific Code?

Testing is crucial for ensuring the correctness, reliability, and reproducibility of scientific computations. Scientific code often involves complex algorithms, numerical methods, and data processing pipelines, making thorough testing essential to avoid errors and ensure confidence in the results. Benefits include:

1. **Early Error Detection:** Catch bugs early in the development process, preventing them from propagating into larger issues.
1. **Improved Code Quality:** Writing tests forces you to think critically about your code's design and functionality, leading to more robust and maintainable code.
1. **Reproducibility:** Tests provide a documented record of your code's behavior, making it easier to reproduce results and verify the correctness of future modifications.
1. **Regression Prevention:** Tests act as a safety net, preventing regressions (the reintroduction of previously fixed bugs) when making changes to your code.
1. **Collaboration:** Well-written tests facilitate collaboration by providing a clear understanding of the code's expected behavior.

## Testing Strategies

### 1. Unit Tests

Unit tests focus on testing individual components (functions, classes, modules) of your code in isolation. This allows you to pinpoint the source of errors more easily.

```python
# src/package/utils/functions.py
def add(x, y):
    return x + y
```

```python
# tests/test_functions.py
import pytest
from package.utils.functions import add


def test_add():
    assert add(2, 3) == 5
    assert add(-1, 1) == 0
    assert add(0, 0) == 0
```

### 2. Integration Tests

Integration tests verify the interaction between different components of your code. They ensure that the components work together correctly as a system.

```python
# src/package/utils/other_functions.py
def subtract(x, y):
    return x - y
```

```python
# tests/test_integration.py
import pytest
from package.utils.functions import add
from package.utils.other_functions import subtract


def test_integration():
    # test add and subtract
    assert add(subtract(5, 3), 2) == 4
```

### 3. End-to-End Tests

End-to-end tests verify the entire system flow from start to finish. They are useful for testing complex interactions and ensuring that all components work together correctly.

### 4. Regression Tests

Regression tests are designed to prevent the reintroduction of previously fixed bugs. They are typically run after making changes to the codebase to ensure that existing functionality remains intact. Best practice dictates incorporating edge cases discovered during testing into existing unit tests, rather than creating entirely separate regression test suites. This keeps the test suite concise and maintainable.

### 5. Performance Tests

Performance tests measure the speed and efficiency of the system. They are useful for identifying performance bottlenecks and ensuring that the system meets performance requirements.

## Best Practices for Multi-Case Testing with `@pytest.mark.parametrize`

The `@pytest.mark.parametrize` decorator is a powerful tool in `pytest` for writing concise and readable multi-case tests. It allows you to run the same test function with different inputs and expected outputs.

Let's say we have a function that calculates the square of a number:

```python
def square(x):
    return x * x
```

We can test this function with multiple inputs using `@pytest.mark.parametrize`:

```python
import pytest


def square(x):
    return x * x


TEST_CASES_SQUARE = [
    (1, 1, "Positive integer"),
    (0, 0, "Zero"),
    (-2, 4, "Negative integer"),
    (3.14, 9.8596, "Float"),
]


@pytest.mark.parametrize(
    "input, expected, description",
    [(tc[0], tc[1], tc[2]) for tc in TEST_CASES_SQUARE],
)
def test_square(input, expected, description):
    assert square(input) == pytest.approx(expected)
```

This approach is significantly more efficient and readable than writing separate test functions for each case. The `pytest.approx` function handles floating-point comparisons, accounting for potential rounding errors. Including a description for each test case improves readability and maintainability.

## Tools and Techniques

- **`pytest`:** A powerful and flexible testing framework for Python. To run all tests, use `pytest`. For verbose output, use `pytest -v`. To run tests in a specific file, use `pytest tests/test_file.py`. You can also specify individual test functions or classes using the `-k` option (e.g., `pytest -k "test_function"`). `pytest` automatically discovers tests based on naming conventions (files starting with `test_` or classes/functions containing `test` in their names).
- **`pytest-cov`:** A pytest plugin that generates code coverage reports. Install `pytest-cov` and run it with `pytest --cov=.`.
- **`hypothesis`:** A library for property-based testing, enabling you to test a wide range of inputs automatically.

## Best Practices

1. **Write Tests First (Test-Driven Development):** Before writing the actual function, consider writing the tests first. This approach, known as Test-Driven Development (TDD), helps clarify requirements and ensures that your code meets its intended purpose.
1. **Write Clear and Concise Tests:** Tests should be easy to understand and maintain.
1. **Test Edge Cases:** Pay attention to boundary conditions and potential error scenarios.
1. **Use Descriptive Test Names:** Test names should clearly indicate what is being tested.
1. **Keep Tests Independent:** Tests should not depend on each other.
1. **Use Assertions Effectively:** Use assertions to clearly state the expected behavior of your code.
1. **Aim for High Code Coverage:** Strive for high code coverage to ensure that most parts of your code are tested.

## Conclusion

Testing is an integral part of developing robust and reliable scientific code. By adopting the best practices outlined in this guide and utilizing tools like `pytest`, you can significantly improve the quality and reproducibility of your scientific computations.
