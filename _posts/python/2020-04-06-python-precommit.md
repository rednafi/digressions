---
title: Running Python Linters with Pre-commit Hooks
toc: true
comments: true
layout: post
description: A sane approach to Python linting with pre-commit hooks
categories: [Python]
---

[Pre-commit hooks](https://pre-commit.com/#introduction) can be a neat way to run automated ad-hoc *tasks* before submitting every git commit. These tasks may include linting, trimming trailing whitespace, running code formatter before code review etc. Let's see how multiple Python linters and formatters can be applied automatically before each commit to impose strict conformity on your codebase.

To keep my sanity, I only use three linters in all of my python projects:

* **[Black](https://github.com/psf/black)**: `Black` is the uncompromising Python code formatter. It uses consistent rules to format your python code and makes sure that they look the same regardless of the project you're reading.

* **[Isort](https://github.com/timothycrosley/isort)**: `Isort` is a Python utility to sort imports alphabetically, and automatically separated into sections and by type.

    Isort parses specified files for global level import lines and puts them all at the top of the file grouped together by the type of import:

    - Future
    - Python Standard Library
    - Third Party
    - Current Python Project
    - Explicitly Local (. before import, as in: `from . import x`)
    - Custom Separate Sections (Defined by `forced_separate` list in the configuration file)
    - Custom Sections (Defined by `sections` list in configuration file)

    Inside each section, the imports are sorted alphabetically. `Isort` automatically removes duplicate python imports, and wraps long from imports to the specified line length (defaults to 79).

* **[Flake8](https://github.com/PyCQA/flake8)**: `Flake8` is a wrapper around `PyFlakes`, `pycodestyle`, Ned Batchelder's [McCabe script](https://github.com/PyCQA/mccabe). The combination of these three linters makes sure that your code is compliant with [PEP 8](https://www.python.org/dev/peps/pep-0008/) and avoids obvious code smells.

## Installing Pre-commit

* Install using `pip`:

    ```python
    pip install pre-commit
    ```

* Install via `curl`:

    ```python
    curl https://pre-commit.com/install-local.py | python -
    ```

## Defining the Pre-commit Config File

Pre-commit configuration is a `.pre-commit-config.yaml` file where you define your tasks that you want to run before every commit. Once you have defined your configuration, it will run automatically every time you say `git commit -m "Commit message"`. The following example shows how black formatter can be added to the config file:

```yaml
# .pre-commit-config.yaml

repos:
-   repo: https://github.com/pre-commit/pre-commit-hooks
    rev: v2.3.0
    hooks:
    -   id: check-yaml
    -   id: end-of-file-fixer
    -   id: trailing-whitespace
-   repo: https://github.com/psf/black
    rev: 19.3b0
    hooks:
    -   id: black
```

## Installing the Git Hook scripts

Run

```bash
pre-commit install
```

This will set up the git hook scripts and should show the following output in your terminal:

```
pre-commit installed at .git/hooks/pre-commit
```

Now you will be able to implicitly or explicitly run the hooks before each commit.

## Running the Hooks Against All the Files
By default, the hooks will run every time you say:

```bash
git commit -m "Commit message"
```

However, if you wish to run the hooks manually on every file, you can do that via:

```bash
pre-commit run --all-files
```

## Running the Linters as Pre-commit Hooks

To run the above mentioned linters as pre-commit hooks, you need to customize the configuation in the `.pre-commit-config.yaml` file. However, there are a few minor issues that needs to be taken care of.

* The default line length of `black` formatter is 88 (you should embrace that) but both `flake8` and `isort` cap the line at 79 characters. This conflict causes the hook to fail.

* Black and isort format the imports differently

* `Flake8` can be overly strict at times. You'll want to ignore basic errors like unused imports, spacing issues etc. However since your IDE/ editor also points out these issues, you should solve these manually. So, we need to configure flake8 to ignore some minor errors.

The following one is an example of how you can use `isort`, `black`, `flake8` linters without raising any conflicts.

```yaml
# .pre-commit-config.yaml

# isort
- repo: https://github.com/pre-commit/mirrors-isort
  rev: v4.3.21
  hooks:
    - id: isort
      args: # arguments to configure isort
        # making isort line length compatible with black
        - --line_length 88
        - --use_parentheses True
        - --include_trailing_comma True
        - --multi_line_output 3

# black
- repo: https://github.com/ambv/black
  rev: stable
  hooks:
    - id: black
      args: # arguments to configure black
        - --line-length=88
        - --include='\.pyi?$'

        # these folders wont be formatted by black
        - --exclude="""\.git |
          \.__pycache__|
          \.hg|
          \.mypy_cache|
          \.tox|
          \.venv|
          _build|
          buck-out|
          build|
          dist"""

      language_version: python3.6

# flake8
- repo: https://github.com/pre-commit/pre-commit-hooks
  rev: v2.3.0
  hooks:
    - id: flake8
      args: # arguments to configure flake8
        # making isort line length compatible with black
        - "--max-line-length=88"
        - "--max-complexity=18"
        - "--select=B,C,E,F,W,T4,B9"

        # these are errors that will be ignored by flake8
        # check out their meaning here
        # https://flake8.pycqa.org/en/latest/user/error-codes.html
        - "--ignore=E203,E266,E501,W503,F403,F401,E402"
```

You can add the above lines to your configuration and run

```bash
pre-commit run --all-files
```

This should apply your pre-commit hooks to your code base harmoniously.
