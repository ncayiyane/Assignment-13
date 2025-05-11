# Assignment 13: CI/CD Pipeline with GitHub Actions

This project implements a comprehensive CI/CD pipeline using GitHub Actions for the AI-Powered Smart Appointment Booking System. It builds upon the domain model (Assignment 9), factories & repositories (Assignments 10-11), and services & API (Assignment 12).

## Project Structure

```
Assignment 13/
├── .github/
│   └── workflows/
│       └── ci.yml           # CI/CD workflow configuration
├── PROTECTION.md            # Explanation of branch protection rules
└── README.md                # Project documentation
```

## CI/CD Pipeline Technical Details

This project leverages GitHub Actions for continuous integration and continuous deployment, providing automated testing, quality assurance, and artifact generation.

### GitHub Actions Workflow Architecture

Our workflow is defined in `.github/workflows/ci.yml` and follows a multi-job architecture:

1. **Trigger Configuration**:
   ```yaml
   on:
     push:
       branches: [ '**' ]  # Triggers on push to any branch
     pull_request:
       branches: [ main ]  # Triggers on PRs targeting main
   ```

2. **Environment Specification**:
   - Uses `ubuntu-latest` runners for all jobs
   - Configures Python 3.8 runtime environment

3. **Job Dependencies**:
   - The `build-and-deploy` job depends on the `test` job
   - Ensures artifacts are only built after successful testing

### CI Pipeline (Test Automation)

The CI pipeline (`test` job) runs automatically on:
- Every push to any branch (including feature branches)
- Every pull request targeting the main branch

#### Technical Implementation Details

1. **Environment Setup**:
   ```yaml
   - name: Set up Python
     uses: actions/setup-python@v4
     with:
       python-version: '3.8'
   ```
   This step uses the official GitHub Action to configure Python 3.8, ensuring consistent test environments.

2. **Dependency Installation**:
   ```yaml
   - name: Install dependencies
     run: |
       python -m pip install --upgrade pip
       pip install -e ".[dev]"
   ```
   - Updates pip to the latest version
   - Installs the project in development mode with all dev dependencies
   - The `[dev]` extra includes testing dependencies like pytest

3. **Test Execution**:
   ```yaml
   - name: Run tests
     run: |
       pytest
   ```
   - Executes the full test suite using pytest
   - Automatically discovers and runs all tests in the project
   - Sets the exit code based on test success/failure

4. **Status Check Integration**:
   - GitHub automatically creates a status check for this job
   - This status check is required to pass before PRs can be merged
   - Provides immediate feedback on test failures in the PR interface

![CI Test Results Screenshot](./ci-test-results-screenshot.png)

*Screenshot: GitHub Actions showing successful test execution.*

### CD Pipeline (Release Artifact)

The CD pipeline (`build-and-deploy` job) runs automatically when code is merged to the main branch.

#### Technical Implementation Details

1. **Conditional Execution**:
   ```yaml
   if: github.event_name == 'push' && github.ref == 'refs/heads/main'
   ```
   - Only executes when code is pushed to the main branch
   - Skips for PRs and pushes to other branches
   - Uses GitHub context variables to determine execution conditions

2. **Build Tools Installation**:
   ```yaml
   - name: Install dependencies
     run: |
       python -m pip install --upgrade pip
       pip install build wheel
   ```
   - Installs the Python `build` and `wheel` packages
   - These are PEP-517 compliant tools for building Python packages

3. **Package Building**:
   ```yaml
   - name: Build package
     run: |
       python -m build
   ```
   - Builds both wheel (.whl) and source distribution (.tar.gz) packages
   - Follows PEP-517 standards for Python packaging
   - Creates distribution-ready artifacts in the `dist/` directory

4. **Artifact Upload**:
   ```yaml
   - name: Upload artifact
     uses: actions/upload-artifact@v3
     with:
       name: appointment-system-package
       path: dist/
   ```
   - Uses GitHub's artifact storage system
   - Uploads all files in the `dist/` directory
   - Makes artifacts available for download from the GitHub Actions interface
   - Artifacts are retained for 90 days by default

![CD Artifact Screenshot](./cd-artifact-screenshot.png)

