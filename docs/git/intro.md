# Git and GitHub for Computational Research

Git and GitHub are essential tools for both research and software development. Whether you're working on a research project or building a Python package, version control helps you track changes, collaborate with others, and maintain high-quality code. This guide shows you how to use these tools effectively in your daily work.

## Overview

Version control is a crucial skill for anyone writing code. You'll learn how to track your code changes, collaborate with others, and manage your projects effectively. We'll cover both research workflows and Python package development, showing you the best practices for each.

To follow along, you'll need Git installed (version 2.0 or higher), a GitHub account, and your favorite code editor. Basic command line knowledge will help, but we'll explain all commands as we go.

## 1. Why Version Control?

Version control helps you in the following cases:

### For Research Code

Your experiments generate lots of code variations and data. Git helps you track which parameters produced which results, maintain a complete history of your analysis changes, and document your computational setup. When collaborating with others, you can easily merge different analyses and review each other's work.

When exploring new ideas, Git lets you try things out without fear of breaking what works. Every failed approach becomes valuable documentation rather than lost work. Years later, you can understand why certain decisions were made and build upon past knowledge.

For example, imagine you have this working function for data processing:

```python
def process_data(data):
    """Process input data using moving average."""
    window_size = 5
    result = []
    for i in range(len(data) - window_size + 1):
        window = data[i : i + window_size]
        result.append(sum(window) / window_size)
    return result
```

Without version control, you might try something like this:

```python
def process_data(data):
    """Process input data using moving average."""
    window_size = 5
    result = []
    for i in range(len(data) - window_size + 1):
        window = data[i : i + window_size]
        result.append(sum(window) / window_size)
    return result


# New version with weighted average
# TODO: Delete old version if this works better
def process_data_weighted(data):
    """Process input data using weighted moving average."""
    window_size = 5
    weights = [0.1, 0.2, 0.4, 0.2, 0.1]
    result = []
    for i in range(len(data) - window_size + 1):
        window = data[i : i + window_size]
        result.append(sum(w * x for w, x in zip(weights, window)))
    return result
```

With Git, you can explore this change more cleanly:

```bash
git checkout -b feature/weighted-average
# Edit process_data() directly in your code
git commit -m "Switch to weighted moving average for better noise handling"
```

If the new approach works better, merge it to main. If not, just switch back:

```bash
git checkout main  # Your original version is safe here
```

The failed experiment isn't wasted—the branch preserves the complete context of what you tried and why.

### For Python Packages

When building Python packages, Git helps you manage releases and maintain backward compatibility. You can develop new features without disrupting users of the stable version, track bugs across releases, and coordinate with contributors worldwide.

## 2. Git Fundamentals

### 2.1 Core Concepts

Git tracks your code's evolution through a series of snapshots. Each snapshot, called a **commit**, records the complete state of your files at a specific point in time. You can always return to any previous state.

**Branches** let you maintain different versions of your code simultaneously. The main branch typically contains your stable code, while feature branches let you experiment without affecting that stability. You can create as many branches as you need and merge them back together later.

**Tags** are permanent markers you attach to specific commits. Unlike branches, tags don't move as you add new commits. They're perfect for marking important points in your project's history, like release versions or significant milestones.

The **`.gitignore`** file tells Git which files it shouldn't track. This is crucial for keeping your repository clean and efficient—you don't want to track temporary files, build outputs, or large data files (`.csv,.pkl,.ckpt`) that can be regenerated.

These concepts map naturally to both research and software development:

| Git Concept | Research Use              | Package Development Use        |
| ----------- | ------------------------- | ------------------------------ |
| Commits     | Snapshot experiment state | Record feature implementation  |
| Branches    | Test different approaches | Develop features independently |
| Tags        | Mark dataset versions     | Release package versions       |
| .gitignore  | Exclude raw data          | Ignore build artifacts         |

### 2.2 Project Structure

A well-structured Python project, whether for research or distribution, typically looks like this:

```
project/
├── src/                 # Source code
│   └── yourpackage/
│       ├── __init__.py
│       └── core.py
├── tests/              # Test suite
├── docs/               # Documentation
├── pyproject.toml     # Dependencies
├── uv.lock           # Dependencies
└── README.md         # Project overview
```

For research projects, you might add:

```
research-project/
├── experiments/        # Experiment code
├── data/              # Data management
└── results/           # Outputs and figures
```

## 3. Branching Strategy

### 3.1 Core Branches

Your main branch represents the stable, published state of your work. For research, this means verified results and polished analysis. For Python packages, it's your latest stable release.

Create feature branches for new work. Use descriptive names like `experiment/neural-net` for research or `feature/async-processing` for package development. This keeps your changes isolated and makes it easy to track different lines of work.

### 3.2 Branch Lifecycle

When you start new work, branch off from main. Work on your changes, commit regularly, and push to GitHub to back up your work. Once you're satisfied with the results, create a pull request for team review. After approval, merge your changes back to main.

If an experiment or feature doesn't work out, don't delete the branch immediately. Instead, tag it (like `archive/failed-approach-1`) to preserve the knowledge of what didn't work and why.

## 4. Essential Git Commands

### Common Workflows

Starting a new project:

```bash
git init                                    # Create new repository
git remote add origin <url>                 # Connect to GitHub
git checkout -b feature/initial-setup       # Start work on first feature
```

Tracking your changes:

```bash
git add .                                   # Stage all changes
git commit -m "Add data processing module"  # Record changes
git push origin feature/data-processing     # Share with team
```

Collaborating with others:

There are two main ways to get changes from your team. Using `pull` is simpler but less precise, while `fetch + rebase` gives you more control:

