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

### Why we rae using github pipeline instead of jenskins
GitHub Actions is tightly integrated with GitHub, requires no server maintenance, and is easy to set up using workflow YAML files stored in the repository. For projects already hosted on GitHub, it provides a simple way to implement CI/CD. Jenkins offers greater customization and is often preferred in large enterprises with existing Jenkins infrastructure or on-premises requirements, but it requires managing servers, plugins, and upgrades.

In jenkins it requires 2 Virtual machines one for jekins master and another for slave to run jobs
But in github pipeline there two runners:
1. free/shared runners
2. self hosted runners

free/ shared runners These are machines provided and managed by GitHub
self hosted runners These are machines that you own and manage.

### What is the difference between GitHub-hosted and self-hosted runners?

GitHub-hosted runners are temporary virtual machines managed by GitHub. They are easy to use and require no maintenance. Self-hosted runners are machines managed by the organization and registered with GitHub Actions. Self-hosted runners are useful when workflows need access to internal networks, specialized software, custom hardware, or persistent build environments. The trade-off is that the organization is responsible for maintaining and securing the runners.

action github market place:- https://github.com/marketplace?type=actions
here we can search for action like checkout or setup-python etc...


## Gitleaks:
Gitleaks is a SAST tool for detecting and preventing hardcoded secrets like passwords, API keys, and tokens in git repos. Gitleaks is an easy-to-use, all-in-one solution for detecting secrets, past or present, in your code. Enable Gitleaks-Action in your GitHub workflows to be alerted when secrets are leaked as soon as they happen.

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

Downloads the repository contents into the GitHub Actions runner(clonning: git clone the repository).

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

## What is a Runner?

A runner is the machine that executes workflow jobs.

## What is a Job?

A job is a collection of steps executed on a runner.

## What is a Step?

A step is an individual task inside a job.

### Creating rules for branch

 path:- Settings --> Branches --> Add classic branch protection rule
 1. we have to specify branch name.
 2. Require a pull request before merging.
 3. Require status checks to pass before merging. (all pipeline should pass).

 ### Git-hub secrets

path for Secrets:- Settings --> Secrets and variables --> Secrets --> New repository secret
Name : Secrets
we can access this secrets.name_of_the_secrets

path for Variables:- Settings --> Secrets and variables --> Variables --> New repository Variables
Name: Value
we can access this varibales.name_of_the_variable

### timeout

Adding timeout prevents a job from running forever if something gets stuck.


Added if: github.event_name condition it will helps to seprate pipeline will run on push and pull_request.

### Flow on Pull Request
pull_request
     ↓
gitleaks-scan
     ↓
test
  ├─ flake8
  └─ pytest


### Flow on Push to Main
push to main
      ↓
gitleaks-scan
      ↓
build

### PR Validation Pipeline

The purpose of the PR validation pipeline is to verify code quality before code is merged into main.

Flow:

Developer creates feature branch
        ↓
Code changes
        ↓
Pull Request → main
        ↓
GitHub Actions Triggered
        ↓
Gitleaks Scan
        ↓
Flake8 Lint Check
        ↓
Pytest Execution
        ↓
Pass/Fail


### Push Build Pipeline

After the PR is approved and merged:

Merge PR
      ↓
Push Event on main
      ↓
GitHub Actions Triggered
      ↓
Gitleaks Scan
      ↓
Install Dependencies
      ↓
Build Package
      ↓
Upload Artifact

I separated the pipeline into two stages. The Pull Request pipeline performs validation activities such as Gitleaks secret scanning, Flake8 linting, and Pytest execution. This ensures that only quality code can be merged into the main branch. Once the Pull Request is approved and merged, a push event on the main branch triggers the build pipeline, which performs security scanning again, installs dependencies, builds the package using Python build tools, and prepares artifacts for deployment. This separation reduces unnecessary builds and provides faster feedback to developers.

### Demo Repository Structure
sample-python-project/
│
├── app/
│   ├── calculator.py
│   └── __init__.py
│
├── tests/
│   └── test_calculator.py
│
├── requirements.txt
├── pyproject.toml
│
└── .github/
    └── workflows/
        ├── pr-validation.yml
        └── build.yml