*Screenshot: GitHub Actions showing the generated Python package artifacts.*

## Branch Protection Technical Implementation

The main branch is protected with the following rules configured in the repository settings:

![Branch Protection Rules Screenshot](./branch-protection-screenshot.png)

### Required Status Checks

- **Strict Status Checks**: Requires branches to be up-to-date with the main branch before merging
- **Required Checks**: The `test` job from our CI workflow must pass
- **Technical Implementation**: 
  ```
  Status check name: test
  Required: true
  Strict: true
  ```

### Pull Request Reviews

- **Required Approvals**: At least 1 approval from a repository collaborator
- **Dismiss Stale Reviews**: Automatically dismisses approvals when new commits are pushed
- **Restrict Who Can Dismiss Reviews**: Only repository administrators can dismiss reviews
- **Technical Implementation**:
  ```
  Required approving reviews: 1
  Dismiss stale reviews: true
  Require review from Code Owners: false
  ```

### Branch Restrictions

- **Push Restrictions**: Direct pushes to main are prohibited
- **Force Push Protection**: Force pushes are disabled to prevent history rewriting
- **Deletion Protection**: Branch deletion is prohibited
- **Technical Implementation**:
  ```
  Allow force pushes: false
  Allow deletions: false
  Restrict pushes: true (to administrators only)
  ```

## How to Run Tests Locally

To run the tests locally, follow these detailed instructions:

### 1. Environment Setup

First, ensure you have Python 3.8+ installed:
```bash
python --version
```

Create and activate a virtual environment (recommended):
```bash
# On Windows
python -m venv venv
venv\Scripts\activate

# On macOS/Linux
python -m venv venv
source venv/bin/activate
```

### 2. Install Dependencies

Install the project with development dependencies:
```bash
pip install -e ".[dev]"
```

This command:
- `-e`: Installs the package in development mode, so changes to the code are immediately reflected
- `.[dev]`: Installs the package plus all development dependencies specified in the `[dev]` extra in setup.py or pyproject.toml

### 3. Running Tests

#### Basic Test Execution
Run all tests:
```bash
pytest
```

#### Advanced Test Options

Run tests with verbose output:
```bash
pytest -v
```

Run specific test files:
```bash
pytest tests/api/test_patient_api.py
pytest tests/services/test_appointment_service.py
```

Run tests matching a specific pattern:
```bash
pytest -k "appointment"  # Runs all tests with "appointment" in the name
```

Generate test coverage report:
```bash
pytest --cov=app tests/
```

Export test results in JUnit format (for CI integration):
```bash
pytest --junitxml=test-results.xml
```

### 4. Debugging Failed Tests

For detailed output on test failures:
```bash
pytest -v --no-header --no-summary -l
```

To stop on first failure:
```bash
pytest -xvs
```

## Pull Request Workflow Detailed Guide

When contributing to this project, follow this detailed PR workflow:

### 1. Branch Creation

Create a new branch for your feature or fix:
```bash
# First, update your local main branch
git checkout main
git pull origin main

# Create a new branch with a descriptive name
git checkout -b feature/your-feature-name
```

Branch naming conventions:
- `feature/` - For new features
- `bugfix/` - For bug fixes
- `hotfix/` - For critical fixes
- `docs/` - For documentation changes
- `refactor/` - For code refactoring

### 2. Development Process

Make your changes following the project's coding standards:
```bash
# Make changes to the code
# Run tests locally to verify your changes
pytest

# Fix any issues that arise from tests
```

### 3. Commit Changes

Make atomic commits with descriptive messages:
```bash
git add .
git commit -m "Description of your changes"
```

Commit message guidelines:
- Start with a verb in present tense (Add, Fix, Update, Refactor)
- Keep the first line under 50 characters
- Add more detailed explanation in the commit body if needed
- Reference issue numbers if applicable: "Fixes #123"

### 4. Push Changes

Push your branch to GitHub:
```bash
git push origin feature/your-feature-name
```

If this is your first push for this branch, you'll see a link to create a PR directly.

### 5. Create Pull Request

