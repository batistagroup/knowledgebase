# Introduction to Python Type Hints

## Overview

Consider the following code block.

```python
def process_smiles(smiles):
    # a lot of lines of code
    ...
```

You probably can infer that this function expects SMILES and performs some operations on them. But does it take one SMILES string or does it take a list of those strings? Or can it take both? Compare this to:

```py
def process_smiles_v1(smiles:str) -> float:
    ...
# or 
def process_smiles_v2(smiles:list[str]) -> list[int]:
    ...
```

although same information could be stored in a docstring, a type annotation is like a shortcut. As a side bonus, if you try to write:

```py
process_smiles_v2("CCO")
```

a type checker will throw an error which will allow you to catch this bug even before you run the code.

!!! tip "Extra benefits of typing"

    Besides catching errors early and improving readability of your code, a huge benefit of type hints is that it improves your experience with LLMs. Higher quality codebases tend to have type annotations, so if your existing code has the type hints, it'll direct LLM into the portion of the latent space that corresponds to higher code quality (just like a math problem written with LaTeX symbols has a higher chance of getting correct result).

!!! info Prerequisites

    - Python 3.9+
    - Basic Python knowledge
    - A type checker (mypy, pytype, or Pyright). I tend to use mypy

## Basic Usage

### 1. Simple Type Annotations

Type annotations are optional when the type can be inferred from the initial value.

```python
name = "Alice"  # Type inferred as str
age = 25  # Type inferred as int
is_active = True  # Type inferred as bool
```

However, explicit annotations are recommended for empty collections or when type is ambiguous

```py
users: list[str] = [] # list of strings
cache: dict[str, int] = {} # dict mapping str keys to int vals
numbers: list[float | int] = []    

# Function annotations are always recommended for clarity
def greet(name: str) -> str:
    return f"Hello, {name}!"

def multiply(x: int, y: int) -> int:
    return x * y
```

### 2. Built-in Collections

```python
# Lists
numbers: list[int] = [1, 2, 3]


def get_users() -> list[str]:
    return ["Alice", "Bob", "Charlie"]


# Dictionaries
scores: dict[str, int] = {"Alice": 95, "Bob": 85}


def get_config() -> dict[str, str]:
    return {"host": "localhost", "port": "8080"}


# Sets and Tuples
valid_codes: set[int] = {200, 201, 204}
point: tuple[float, float] = (2.0, 3.0)
```

### 3. Nullable Types and Default Values

If a variable may have type A or B, a union operator `|` can be used:

```python
# Parameters that might be None
def find_user(user_id: str | None) -> str:
    if user_id is None:
        return "Guest"
    return f"User {user_id}"
```

default values can be specified after the type:

```py
def connect(host: str = "localhost", port: int = 8080) -> bool:
    return True
```

!!! info "Old type annotation syntax"

    Before python 3.9, a union of types was shown as `Union[str, None]`. An optional value, e.g. `user_id: str | None = None` was written `user_id: Optional[str] = None` with an import `from typing import Union, Optional`. You might see this syntax in older codebases; the new one is recommended. If you're up for a rabbit hole, python core devs had a [holy war](https://discuss.python.org/t/clarification-for-pep-604-is-foo-int-none-to-replace-all-use-of-foo-optional-int/26945) over this change.

### 4. Multiple Return Types

```python
def parse_value(value: str) -> int | float:
    """Parse a string into either an integer or float."""
    try:
        return int(value)
    except ValueError:
        return float(value)


def fetch_user_data(user_id: int) -> dict[str, str | int] | None:
    """Fetch user data, returns None if user not found."""
    if user_id < 0:
        return None
    return {"name": "Alice", "age": 25}
```

### 5. Type Aliases

Type aliases help manage complex type annotations and make code more readable:

```python
# Type aliases for complex types
UserId = int
UserDict = dict[UserId, str]
JsonValue = str | int | float | bool | None
JsonObject = dict[str, JsonValue]


def process_json(data: JsonObject) -> list[str]:
    """Process a JSON object and return list of keys with string values."""
    return [k for k, v in data.items() if isinstance(v, str)]


def get_user_data() -> UserDict:
    return {1: "Alice", 2: "Bob"}
```

