# Python CI/CD Pipeline with GitHub Actions

## Overview

This project demonstrates a modern Continuous Integration (CI) pipeline for Python applications using:

* Python 3.13
* GitHub Actions
* uv
* Ruff
* Pytest
* Gitleaks

The goal is to automatically validate code quality, security, and functionality whenever code is pushed to the repository or a pull request is created.

---

# Table of Contents

1. What is CI/CD?
2. Why GitHub Actions Instead of Jenkins?
3. GitHub Actions Fundamentals
4. GitHub Hosted vs Self Hosted Runners
5. Project Structure
6. CI Workflow
7. Workflow Steps
8. Needs
9. if Condition (github.event_name)
10. Gitleaks Security Scanning
11. PR Validation Pipeline
12. Push Build Pipeline
13. Branch Protection Rules
14. GitHub Secrets and Variables
15. Timeout Configuration
16. Running Locally
17. Migrating to uv
18. Benefits
19. Technologies Used

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

```text
Developer writes code
        ↓
Developer manually runs tests
        ↓
Developer pushes code
        ↓
Issues may reach production
```

### CI Workflow

```text
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
```

---

## Continuous Delivery / Deployment (CD)

CD extends CI by automatically preparing or deploying applications after successful validation.

Examples:

* Deploy FastAPI applications
* Deploy Docker containers
* Publish Python packages
* Release application artifacts

This repository focuses primarily on the CI portion.

---

# Why GitHub Actions Instead of Jenkins?

GitHub Actions is tightly integrated with GitHub, requires no server maintenance, and is easy to configure using YAML workflow files stored directly in the repository.

Jenkins offers greater customization and is often preferred in large enterprises with existing Jenkins infrastructure or on-premises requirements, but it requires:

* Server installation
* Plugin management
* Upgrades
* Backup management
* Infrastructure maintenance

### Comparison

| GitHub Actions                | Jenkins                                             |
| ----------------------------- | --------------------------------------------------- |
| Managed by GitHub             | Self-managed                                        |
| No infrastructure maintenance | Requires server management                          |
| Native GitHub integration     | Plugin-based integration                            |
| Easy setup                    | More complex setup                                  |
| Ideal for GitHub projects     | Ideal for highly customized enterprise environments |

---

# GitHub Actions Fundamentals

## What is a Runner?

A runner is the machine that executes workflow jobs.

## What is a Job?

A job is a collection of steps executed on a runner.

## What is a Step?

A step is an individual task executed within a job.

---

# Job Dependencies using needs

The `needs` keyword is used to create dependencies between jobs.

By default, GitHub Actions runs jobs in parallel whenever possible.

Example:

```yaml
jobs:
  build:
    runs-on: ubuntu-latest

  test:
    runs-on: ubuntu-latest
```

In this case:

```text
build ──────┐
            ├── Run in Parallel
test  ──────┘
```

Both jobs start at the same time.

---

## Why use needs?

Sometimes a job should only run after another job completes successfully.

Example:

```yaml
jobs:
  gitleaks-scan:
    runs-on: ubuntu-latest

  test:
    needs: gitleaks-scan
    runs-on: ubuntu-latest
```

Flow:

```text
Gitleaks Scan
       ↓
      Test
```

The test job starts only after the Gitleaks scan succeeds.

If Gitleaks fails:

```text
Gitleaks Scan ❌
       ↓
Test (Skipped)
```

This prevents unnecessary execution of downstream jobs.

---

## Example from this Project

The pipeline uses Gitleaks as a prerequisite job.

```yaml
gitleaks-scan:
  runs-on: ubuntu-latest

build:
  needs: gitleaks-scan

test:
  needs: gitleaks-scan
```

### Pull Request Flow

```text
pull_request
      ↓
gitleaks-scan
      ↓
test
```

### Push to Main Flow

```text
push
 ↓
gitleaks-scan
 ↓
build
```

This ensures:

* Secrets are scanned first
* Build/Test jobs run only when security checks pass
* CI resources are not wasted on failed scans

---

## Multiple Dependencies

A job can depend on multiple jobs.

Example:

```yaml
deploy:
  needs:
    - build
    - test
```

Flow:

```text
build ───┐
         ├── deploy
test  ───┘
```

The deploy job starts only when both build and test complete successfully.

---

## Benefits of needs

* Controls execution order
* Prevents unnecessary job execution
* Improves pipeline reliability
* Makes workflows easier to understand
* Enables secure CI/CD pipelines

---

## Interview Question

### What is `needs` in GitHub Actions?

`needs` is used to define job dependencies. A job configured with `needs` will wait until the specified job or jobs complete successfully before starting. If the dependency fails, the dependent job is skipped. This helps control workflow execution order and prevents unnecessary builds, tests, or deployments.


# Conditional Job Execution using if

GitHub Actions provides the `if` condition to control when a job should run.

This project uses:

```yaml
build:
  if: github.event_name == 'push'
```

and

```yaml
test:
  if: github.event_name == 'pull_request'
```

This helps separate the pipeline based on the GitHub event.

