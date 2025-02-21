# GitHub Workflows: Automated CI/CD Pipeline Management

## Overview

GitHub Workflows are automated processes that help you build, test, package, release, and deploy projects directly from your GitHub repository. They enable continuous integration and continuous deployment (CI/CD) through declarative YAML files stored in your `.github/workflows` directory.

In the [intro to git](/git/intro/#3-branching-strategy), a recommended workflow was making all changes on separate branches, which are merged into master through a Pull Request that is reviewed by the team. Workflows allows to automate many of the check the team might want to perform. For example, you'd want to know: is the code properly [linted & formatted](/docs/python/lint.md)? Is it [type annotated](/docs/python/typing/intro.md)? Does it pass all [test cases](/docs/python/tests/intro.md)? Because even if you recommend people to use [pre-commit hooks](/docs/git/pre-commit.md), there's no guarantee that they're actually used. So you could have workflows to answer all of the questions above.

## Prerequisites

To work with GitHub Workflows, you'll need a GitHub repository and write access to configure workflows. You should also be familiar with basic Git operations and YAML syntax, as workflows are defined using YAML configuration files.

## Core Concepts

### Workflow Structure

A GitHub workflow consists of:

```yaml
name: Workflow Name           # The name that appears in GitHub Actions
on: [trigger_events]         # Events that trigger the workflow
jobs:                        # Groups of steps to execute
  job_id:                    # Unique identifier for the job
    runs-on: ubuntu-latest   # Runner environment
    steps:                   # Individual tasks to perform
      - name: Step Name      # Description of the step
        uses: action/repo    # Action to use
        with:                # Input parameters
          param: value
```

### Common Trigger Events

Workflows can be triggered by various events in your repository. The most common triggers include pushes to specific branches, pull request activities (opened, reopened, or synchronized), manual triggers through `workflow_dispatch`, and scheduled runs using cron syntax.

## Practical Examples

As an example, we'll go through workflows configured in our [python-template repo](https://github.com/batistagroup/python-template/tree/master).

### Code Quality

The [.github/workflows/quality.yml](https://github.com/batistagroup/python-template/blob/master/.github/workflows/quality.yml) workflow proceeds as:

1. Install the package with dev dependencies `uv pip install -e ".[dev]"`
1. Check linting `ruff check`
1. Check formatting \`ruff format --check\`\`
1. Check type annotations `mypy .` (this will follow the config in `pyproject.toml`, i.e. this will be a strict check if pyproject has `strict=True` flag)

The particular implementation above caches the venv so that `uv pip install -e ".[dev]"` doesn't have to redownload all dependencies each time the workflow is triggered. The quality workflow runs on any push and any pull request.

### Tests

The [.github/workflows/tests.yml](https://github.com/batistagroup/python-template/blob/master/.github/workflows/tests.yml) workflow mirrors the `quality.yml` but instead of running ruff and mypy, it runs `pytest -v`. As written, tests workflow is also triggered on every push and pull request. Properly maintained codebases may have such extensive test suits that their execution takes awhile, which is why the tests workflow is only triggered upon a PR (and not every individual commit). To make `tests.yml` run only on pull requests, simply modify

```yaml
# replace this
on: [push, pull_request] 
# with
on: [pull_request] 
```

### Release Drafter

The Release Drafter automates changelog generation by categorizing pull requests based on their labels. It consists of two key components:

1. **Configuration** ([.github/release-drafter.yml](https://github.com/batistagroup/python-template/blob/master/.github/release-drafter.yml)):

```yaml
name-template: v$RESOLVED_VERSION 📚      # Release name format
categories:                               # Organizes changes by category
  - title: ⚛️ Core Science               # Scientific updates
    labels:
      - quantum-mechanics
      - quantum-computing
      - quantum-dynamics
  - title: 💻 Programming & Tools         # Development updates
    labels:
      - git
      - python
  - title: 📖 Educational                # Learning materials
    labels:
      - tutorials
      - lectures
```

1. **Version Resolution** ([.github/workflows/release-drafter.yml](https://github.com/batistagroup/python-template/blob/master/.github/workflows/release-drafter.yml)):
    The workflow employs a semantic versioning strategy based on PR labels. Major version bumps are triggered by breaking changes, minor versions by new features in areas like quantum mechanics or tutorials, and patch versions by maintenance updates. If no specific version label is present, the system defaults to a patch version increment.

When a PR is opened or updated, the workflow analyzes commit messages for keywords, automatically assigns corresponding labels, uses these labels to categorize changes in the release draft, and updates the version number based on the labels' significance. For instance, a PR with commit message "feat(quantum): Add new QM simulation" would be labeled as "quantum-mechanics", appear under "Core Science" in the changelog, trigger a minor version bump, and be formatted as "- Add new QM simulation @author (#123)". To see existing labels for the knowledgebase repo, see [labels doc](/docs/labels.md).

### Update Major Minor Tags

The [.github/workflows/update-major-minor-tags.yml](https://github.com/batistagroup/python-template/blob/master/.github/workflows/update-major-minor-tags.yml) workflow updates major/minor release tags on a tag push. e.g. update v1 and v1.2 tag when released v1.2.3.

Tags accompany releases and allow you to reference particular commits (states of the repo) when that version was released. e.g. you can run `git checkout v1` (which will point to the latest version of v1 release) or `git checkout v1.2.3`.

### Version Bump

The [.github/workflows/version-bump.yml](https://github.com/batistagroup/python-template/blob/master/.github/workflows/version-bump.yml) updates version line in `pyproject.toml` when a new release is published.

### Publish Release

The [.github/workflows/publish-release.yml](https://github.com/batistagroup/python-template/blob/master/.github/workflows/publish-release.yml) builds the package when release is published and attaches `.tar.gz` to the assets of the release.

## Best Practices

### Security

Protecting your workflow's security involves using secrets for sensitive data through `${{ secrets.SECRET_NAME }}` syntax, setting minimum required permissions for each job, and using specific versions for actions rather than floating references.

### Performance

To optimize workflow performance, use Ubuntu runners for faster startup times, implement dependency caching where possible, and structure your workflows to run independent jobs in parallel.

### Maintenance

Maintain a clean and organized workflow structure by keeping all workflow files in the `.github/workflows` directory. Use descriptive names for your workflows and jobs, and include clear comments to document complex steps or configurations.

## Common Pitfalls

When working with GitHub Workflows, watch out for common issues such as missing required permissions, using floating references like `@master` instead of specific versions, inadequate error handling, and overly complex workflow structures that are difficult to maintain.

## Related Resources

For more detailed information about GitHub Actions and workflows, consult the [GitHub Actions Documentation](https://docs.github.com/en/actions), browse available actions in the [GitHub Marketplace](https://github.com/marketplace?type=actions), or refer to the [Workflow Syntax Reference](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions).
