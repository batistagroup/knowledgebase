# Introduction

This guide introduces static typing in Python, covering its benefits, usage, and best practices.

## Why Use Typing?

1. **Improved Code Readability**

   - Type hints serve as documentation, clarifying the expected data types.
   - Makes code easier to understand and maintain.

1. **Early Error Detection**

   - Type checkers like MyPy can identify type-related errors before runtime.
   - Reduces the likelihood of unexpected behavior in production.

1. **Enhanced Code Maintainability**

   - Facilitates refactoring and code evolution.
   - Provides a safety net when making changes to existing code.

## What to Type

### 1. Function Signatures

Type hints specify the expected types of function arguments and the return value.

```python
def add(x: int, y: int) -> int:
    """Adds two integers and returns the result."""
    return x + y
```

### 2. Variable Annotations

```python
pi: float = 3.14159
name: str = "Alice"
```

### 3. Data Structures

Type hints can also be used to define complex data structures. For better organization and maintainability, it's best practice to define these types in a separate file (e.g., `utils/type_definitions.py`). This keeps your type definitions centralized and reusable across your project.

For example:

```python
# utils/type_definitions.py
from typing import TypeAlias

BondType: TypeAlias = tuple[int, int]
BondsType: TypeAlias = list[BondType]
AtomType: TypeAlias = tuple[str, float, float, float]
AtomsType: TypeAlias = list[AtomType]
```

Then, you can import and use these types elsewhere:

```python
from utils.type_definitions import BondType, BondsType, AtomType, AtomsType

bond: BondType = (1, 2)
bonds: BondsType = [(1, 2), (2, 3), (3, 1)]
atom: AtomType = ("H", 1.0, 2.0, 3.0)
atoms: AtomsType = [("H", 1.0, 2.0, 3.0), ("O", 1.5, 2.5, 3.5)]
```

This approach improves code readability and maintainability, especially in larger projects. Using `TypeAlias` provides clarity and allows for better type checking.

### 4. Class Attributes

```python
class Circle:
    radius: float
    color: str
    is_filled: bool
```

## Typing Concepts

### 1. Basic Types

- `int`, `float`, `str`, `bool`
- `None` (for `NoneType`)
- `Any` (for dynamically typed code)

### 2. Collection Types

- `list[T]`, `set[T]`, `tuple[T, ...]`, `dict[K, V]`
- `List`, `Set`, `Tuple`, `Dict` (lowercase built-in counterparts are now preferred for type hints)

### 3. Union Types

The `Union` type hint, from the `typing` module, specifies that a variable can accept one of several types. While `Union` remains functional, modern Python style prefers the pipe operator (`|`) for expressing union types. This is generally considered more concise and readable.

```python
from typing import (
    Union,
)  # Union is still useful for backward compatibility and in some contexts


def process_data(data: int | float | str) -> None:
    """Processes data that can be an int, float, or string."""
    print(f"Processing: {data}")


def process_data_union(data: Union[int, float, str]) -> None:
    """Processes data that can be an int, float, or string using Union."""
    print(f"Processing (using Union): {data}")
```

### 4. Optional Types

```python
from typing import Optional


def greet(name: Optional[str] = None) -> None:
    """Greets the user, optionally by name."""
    if name:
        print(f"Hello, {name}!")
    else:
        print("Hello!")
```

### 5. Callable Types

Callable type hints describe functions. The syntax `Callable[[arg1_type, arg2_type,...], return_type]` specifies the argument types and the return type.

```python
from typing import Callable


def apply_function(func: Callable[[int], int], value: int) -> int:
    """Applies a function to a value."""
    return func(value)


def square(x: int) -> int:
    return x * x


result = apply_function(square, 5)
```

### 6. Type Aliases

Type aliases provide a way to give a more descriptive name to an existing type. This improves code readability and maintainability, especially when dealing with complex types.

```python
# Define a type alias for a list of floats representing a vector
Vector: list[float]


def scale_vector(vector: Vector, scalar: float) -> Vector:
    """Scales a vector by a scalar."""
    return [scalar * x for x in vector]  # Corrected the multiplication operator
```

### 7. Generics

Generics allow you to write functions and data structures that can work with different types without losing type information. The `TypeVar` class from the `typing` module is used to define type variables.

```python
from typing import TypeVar, Optional

# Define a type variable T
T = TypeVar("T")


def first_element(data: list[T]) -> Optional[T]:
    """Returns the first element of a list.  Returns None if the list is empty."""
    if data:  # Added check for empty list
        return data[0]
    else:
        return None  # Handle empty list case
```

## Tools and Techniques

### 1. MyPy

- Static type checker for Python
- Integrates with popular IDEs
- Can be run from the command line

### 2. Type Stubs

- `.pyi` files that provide type hints for existing libraries
- Used when the original library does not include type hints

### 3. Gradual Typing

- Gradually add type hints to your codebase
- Start with critical sections and expand over time

## Best Practices

1. **Be Consistent**

   - Follow a consistent style for type hints
   - Use a linter to enforce consistency

1. **Type Function Signatures**

   - Always type function arguments and return values
   - Improves code clarity and reduces errors

1. **Use Descriptive Names**

   - Use meaningful names for type variables and aliases
   - Enhances code readability

1. **Keep Type Hints Up-to-Date**

   - Update type hints when the code changes
   - Ensures that the type hints remain accurate

## Common Pitfalls

1. **Overuse of `Any`**

   - Avoid using `Any` unless absolutely necessary
   - Reduces the benefits of static typing

1. **Ignoring Type Errors**

   - Treat type errors as serious issues
   - Fix them promptly to prevent runtime errors

1. **Complex Type Hierarchies**

   - Keep type hierarchies simple and understandable
   - Avoid over-engineering type definitions

## When to Skip Typing

While typing is generally beneficial, some cases might not require it:

1. **Small Scripts**

   - Simple scripts with limited functionality
   - Where the benefits of typing are minimal

1. **Prototyping**

   - Early-stage prototyping where code is rapidly changing
   - Typing can add overhead without significant benefit

1. **Legacy Code**

   - Codebases where adding type hints is impractical
   - Due to the size or complexity of the code
   - For example, integrating type hints into a large, established project like RDKit might require significant refactoring and could be a substantial undertaking. Prioritize adding types to new code or critical sections of legacy code (perhaps by type stubs) rather than attempting a complete overhaul.

## Conclusion

Typing in Python enhances code readability, enables early error detection, and improves maintainability. By adopting typing best practices and utilizing tools like MyPy, you can write more robust and reliable Python code.