---

## Why use if?

Without conditions:

```text
Pull Request
      ↓
Build
Test
```

Both jobs would run.

With conditions:

```text
Pull Request
      ↓
Only Test Job Runs
```

and

```text
Push to Main
      ↓
Only Build Job Runs
```

This reduces unnecessary executions and speeds up the pipeline.

---

## Pull Request Flow

When a Pull Request is raised against the main branch:

```text
pull_request
      ↓
gitleaks-scan
      ↓
test
  ├── Ruff
  └── Pytest
```

Purpose:

* Validate code quality
* Detect secrets
* Execute tests
* Prevent broken code from being merged

No build is performed.

---

## Push to Main Flow

When code is merged into the main branch:

```text
push to main
      ↓
gitleaks-scan
      ↓
build
```

Purpose:

* Security validation
* Package creation
* Artifact generation
* Deployment preparation

Testing has already been completed during the Pull Request stage.

---

## Benefits

* Faster feedback
* Reduced CI execution time
* Clear separation of responsibilities
* Avoids unnecessary builds during PR validation
* Follows common enterprise CI/CD practices

---

## Interview Question

### Why did you use github.event_name?

I used `github.event_name` to separate validation and build activities. Pull Requests trigger validation jobs such as secret scanning, linting, and testing, while pushes to the main branch trigger build-related activities. This keeps the pipeline efficient and prevents unnecessary builds during code review.


# GitHub Hosted vs Self Hosted Runners

## GitHub Hosted Runners

These are machines provided and managed by GitHub.

Benefits:

* No maintenance
* Auto-updated
* Easy setup
* Suitable for most projects

## Self Hosted Runners

These are machines owned and managed by the organization.

Benefits:

* Access to internal networks
* Custom software
* Specialized hardware
* Persistent build environments

Trade-off:

* Maintenance responsibility
* Security management
* Infrastructure cost

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

* Any push to main triggers the pipeline
* Any pull request targeting main triggers the pipeline

---

# Workflow Steps

## 1. Checkout Repository

```yaml
- uses: actions/checkout@v5
```

Downloads repository contents into the GitHub runner.

---

## 2. Setup Python

```yaml
- uses: actions/setup-python@v6
```

Installs the required Python version.

---

## 3. Setup uv

```yaml
- uses: astral-sh/setup-uv@v6
```

Installs uv.

Benefits:

* Faster dependency resolution
* Faster installations
* Simplified environment management

---

## 4. Install Dependencies

```bash
uv sync
```

Installs dependencies from:

* pyproject.toml
* uv.lock

---

## 5. Run Ruff

```bash
uv run ruff check .
```

Checks:

* Syntax errors
* Unused imports
* Undefined variables
* Style violations

---

## 6. Run Format Check

```bash
uv run ruff format --check .
```

Validates formatting consistency.

---

## 7. Run Pytest

```bash
uv run pytest -v
```

Executes automated tests.

---

# Gitleaks Security Scanning

Gitleaks is a SAST (Static Application Security Testing) tool used to detect and prevent hardcoded secrets such as:

* Passwords
* API Keys
* Tokens
* Cloud Credentials

Example:

```bash
gitleaks detect --source . --exit-code 1
```

If secrets are detected, the workflow fails immediately.

---

# PR Validation Pipeline

The purpose of the PR Validation Pipeline is to verify code quality before code is merged into the main branch.

```text
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
Ruff Check
        ↓
Pytest Execution
        ↓
Pass / Fail
```

Only validated code can be merged.

---

# Push Build Pipeline

After the PR is approved and merged:

```text
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
```

This separates validation from build activities.

---

# Branch Protection Rules

Location:

```text
Settings
    ↓
Branches
    ↓
Add Classic Branch Protection Rule
```

Recommended settings:

* Require a pull request before merging
* Require status checks to pass before merging
* Require review approval
* Prevent force pushes

---

# GitHub Secrets and Variables

## Repository Secrets

Location:

```text
Settings
    ↓
Secrets and Variables
    ↓
Actions
    ↓
Secrets
```

Usage:

```yaml
${{ secrets.SECRET_NAME }}
```

---

## Repository Variables

Location:

```text
Settings
    ↓
Secrets and Variables
    ↓
Actions
    ↓
Variables
```

Usage:

```yaml
${{ vars.VARIABLE_NAME }}
```

---

# Timeout Configuration

Timeout prevents jobs from running indefinitely.

Example:

```yaml
timeout-minutes: 15
```

If a job exceeds the specified duration, GitHub automatically terminates it.

---

# Running Locally

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

# Migrating to uv

## Before

```bash
pip install -r requirements.txt
flake8 .
pytest
```

## After

```bash
uv sync
uv run ruff check .
uv run pytest
```

Benefits:

* Faster installs
* Lock file support
* Better dependency management
* Modern Python workflow

---

# Benefits

* Automated quality checks
* Security scanning
* Faster feedback
* Consistent coding standards
* Early bug detection
* Improved reliability
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
| Gitleaks       | Secret Scanning       |