```bash
# Option 1: Simple but less control
git pull                                    # Fetch + merge in one step

# Option 2: More explicit and clean history
git fetch                                   # Download changes without applying
git rebase origin/main                      # Replay your work on top of team's changes
```

The key difference is how your changes combine with your team's work:

- `git pull` downloads and merges changes immediately, which can create merge commits
- `git fetch` only downloads changes, letting you decide how to integrate them
- `git rebase` replays your changes on top of the team's work, keeping history linear

Choose `fetch + rebase` when you want a clean, linear history. Use `pull` for quick updates when the extra merge commits don't matter.

### Advanced Techniques

#### Finding Bugs with git bisect

`git bisect` is a powerful debugging tool that helps you find which commit introduced a bug using binary search. You start by telling Git two points in your project's history: a "good" commit where everything worked and a "bad" commit where the bug exists. Git then walks you through the commits between these points, efficiently narrowing down where the problem started.

The process is interactive. For each commit Git checks out, you test your code to see if the bug exists. This could mean running your test suite, checking if your API returns correct results, or verifying your ML model's accuracy. You then tell Git whether this commit is "good" or "bad", and it uses this information to choose the next commit to test. Through this binary search process, Git can find the exact commit that introduced the problem in just a few steps, even if there are hundreds of commits to search through.

For example, imagine your machine learning model's accuracy dropped from 90% to 60%. You know it worked well last month but can't pinpoint what change caused the regression. With bisect, you'd mark last month's commit as "good" and the current state as "bad". Git would then help you systematically test commits in between, perhaps revealing that a seemingly innocent change to your data preprocessing pipeline three weeks ago actually caused the accuracy drop.

```bash
git bisect start
git bisect bad HEAD
git bisect good v1.0
# Git will now guide you through the binary search
```

When bisect finishes, it shows you the exact commit that introduced the problem, complete with the commit message, author, and date. This information is invaluable for understanding not just what broke your code, but why and who might have context about the change.

<!-- #### Managing Large Files with Git LFS

Git LFS (Large File Storage) solves the problem of storing large files (like datasets, models, or images) in your repository. Instead of storing the actual files in Git, LFS replaces them with small pointer files and stores the large files on a separate server.

Here's how to use it:

```bash
# Install Git LFS
git lfs install

# Tell LFS which files to track
git lfs track "*.csv"          # Track all CSV files
git lfs track "*.pkl"          # Track all pickle files
git lfs track "models/**/*.h5" # Track all HDF5 files in models directory

# The tracking rules are stored in .gitattributes
git add .gitattributes
git commit -m "Configure Git LFS tracking"

# Now use Git normally - LFS handles the large files automatically
git add my_large_dataset.csv
git commit -m "Add training dataset"
```

LFS is particularly useful for:
- Large datasets that need versioning
- Trained model checkpoints
- High-resolution images or videos
- Any file over 50MB (GitHub's file size limit)

When someone clones your repository, they can choose to download the large files or not:
```bash
git clone --no-checkout your-repo   # Clone without downloading LFS files
git lfs pull                        # Download LFS files when needed
``` -->

## 5. Working with GitHub

### Code Review

Code review is essential for both research and package development. Create a pull request when you're ready to merge your changes. Describe what you've done and what problem it solves. For research code, include key results or visualizations. For package features, explain the API changes and include usage examples.

### Project Management

Use GitHub Issues to track work. For research, create issues for each experiment or analysis task. For package development, track feature requests, bug reports, and improvements. Link related pull requests to automatically close issues when work is merged.

### Continuous Integration

Set up GitHub Actions to automate checks:

```yaml
name: CI
on: [push, pull_request]
jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v2
      - name: Run checks
        run: |
          uv pip install -e ".[test]"
          pytest
          ruff check
          ruff format --check
          mypy .
```

## 6. Security and Data Management

### Sensitive Data

Never commit sensitive data or credentials. Use environment variables for secrets and add sensitive file patterns to .gitignore. For research data that can't be public, use Git-crypt or similar tools to encrypt specific files.

### Access Control

Configure branch protection on main to require reviews and passing tests before merging. This prevents accidental changes to your stable code. For private repositories, carefully manage team access levels and regularly audit who has access.

## 7. Best Practices

### For Research Projects

Write clear commit messages explaining what changed and why. Include enough detail to understand the experiment parameters and key results. Tag important states of your analysis for easy reference later.

Keep your analysis reproducible by managing dependencies carefully. Use virtual environments and record exact package versions. Document setup steps clearly in your README.

### For Python Packages

Follow semantic versioning for releases. Major version changes (1.0 to 2.0) indicate breaking changes, minor versions (1.1 to 1.2) add features, and patches (1.1.1 to 1.1.2) fix bugs.

Write comprehensive tests and maintain good documentation. Set up automated builds to catch issues early. Keep a detailed changelog to help users understand what's changed between versions.

## 8. Common Mistakes to Avoid

Committing large data files directly to Git will bloat your repository. Instead, use Git LFS or external data storage. Store only processed, minimal datasets in Git when necessary.

Don't make huge, sweeping changes in a single commit. Break work into logical, reviewable chunks. Each commit should be focused and self-contained.

Avoid committing temporary files, build artifacts, or environment-specific settings. Keep your repository clean and focused on source code and essential documentation.

## 9. Additional Resources

The [Turing Way](https://the-turing-way.netlify.app/) provides excellent guidance for reproducible research. For Python packaging, the [Python Packaging User Guide](https://packaging.python.org) is the authoritative resource.

Learn more about Git with [Pro Git](https://git-scm.com/book/en/v2) and [GitHub's documentation](https://docs.github.com). For research-specific workflows, check out [Git for Scientists](https://milesmcbain.github.io/git_4_sci/).