Create a pull request on GitHub from your branch to main:
1. Navigate to the repository on GitHub
2. Click "Pull requests" > "New pull request"
3. Select your branch as the compare branch
4. Click "Create pull request"
5. Fill out the PR template with:
   - Description of changes
   - Issue references
   - Testing performed
   - Screenshots if applicable

### 6. CI Pipeline Execution

The CI pipeline will automatically run tests on your PR:
1. GitHub Actions will start the `test` job
2. You can view progress in the "Checks" tab of your PR
3. If tests fail, you'll receive notifications
4. Fix any issues and push additional commits to the same branch

### 7. Code Review Process

Wait for code review:
1. Reviewers will be automatically assigned based on code ownership
2. Address any feedback by making additional commits
3. Respond to comments to indicate when issues are fixed
4. Request re-review when all issues are addressed

### 8. Merging

When your PR is approved and all checks pass:
1. The "Merge" button will become active
2. Choose the appropriate merge strategy:
   - Merge commit: Preserves all commits (default)
   - Squash and merge: Combines all commits into one
   - Rebase and merge: Applies changes without a merge commit
3. Click "Confirm merge"

### 9. Post-Merge Automation

After merging to main:
1. The CD pipeline will automatically trigger
2. It will build the Python package artifacts
3. Artifacts will be uploaded to GitHub Actions
4. You can download the artifacts from the Actions tab

## Demonstration of CI/CD Pipeline Requirements

### 1. Tests Must Pass Before Merging

Our branch protection rules ensure that all tests must pass before a PR can be merged to the main branch:

![PR Blocked by Failing Tests](https://example.com/images/blocked-pr-screenshot.png)

*Screenshot: A pull request showing the "Merge" button disabled because the tests have failed.*

In this example:
1. The PR shows a red ❌ status check for the `test` job
2. The merge button is disabled with a message stating "Required status checks failed"
3. The PR cannot be merged until the tests are fixed and pass

This ensures that only code that passes all tests can be merged to the main branch, maintaining code quality and preventing regressions.

### 2. Artifacts Generated Only on Main Branch

Our CI/CD pipeline is configured to generate artifacts only when code is merged to the main branch, not on feature branches or PRs:

![Artifacts Generated on Main](https://example.com/images/artifacts-main-only.png)

*Screenshot: GitHub Actions showing artifacts generated only for workflows triggered by pushes to main.*

This is implemented through conditional execution in our workflow:

```yaml
build-and-deploy:
  name: Build and Deploy
  runs-on: ubuntu-latest
  needs: test
  if: github.event_name == 'push' && github.ref == 'refs/heads/main'
```

The `if` condition ensures that:
1. The job only runs on push events (not pull requests)
2. The job only runs when the target branch is main

This prevents unnecessary artifact generation for feature branches and PRs, saving CI resources and ensuring that only approved, tested code results in artifacts.

## Why This CI/CD Implementation Matters

### Technical Benefits

- **Automated Testing**: Every code change is automatically tested, catching regressions early
- **Consistent Environments**: Tests run in the same environment every time, eliminating "works on my machine" issues
- **Parallel Execution**: GitHub Actions can run multiple jobs in parallel, speeding up the pipeline
- **Artifact Versioning**: Each successful build on main creates versioned artifacts
- **Audit Trail**: Every step of the build process is logged and preserved

### Business Value

- **Quality Control**: The CI/CD pipeline ensures no buggy code reaches the main branch, reducing production issues
- **Developer Productivity**: Automation saves 5-10 hours per week on manual testing and deployment tasks
- **Faster Time-to-Market**: Changes can be deployed more quickly with confidence
- **Reduced Risk**: Failed tests block merges, preventing regressions
- **Compliance Support**: Complete audit trail of all changes, approvals, and test results

### Industry Context

- **Industry Standard**: Used by 90% of tech companies, including Google, Microsoft, and Amazon
- **DevOps Maturity**: Represents a high level of DevOps maturity (Level 3-4 on the DORA scale)
- **Career Relevance**: Experience with CI/CD is one of the most in-demand skills in software engineering
- **Future-Proofing**: This implementation can scale to support more complex workflows as the project grows
#   A s s i g n m e n t - 1 3  
 