# Python Code Linting with Ruff

## Overview

Ruff is a fast, Python-native code linter that helps maintain code quality and consistency across your Python projects. It replaces multiple traditional Python linters (like flake8, pylint, isort) with a single, blazingly fast tool written in Rust.

## Why Ruff?

- **Speed**: Up to 100x faster than traditional Python linters
- **All-in-One**: Combines functionality of multiple linters in a single tool
- **Extensible**: Rich set of rules and easy configuration
- **Auto-fixing**: Can automatically fix many common issues
- **Modern**: Supports the latest Python features and best practices

## Prerequisites

- Python 3.7+
- A Python project with `pyproject.toml` configuration

## Installation

```bash
uv add ruff
```

## Configuration

Our [template](https://github.com/batistagroup/python-template/) project uses Ruff with the following configuration in `pyproject.toml`:

```toml
[tool.ruff]
line-length = 120
lint.select = ["E", "F", "UP", "B", "SIM", "I"]
```

### Rule Sets Explained

1. **E (pycodestyle)**

    - Enforces PEP 8 style guide
    - Handles spacing, naming, and basic code style

1. **F (Pyflakes)**

    - Detects logical errors
    - Finds unused imports and variables
    - Identifies undefined names

1. **UP (pyupgrade)**

    - Upgrades syntax to newer Python versions
    - Modernizes code constructs

1. **B (flake8-bugbear)**

    - Catches common bugs and design problems
    - Enforces best practices

1. **SIM (flake8-simplify)**

    - Suggests code simplifications
    - Improves code readability

1. **I (isort)**

    - Sorts and organizes imports
    - Maintains consistent import ordering

## Usage

### Basic Commands

1. Check your code:

    ```bash
    ruff check .
    ```

1. Automatically fix issues:

    ```bash
    ruff check --fix .
    ```

### Integration with Development Tools

See page on [pre-commits](/docs/git/pre-commit.md).

## Ruff Formatting

Perhaps one of the first effects of ruff you'll notice is that it'll break long lines. By default, Ruff follows Black's config and limits number of characters to 88, I personally extend it to 120. This change might feel very counterintuitive to you (you'll see what I mean in practice), but if you trust it for awhile, you'll start seeing the benefits. A few examples.

```python
# Example 1: Complex Boolean Logic
# Bad - Hard to understand the logic and spot errors
if (
    user.is_active
    and user.email_verified
    and (user.role == "admin" or user.role == "moderator")
    and user.has_permission("edit_posts")
    and not user.is_banned
):
    allow_access()

# Good - Logic is clear and errors are easy to spot
if (
    user.is_active
    and user.email_verified
    and user.role in ("admin", "moderator")
    and user.has_permission("edit_posts")
    and not user.is_banned
):
    allow_access()

# Example 2: Error Handling
# Bad - Exception types are hard to read and modify
try:
    process_data()
except (
    ValueError,
    TypeError,
    KeyError,
    DatabaseError,
    NetworkTimeout,
    ValidationError,
) as e:
    log_error(e)

# Good - Each exception is clear and git diffs will show exactly what changed
try:
    process_data()
except (
    ValueError,
    TypeError,
    KeyError,
    DatabaseError,
    NetworkTimeout,
    ValidationError,
) as e:
    log_error(e)

# Example 3: Function Calls with Named Parameters
# Bad - Parameter names and values are hard to scan
create_user(
    username="johndoe",
    email="john@example.com",
    role="admin",
    department="engineering",
    active=True,
    send_welcome_email=True,
)

# Good - Each parameter is clearly visible and self-documenting
create_user(
    username="johndoe",
    email="john@example.com",
    role="admin",
    department="engineering",
    active=True,
    send_welcome_email=True,
)
```

The benefits of proper line formatting become obvious when you need to:

- Debug complex boolean conditions
- Review which exceptions are being caught
- Understand the parameters being passed to a function
- Track changes in git history (each parameter change appears on its own line)

Note the trailing comma after the last item in multi-line structures - this is a deliberate practice that makes future modifications cleaner in version control and prevents syntax errors when reordering lines.

## Best Practices

1. Run Ruff locally before committing changes
1. Configure your IDE for automatic linting on save
1. Use [pre-commit hooks](/docs/git/pre-commit.md) to enforce linting rules
1. Regularly update Ruff to get the latest rules and fixes

## Related Resources

- [Ruff Documentation](https://docs.astral.sh/ruff/)
- [PEP 8 Style Guide](https://peps.python.org/pep-0008/)
- [Pre-commit Hooks](https://pre-commit.com/)
