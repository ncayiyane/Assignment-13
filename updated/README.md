# Assignment 13: CI/CD Pipeline with GitHub Actions for AI-Powered Appointment System

This project implements a robust CI/CD pipeline using GitHub Actions for the AI-Powered Smart Appointment Booking System, featuring automated testing, artifact generation, and branch protection rules.

## Project Structure
```
Repository/
├── .github/
│   └── workflows/
│       └── ci.yml           # CI/CD workflow configuration
├── PROTECTION.md            # Explanation of branch protection rules
├── README.md                # Project documentation
└── src/                     # Source code for the appointment system
    ├── api/                 # Backend API components
    ├── ai/                  # AI scheduling algorithms
    └── frontend/            # User interface components
```

## CI/CD Pipeline Technical Details
This project leverages GitHub Actions for continuous integration and continuous deployment, providing automated testing, quality assurance, and artifact generation for our healthcare appointment system.

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
   - Configures Python 3.8 runtime environment for AI components
   - Sets up Node.js for frontend testing

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
   This step uses the official GitHub Action to configure Python 3.8, ensuring consistent test environments for our AI algorithms.

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

## Branch Protection Technical Implementation

To ensure code quality and maintain a stable main branch, we've implemented the following branch protection rules:

### Required Status Checks
- The `test` job must pass before merging is allowed
- This ensures all tests are passing on the latest code

### Pull Request Reviews
- At least one approved review is required before merging
- This ensures code is reviewed by at least one other team member

### Branch Restrictions
- Direct pushes to the main branch are prohibited
- All changes must go through the pull request workflow

## How to Run Tests Locally

### 1. Environment Setup
```bash
# Create a virtual environment
python -m venv venv

# Activate the virtual environment
# On Windows
venv\Scripts\activate
# On macOS/Linux
source venv/bin/activate
```

### 2. Install Dependencies
```bash
# Install the project in development mode
pip install -e ".[dev]"
```

### 3. Running Tests
```bash
# Run all tests
pytest

# Run tests with verbose output
pytest -v

# Run tests with coverage report
pytest --cov=src
```

### 4. Debugging Failed Tests
If tests fail, check the following:
- Ensure all dependencies are installed
- Verify that the test environment is properly configured
- Check for any syntax errors in the code
- Review the test output for specific error messages

## Pull Request Workflow Detailed Guide

### 1. Branch Creation
```bash
# Create a new feature branch
git checkout -b feature/new-feature
```

### 2. Development Process
- Write code for the new feature
- Add tests for the new functionality
- Ensure all tests pass locally

### 3. Commit Changes
```bash
git add .
git commit -m "Add new feature: brief description"
```

### 4. Push Changes
```bash
git push origin feature/new-feature
```

### 5. Create Pull Request
- Go to the repository on GitHub
- Click "New Pull Request"
- Select your feature branch
- Add a descriptive title and detailed description
- Reference any related issues

### 6. CI Pipeline Execution
- The CI pipeline will automatically run
- Check that all tests pass
- Address any failures or issues

### 7. Code Review Process
- Wait for reviewer feedback
- Make requested changes if needed
- Push additional commits to the same branch

### 8. Merging
- Once approved and all checks pass, merge the PR
- Use the "Squash and merge" option for a clean history

### 9. Post-Merge Automation
- The CD pipeline will automatically run on the main branch
- Artifacts will be generated and uploaded
- The feature branch can be deleted

## Demonstration of CI/CD Pipeline Requirements

### 1. Tests Must Pass Before Merging
Our branch protection rules require all tests to pass before a pull request can be merged. This ensures that only working code is integrated into the main branch.

### 2. Artifacts Generated Only on Main Branch
Our CD pipeline is configured to run only when code is pushed to the main branch. This ensures that artifacts are only generated for stable, reviewed code.

## Why This CI/CD Implementation Matters

### Technical Benefits
- **Automated Testing**: Catches bugs early in the development process
- **Consistent Builds**: Ensures reproducible builds across environments
- **Quality Control**: Enforces code reviews and passing tests
- **Artifact Management**: Automatically generates and stores release artifacts

### Business Value
- **Faster Time to Market**: Streamlines the development and release process
- **Reduced Risk**: Minimizes the chance of introducing bugs into production
- **Improved Collaboration**: Facilitates team collaboration through pull requests
- **Enhanced Reliability**: Ensures the appointment system is stable and reliable

### Industry Context
In the healthcare industry, reliability and security are paramount. Our CI/CD pipeline helps ensure that:
- Patient data remains secure through consistent testing
- The appointment system remains available and functional
- New features can be safely added without disrupting existing functionality
- The system meets healthcare industry compliance requirements