## Type Checking in Practice

### Static vs Runtime Type Checking

Type hints enable static analysis of your code before execution. This creates two distinct phases of type checking:

1. **Static Analysis** (Before Runtime):

    - Checks type consistency
    - Validates function signatures
    - Ensures type-safe operations
    - No performance impact at runtime

1. **Runtime Behavior** (During Execution):

    - Python's normal dynamic type checking
    - No overhead from type hints (they're ignored)
    - Actual type enforcement

Here's how this dual system works:

```python
def process_items(items: list[str]) -> dict[str, int]:
    """Process a list of strings and return their lengths."""
    return {item: len(item) for item in items}


# Static type checker catches these errors:
process_items([1, 2, 3])  # Error: list[int] is not list[str]
process_items("not a list")  # Error: str is not list[str]


# Runtime errors (not caught by static checker):
def load_and_process_data(filepath: str) -> dict[str, int]:
    """Load data from a file and process it.

    The type checker trusts our annotation that the file contains strings,
    but at runtime, the file might contain any type of data.
    """
    with open(filepath) as f:
        # Type checker assumes this creates List[str] based on annotation
        items: list[str] = f.read().splitlines()
        return process_items(items)


# These will pass type checking but might fail at runtime:
load_and_process_data("data.txt")
# Runtime error if file contains non-string data or if file doesn't exist
```

The key difference is:

- Static type checking validates code structure and explicit type annotations
- Runtime checks validate actual data values and operations
- External data (files, network, user input) can't be verified by the type checker

### Common Type Checking Scenarios

Here are realistic examples of issues that type checkers catch:

```python
# Function argument mismatches
def calculate_average(numbers: list[float]) -> float:
    return sum(numbers) / len(numbers)


data = ["1", "2", "3"]  # Error: list[str] incompatible with List[float]
calculate_average(data)


# Incorrect attribute access
class User:
    def __init__(self, name: str):
        self.name = name


def print_email(user: User) -> None:
    print(user.email)  # Error: 'User' has no attribute 'email'


# Type narrowing errors
def process_value(value: str | int) -> str:
    if isinstance(value, int):
        return value.upper()  # Error: 'int' has no attribute 'upper'
    return value.upper()


# Container type mismatches
cache: dict[str, list[int]] = {
    "numbers": ["1", "2", "3"]  # Error: list[str] incompatible with list[int]
}


# Function return type violations
def get_user_ids() -> list[int]:
    users = {"1": "Alice", "2": "Bob"}
    return list(users.keys())  # Error: list[str] incompatible with list[int]
```

## Best Practices

1. **Always Annotate Public APIs**

    - Functions and methods exposed to other modules
    - Class attributes and properties
    - Module-level variables

1. **Use Type Aliases for Complex Types**

    ```python
    from typing import TypeAlias

    JsonDict: TypeAlias = dict[str, str | int | float | bool | None]
    ```

1. **Handle Optional Types Safely**

    ```python
    def process_data(data: str | None) -> str:
        if data is None:
            return "No data"
        return data.upper()
    ```

1. **Document Type Variables**

    ```python
    from typing import TypeVar

    T = TypeVar("T", str, int)  # T can be str or int


    def first_element(lst: list[T]) -> T:
        return lst[0]
    ```

## Configuring mypy

Enabling an automatic type checker for your codebase is quite easy. Simply `uv add mypy` in your [venv](/docs/python/essential-tools.md), and then you can run `mypy .`. Once your codebase satisfies basic rules (checked by `mypy .` call), you can try & run `mypy --strict .`.

You can also update your `pyproject.toml` (alredy done in [python-template](https://github.com/batistagroup/python-template/)) to have strict enabled by default.

```toml
[tool.mypy]
strict = true
exclude = ["tests"]                    # specifies which directories are not checked
ignore_missing_imports = true          # so that it doesn't complain about types in external libraries
disable_error_code = ["unused-ignore"]
```

## Summary

Type hints are a powerful tool for improving code clarity, reducing bugs, and ensuring robust software development. Whether you're working on a small script or a large-scale project, leveraging Python's type system can significantly enhance your development process.
