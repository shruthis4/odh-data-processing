# **ODH Data Processing - Test Strategy Document**

## **1. Overview & Current State**

### **1.1 Purpose**

This document defines the testing strategy for the ODH Data Processing repository, outlining testing approaches for each repository module, quality metrics, and implementation roadmap. The strategy balances comprehensive coverage with practical implementation bandwidth for the repository.

### **1.2 Repository Modules**

The ODH Data Processing repository contains several distinct module types, each requiring tailored testing approaches:

- **Jupyter Notebooks** (Tutorials & Use Cases)
- **Kubeflow Pipelines** (KFP Components)  
- **Python Scripts** (Subset Selection & Utilities)
- **Custom Workbench Images** (Container Builds)
- **Configuration Files** (CI/CD, Dependencies)
- **Documentation** (README, Guides, Examples)

### **1.3 Current Testing Infrastructure**

**Existing Testing Components:**
- ✅ **Pytest Framework** - Notebook parameter validation
- ✅ **Pre-commit Hooks** - Code formatting (ruff) and notebook cleaning (nbstripout)
- ✅ **CI/CD Workflows** - Automated validation and limited-execution(only for document-coversion-standard )
- ✅ **Makefile Targets** - Easy test execution (`make test-notebook-parameters`)
- ✅ **Papermill Integration** - Notebook execution testing

**Testing Gaps:**
- Limited unit test coverage for Python modules
- No integration testing for KFP pipelines
- No container image validation
- No performance/load testing
- No security scanning integration

---

## **2. Testing Strategy by Repository Module**

### **2.1 Repository Module Testing Matrix**

**Status Legend:**
- ✅ **Done** - Implemented and working
- ❌ **Not Started** - No implementation yet  
- 🔄 **In Progress** - Partially implemented

| Repository Module | Testing Approach | Must-Have Test Cases | Nice-to-Have | Tools/Environment | Priority |
|-------------------|-----------------|-----------|--------------|--------|----------|
| **Jupyter Notebooks (Tutorials & Use Cases)** | Execution + Validation | Parameter cell validation ✅, Successful execution ❌ | Output validation, Performance benchmarks, Load testing with large datasets, Multi-environment testing, Vulnerability checks ❌  | pytest, papermill, nbformat, AWS EC2,snyk | HIGH |
| **KFP Pipelines (Standard & VLM)** | Component + Integration | Unit tests for components ❌ , Pipeline compilation ✅, Run pipeline with real cluster ❌ | Mock data execution, Performance profiling, Resource usage testing | pytest, kfp SDK, unittest.mock | HIGH |
| **Python Scripts - Subset Selection(TODO-talk to Ali)** | Unit + Integration | Function unit tests ❌, CLI interface testing ❌, File I/O validation ❌ | Performance benchmarks, Memory usage testing, Large dataset testing | pytest, pytest-mock, pytest-benchmark | HIGH |
| **Custom Workbench Image(may not need it anymore)** | Container Testing | Build success 🔄, Base functionality ❌, Security scanning ❌ ,entry-point scritp validation  ❌| Multi-arch builds, OpenShift AI integration | Docker, hadolint, trivy, pytest-docker | HIGH |
| **Configuration Files** | Validation + Syntax | YAML/TOML syntax validation ✅, Schema compliance ✅, Reference validation ❌ | Automated updates, Drift detection, Impact analysis | yamllint, jsonschema, custom validators | LOW |
| **Documentation** | Content + Link Validation | Link checking ❌, Code example execution ❌, Formatting validation ❌ | Accessibility testing, Translation validation | markdown-link-check, pytest-doctests | LOW |

---

## **3. Quality Metrics & Targets**

### **3.1 Code Coverage Metrics**

**Primary Metrics:**
```yaml
Code Coverage Targets:
  Python Modules:
    Must-Have: 70% line coverage
    Nice-to-Have: 85% line coverage + branch coverage
    
  Notebook Validation:
    Must-Have: 100% notebook parameter validation
    Nice-to-Have: 90% successful execution rate
    
  KFP Components:
    Must-Have: 60% function coverage
    Nice-to-Have: 80% + integration test coverage
```

**Coverage Implementation:**
```bash
# Generate coverage reports
pytest --cov=kubeflow-pipelines --cov=scripts --cov-report=html
pytest --cov=tests --cov-report=xml  # For CI integration

# Coverage thresholds in pyproject.toml
[tool.coverage.run]
source = ["kubeflow-pipelines", "scripts"]
omit = ["*/tests/*", "*/test_*"]

[tool.coverage.report]
fail_under = 70
show_missing = true
```

### **3.2 Quality Gates**

**Must-Have Quality Gates:**
- ✅ All notebooks execute successfully with default parameters
- ✅ 70%+ code coverage for Python modules  
- ✅ Automated dependency vulnerability scanning
- ✅ All KFP pipelines compile without errors
- ✅ Pre-commit hooks pass (formatting, linting)


**Nice-to-Have Quality Gates:**
- ⭐ 85%+ code coverage with branch coverage
- ⭐ Performance benchmarks within acceptable ranges
- ⭐ Cross-platform compatibility testing
- ⭐ Documentation links are valid
- ⭐ Integration testing with real ODH cluster
- ⭐ Accessibility compliance for documentation


### **3.4 Reliability Metrics**

**Test Reliability Targets:**
```yaml
Reliability Metrics:
  CI/CD Success Rate:
    Must-Have: 95% green builds on main branch
    Nice-to-Have: 98% + flaky test detection
    
```
---

## **5. Tools & Infrastructure**

### **5.1 Testing Framework Stack**

**Core Testing Tools:**
```yaml
Primary Framework: pytest
Extensions:
  - pytest-cov (Code coverage)
  - pytest-benchmark (Performance testing)
  - pytest-mock (Mocking and patching)
  - pytest-xdist (Parallel test execution)
  - pytest-html (HTML test reports)
  
Notebook Testing:
  - papermill (Notebook execution)
  - nbformat (Notebook validation) 
  - jupyter-client (Kernel management)
  
Container Testing:
  - docker (Container operations)
  - hadolint (Dockerfile linting)
  - trivy (Security scanning)
  - pytest-docker (Container testing integration)
```
### **5.2 CI/CD Integration**

**GitHub Actions Workflows:**

**Extended Validation Workflow:**
```yaml
# .github/workflows/test-comprehensive.yml
name: Comprehensive Testing

on:
  pull_request:
    branches: [main]
  push:
    branches: [main]

jobs:
  unit-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Run Python Unit Tests
        run: |
          pytest tests/ --cov=kubeflow-pipelines --cov=scripts \
                 --cov-report=xml --cov-report=html
      
      - name: Upload Coverage Reports  
        uses: codecov/codecov-action@v3
        with:
          file: ./coverage.xml
          
  notebook-tests:
    runs-on: ubuntu-latest  
    steps:
      - uses: actions/checkout@v4
      - name: Execute All Notebooks
        run: |
          pytest tests/test_notebook_execution.py -v
          
  container-tests:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - name: Build and Test Container
        run: |
          docker build -t test-workbench custom-workbench-image/
          trivy image --severity HIGH,CRITICAL test-workbench
```