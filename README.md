# Python CI/CD Pipeline with GitHub Actions

## Overview

This project demonstrates a modern Continuous Integration (CI) pipeline for Python applications using:

* Python 3.13
* GitHub Actions
* uv
* Ruff
* Pytest

The goal is to automatically validate code quality and functionality whenever code is pushed to the repository or a pull request is created.

---

# What is CI/CD?

## Continuous Integration (CI)

Continuous Integration is the practice of automatically validating code whenever developers make changes.

Instead of manually checking:

* Code quality
* Formatting
* Unit tests

a CI pipeline performs these checks automatically.

### Traditional Workflow

Developer writes code
↓
Developer manually runs tests
↓
Developer pushes code
↓
Issues may reach production

### CI Workflow

Developer writes code
↓
Pushes code to GitHub
↓
GitHub Actions automatically runs
↓
Linting executes
↓
Tests execute
↓
Pass ✅ or Fail ❌

This helps catch issues early.

---

## Continuous Delivery / Deployment (CD)

CD extends CI by automatically preparing or deploying applications after successful validation.

Examples:

* Deploy FastAPI application
* Deploy Docker container
* Publish Python package
* Release application artifacts

This repository focuses on the CI portion.

---

# Project Structure

```text
.
├── .github
│   └── workflows
│       └── ci.yml
├── src
│   └── application code
├── tests
│   └── test files
├── pyproject.toml
├── uv.lock
└── README.md
```

---

# CI Workflow

The workflow file is located at:

```text
.github/workflows/ci.yml
```

The workflow executes automatically when:

```yaml
on:
  push:
    branches:
      - main

  pull_request:
    branches:
      - main
```

This means:

* Any push to main triggers the pipeline.
* Any pull request targeting main triggers the pipeline.

---

# Workflow Steps

## 1. Checkout Repository

```yaml
- uses: actions/checkout@v5
```

Downloads the repository contents into the GitHub Actions runner.

Without this step the workflow cannot access project files.

---

## 2. Setup Python

```yaml
- uses: actions/setup-python@v6
```

Installs the required Python version.

Example:

```yaml
python-version: "3.13"
```

---

## 3. Install uv

```yaml
- uses: astral-sh/setup-uv@v6
```

Installs uv.

uv is a modern replacement for:

* pip
* virtualenv
* pip-tools

Benefits:

* Faster dependency resolution
* Faster installations
* Simplified environment management

---

## 4. Install Dependencies

```yaml
uv sync --all-extras --dev
```

Installs:

* Application dependencies
* Development dependencies
* Testing dependencies

from pyproject.toml.

---

## 5. Run Ruff

```yaml
uv run ruff check .
```

Ruff performs static code analysis.

Checks for:

* Syntax errors
* Unused imports
* Undefined variables
* Style violations

Example:

```python
import os

print("Hello")
```

Ruff reports:

```text
os imported but unused
```

---

## 6. Run Formatting Validation

```yaml
uv run ruff format --check .
```

Ensures code formatting follows project standards.

The workflow fails if formatting is incorrect.

---

## 7. Run Pytest

```yaml
uv run pytest -v
```

Executes all test cases.

Example:

```python
def test_add():
    assert 2 + 2 == 4
```

Expected result:

```text
PASSED
```

---

# CI Pipeline Flow

```text
Push Code
    ↓
GitHub Actions Triggered
    ↓
Checkout Repository
    ↓
Setup Python
    ↓
Install uv
    ↓
Install Dependencies
    ↓
Run Ruff
    ↓
Run Format Check
    ↓
Run Pytest
    ↓
Pass ✅ / Fail ❌
```

---

# Running Locally

## Clone Repository

```bash
git clone <repository-url>
cd <repository-name>
```

## Install uv

```bash
pip install uv
```

## Install Dependencies

```bash
uv sync
```

## Run Ruff

```bash
uv run ruff check .
```

## Run Format Check

```bash
uv run ruff format --check .
```

## Run Tests

```bash
uv run pytest -v
```

---

# Benefits of this Pipeline

* Automated quality checks
* Faster feedback
* Consistent coding standards
* Early bug detection
* Improved code reliability
* Industry-standard CI workflow

---

# Technologies Used

| Technology     | Purpose               |
| -------------- | --------------------- |
| GitHub Actions | CI Automation         |
| Python         | Programming Language  |
| uv             | Dependency Management |
| Ruff           | Linting & Formatting  |
| Pytest         | Unit Testing          |

---
