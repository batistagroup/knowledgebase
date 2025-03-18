# Pre-commit Workflows

After reading docs on [linting & formatting](../python/lint.md) and [typing](../python/typing/intro.md), your [committing](../git/intro.md) workflow will look like:

```bash
ruff check --fix
ruff format
mypy .
git add .
git commit -m "feat: some new feature"
```

this might get tiresome, and it doesn't have to be that way. Enter pre-commit hooks - defined workflow that run when you try to commit something. With a `.pre-commit-config.yaml` (our [template](https://github.com/batistagroup/python-template/) has one) you can have ruff & mypy executed automatically when you make a commit:

```bash
git add .
git commit -m "feat: some new feature"
```

if the pre-commit hook will change formatting (and so the pre-commit check will fail), you'll have to run the two commands above again. Done.

### Setup Instructions

1. Make sure pre-commit are installed

    ```bash
    uv add pre-commit
    ```

1. Install the hooks:

    ```bash
    pre-commit install
    ```

### Common Issues and Solutions

- **Hook fails to install**: Ensure Python 3.12 is available in your environment
- **Ruff errors**: Run `ruff check --fix` manually to see detailed errors
- **MyPy errors**: Add type annotations to resolve strict type checking issues

### Best Practices

1. Run `pre-commit run --all-files` after installing to verify setup
1. Commit frequently to catch issues early
1. Update hooks regularly with `pre-commit autoupdate`

### Related Resources

- [Pre-commit Documentation](https://pre-commit.com/)
- [Ruff Documentation](https://docs.astral.sh/ruff/)
- [MyPy Documentation](https://mypy.readthedocs.io/)
