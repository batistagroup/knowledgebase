# Python Starter Pack

## Overview

This guide provides a modern, efficient approach to Python project management using `uv`, a high-performance replacement for traditional tools like pip, pyenv, and poetry. It's designed for Python developers who want to streamline their development workflow and adopt current best practices for dependency management.

!!! info "Prerequisites"

    - Basic familiarity with Python and command line
    - Basic understanding of virtual environments and package management
    - macOS, Linux, or Windows system

## Choosing uv as a package manager

[uv](https://docs.astral.sh/uv/) is a python project and package manager that replaces `pip`, `pyenv`, `poetry`, and mostly `conda`. It's faster, simpler, and better.

### Why not conda?

You can have many anti-conda rants on the internet. It's bulky, it has complicated installation process that easily tricks you into installing the full anaconda suit with all the things you never need. Pragmatically, each time you create a new virtual environment, it installs a new python in that env, so if you have 5 different projects each using python 3.12.8, you will have 5 copies of python 3.12.8 installed. Why would you want that?

### Why not pyenv + venv?

A more reasonable alternative, until Feb 2024, was using pyenv, which installs global versions of python that you can easily switch between, and then creating `venv` using python native `python -m venv venv`. Your workflow was:

```bash
pyenv local 3.12
python -m venv venv
source venv/bin/activate
```

### So why uv?

uv basically mirrors workflow above, but makes everything faster.

```bash
uv venv --python 3.12
source .venv/bin/activate
```

As a bonus, [installing uv](https://docs.astral.sh/uv/getting-started/installation/) is incredibly easy, just one curl script that is easily found in docs without requiring you to go through 10 accordeons and tabs.

## Starting a new project

After creating and `cd` into your project directory, you can run `uv init`, which will create:

- `.python-version` - a simple text file specifying python version you chose when running `uv venv --python 3.12`. If a project has `.python-version` specified, you can simply run `uv venv` and it'll use (or install if that python is not installed already) python version specified in `.python-version`.
- `pyproject.toml` - a modern replacement to `setup.py`, `requirements.txt`.
- `main.py` - your first `.py` file.

See [python-template repo](https://github.com/batistagroup/python-template) for an example of a filled `pyproject.toml`.

## Creating your package

Choosing when to structure your code as a package is ultimately a matter of preference. As of today, I prefer to create a package from the very beginning. Simply because:

- if it's a research project, I will release the code regardless of whether it truly is a standalone package (i.e. other people will use it in their projects as a dependency) just for reproducibility reasons
- given that `uv` uses `pyproject.toml` to enumerate dependencies already, it's a matter of a few extra lines to convert codebase into a package
- having a central package removes the necessity to either explicitly add path to sys or use some complicated relative path imports. If you had these issues before, you know what I mean.

### Why should I put my package under src/

You might notice that [python-template repo](https://github.com/batistagroup/python-template) or [DirectMultiStep](https://github.com/batistagroup/DirectMultiStep) hides the package files under `src/` directory. This is a good practice, because, if you have the following structure:

```md
my-project/
├── pyproject.toml
├── README.md
├── my_package/
│   ├── __init__.py
│   └── module.py
└── script.py
```

when you install `my_package` in development mode (e.g. `uv pip install -e .`), the project root directory (`my-project/`) is added to the python path. This means that any file in your project root can be accidentally imported as part of your package. For example, if you have a `script.py` in your project root, it could be mistakenly imported as `my_package.test`.

By placing your package under `src/`, like this:

```md
my-project/
├── pyproject.toml
├── README.md
├── src/
│   └── my_package/
│       ├── __init__.py
│       └── module.py
└── tests/
```

and configuring your `pyproject.toml` accordingly, only the `src/` directory is added to the python path during development installation. This clearly separates your package code from other project files (like configuration, tests, documentation, etc.) and prevents potential naming conflicts and accidental imports from the project root. It's a cleaner and more robust way to structure your project.

### Installing Dependencies

#### Your package

Once you've created some package in `src/`, all that remains is to add this to your `pyproject.toml`:

```toml
[tool.setuptools]
packages = ["my_package"]
package-dir = { "" = "src" }
```

!!! warning "Don't forget to activate your venv"

    Once you have your project set up, your venv should be pretty much always activated. Your IDE (VSCode, Cursor) should automatically recognize presence of `.venv` folder and activate it in the terminal sessions. But if you open your repository in a standalone terminal window, you'll have to manually `source .venv/bin/activate`.

After this, you can install your package in development mode `uv pip install -e .`. The `-e` specifies dev mode and effectively it means that any changes made to the source code in `src/my_package` will be immediately available (i.e. you don't need to reinstall your package).

#### External Dependencies

Instead of running `pip install package` you should use `uv add package`. Using `uv add` instead of `pip install` results in:

- `package` added as dependency to `pyproject.toml`
- `uv.lock` updated

You might find [uv's docs](https://docs.astral.sh/uv/concepts/projects/dependencies/) helpful if you need to specify specific (including platform-specific) source for the package.

#### pyproject.toml and uv.lock

At this point, you might wonder -- what is `uv.lock` and how is it different from `pyproject.toml`?

In short:

- `pyproject.toml` is a soft specification of minimal version requirements of top-level packages
- `uv.lock` is a strict and full specification of all packages installed in current venv.

As an example, if you `uv add` a package, say, SciPy, `pyproject.toml` will add `scipy>=1.15.2.`, i.e. a spec of minimally required version. However, SciPy itself depends on NumPy, so NumPy will be installed in the venv and reflected in `uv.lock`, but not in `pyproject.toml`.

In some sense, `uv.lock` is similar to what you'd get from `pip freeze`. Why do we need both `pyproject.toml` and `uv.lock`?

- `pyproject.toml` is useful to keep track of which packages you intentionally added. If you want to update, say, SciPy to 1.16, you can update `pyproject.toml` and run `uv lock --upgrade`. If SciPy depends on 10 other packages, you shouldn't be required to manually update dependencies of all those packages, which is why it's all handled for you by `uv.lock`.
- `uv.lock` is useful because it allows any other user to recreate your local environment exactly. You never need to manually modify this file, as long as you use `uv add`, `uv remove` to handle dependencies.

See [uv locking and syncing page](https://docs.astral.sh/uv/concepts/projects/sync/#exporting-the-lockfile) for more details.

!!! tip "uv.lock gives you confidence that your code will always work"

    Basically, once you have a working codebase, even if you stop working on it, you (or any other human) can come back to it at any point in the future, and he'll be able to run it without any issues. Your code will work even if numpy updated 10 times in the meantime or some other package stopped being maintained. Fun fact: lock files have been a standard in web dev for almost 15 years, you might be familiar with `package-lock.json` and `yarn.lock`.

#### Dependency Groups

By default, every package you `uv add` appears as:

```toml
[project]
dependencies = ["numpy==1.26.4"]
```

However, you can also have optional dependency groups:

```toml
[project.optional-dependencies]
dev = ["ruff>=0.9.6", "mypy>=1.15.0"]
web = ["flask>=3.1.0"]
```

which you can install by:

```bash
uv pip install -e ".[dev]" # will install main deps and dev deps
uv pip install -e ".[web]" # will install main + web deps
uv pip install -e ".[dev,web]" # will install main + dev + web deps
```

which is useful when, say, 2 people are working on the project and only one of them develops the web-facing code. There's no reason for the other person to have all web-related dependencies in his venv, so they can be separated into a dep group.

## Migrating from Traditional Tools

### From pip + venv

If you're currently using pip with venv, migration is straightforward:

1. Install uv: Follow the installation instructions above

1. For existing projects:

    ```bash
    # In your project directory
    uv pip freeze > requirements.txt  # Export current dependencies
    deactivate  # Exit current venv
    rm -rf venv/  # Remove old venv
    uv venv  # Create new venv
    source .venv/bin/activate
    uv pip install -r requirements.txt  # Install dependencies
    ```

1. Convert to modern structure:

    - Create `pyproject.toml` using `uv init`
    - Move dependencies from `requirements.txt` to `pyproject.toml`
    - Run `uv pip install -e .` to install in editable mode

### From conda

For conda users, the transition requires a few additional steps:

1. Export your conda environment:

    ```bash
    conda list --explicit > conda-packages.txt
    ```

1. Identify pure Python packages from conda-packages.txt

1. Create new project with uv:

    ```bash
    uv venv
    source .venv/bin/activate
    ```

1. Install required packages using `uv add`

1. For conda-specific packages (like MKL-optimized numpy), refer to package documentation for pip-compatible alternatives

## Summary

### Key Takeaways

- `uv` provides a faster, simpler alternative to traditional Python tooling
- The src/ layout with pyproject.toml offers a clean, modern project structure
- Lock files ensure reproducible environments across team members and time
- Dependency groups help manage optional features efficiently

### Related Resources

- [uv Documentation](https://docs.astral.sh/uv/)
- [Python Packaging User Guide](https://packaging.python.org/)
- [Batista Group Python Template](https://github.com/batistagroup/python-template)
