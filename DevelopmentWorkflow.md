# **ODH Data Processing - Development Workflow**

## **1. Quick Start**

### **Repository Overview**
The ODH Data Processing repository provides reference pipelines and examples for [Open Data Hub](https://github.com/opendatahub-io) / [Red Hat OpenShift AI](https://www.redhat.com/en/products/ai/openshift-ai):

- **Kubeflow Pipelines** - Document processing workflows using [Docling](https://docling-project.github.io/docling/)
- **Jupyter Notebooks** - Tutorials and use-case examples
- **Scripts** - Utility tools for data processing
- **Custom Workbench** - OpenShift AI container setup

### **Setup**
```bash
# 1. Clone repository
git clone https://github.com/opendatahub-io/data-processing.git
cd data-processing

# 2. Install dependencies
pip install -r requirements-dev.txt

# 3. Setup pre-commit hooks
pre-commit install

# 4. Verify setup
make test-notebook-parameters
```

### **Core Workflow**
1. **Create branch**: `git checkout -b feature/my-feature`
2. **Make changes** and commit regularly
3. **Test locally**: `make test-all`
4. **Push**: `git push origin feature/my-feature`
5. **Create PR** via GitHub interface
6. **Review & merge** - Mergify auto-merges when approved

## **2. Daily Development**

### **Repository-Specific Commands**
```bash
# Essential workflow commands
make test-all                 # Run all validation checks
make test-notebook-parameters # Validate notebook structure  
make format-notebooks         # Clean notebook outputs
make format-python-check      # Verify code formatting

# Pre-commit integration
pre-commit run --all-files    # Run all hooks manually
pre-commit install            # Setup hooks for commits
```

### **Branch Management**
```bash
# Start new work from latest main
git checkout main && git pull origin main
git checkout -b feature/your-feature-name

# Keep branch current with main
git fetch origin && git rebase origin/main
git push --force-with-lease origin feature/your-feature-name
```

### **Branch Naming Conventions**
- `feature/JIRA-123-add-validation` - New features with ticket reference
- `fix/notebook-timeout-issue` - Bug fixes  
- `docs/update-workflow-guide` - Documentation updates
- `chore/update-dependencies` - Maintenance tasks

### **Development Tools**

**Pre-commit Hooks** (auto-configured):
- **Ruff** - Python formatting and linting
- **NBStripout** - Notebook output cleaning
- Automatically runs on every commit

**Repository Testing:**
```bash
# Run specific test suites
pytest tests/test_notebook_parameters.py -v
pytest tests/ -k "notebook" -v

# Notebook validation
make test-notebook-parameters

# Full validation pipeline
make test-all
```

## **3. Pull Requests**

### **Creating a PR**

**Pre-PR Validation:**
```bash
# Mandatory checks before creating PR
make test-all                 # Must pass all tests
git status                   # Ensure clean working tree
git push origin feature/your-branch
```

**PR Requirements:**
1. **Title**: `feat(notebooks): JIRA-123 Add parameter validation`
   - Include JIRA ticket reference when applicable
   - Use conventional commit format
2. **Description**: Explain business value and technical changes
3. **Reviewers**: Minimum one team member with domain expertise
4. **Labels**: 
   - `backport/stable-3.0` - For stable branch fixes
   - `do-not-merge` - Temporary hold on merging
   - `breaking-change` - API or workflow changes

### **Auto-Merge System (Mergify)**

**Conditions for Auto-Merge:**
- ✅ **1+ approvals** with no change requests
- ✅ **All CI workflows pass** (validate-notebooks, execute-notebooks, validate-python)
- ✅ **No merge conflicts** with target branch
- ✅ **Ready for review** (not draft status)
- ✅ **No blocking labels** (`do-not-merge`, `breaking-change`)

**Mergify Troubleshooting:**
- **Check PR comments** for Mergify bot status updates
- **Missing CI condition** often indicates path filters didn't trigger workflows
- **Manual override**: Add `do-not-merge` label, then use GitHub merge button

### **PR Updates and Conflict Resolution**
```bash
# Address review feedback
git add . && git commit -m "address review: improve error handling"
git push origin feature/your-branch

# Resolve conflicts with main branch
git fetch origin && git rebase origin/main
# Resolve conflicts in files, then:
git add . && git rebase --continue
git push --force-with-lease origin feature/your-branch
```

## **4. CI/CD Workflows**

### **Automated Quality Gates**
Three workflows validate every PR:

1. **validate-notebooks.yml** - Notebook structure and parameter validation
2. **execute-notebooks.yml** - End-to-end notebook execution testing  
3. **validate-python.yml** - Code formatting and style compliance

### **Workflow-Specific Troubleshooting**
```bash
# Notebook validation failures
# → Add 'parameters' tag to code cell in Jupyter
# → Run: make format-notebooks

# Notebook execution timeouts
# → Reduce dataset size in examples
# → Optimize processing algorithms
# → Check for infinite loops or blocking operations

# Python validation failures  
pre-commit run --all-files
git add . && git commit --amend
```

### **Path-Based Triggering**
Workflows only run when relevant files change:
- **Notebooks**: `notebooks/**/*.ipynb`
- **Python**: `**/*.py`, `pyproject.toml`, `.pre-commit-config.yaml`
- **Workflows**: `.github/workflows/*.yml`

## **5. Automation Tools**

### **Mergify Configuration**
**Auto-merge conditions** (defined in `.github/mergify.yml`):
```yaml
- "#approved-reviews-by>=1"
- "#changes-requested-reviews-by=0" 
- "check-failure=~^$"
- "base=main"
- "label!=do-not-merge"
- "label!=breaking-change"
```

**Team-Specific Labels:**
- `do-not-merge` - Hold for coordination or special timing
- `breaking-change` - Requires additional architecture review
- `backport` - Automatic backport to stable -.0

### **Dependabot Integration**
**Automated dependency management** (configured in `.github/dependabot.yml`):
- **Security patches**: Immediate high-priority updates
- **Minor updates**: Weekly Saturday schedule (non-disruptive)
- **GitHub Actions**: Automated workflow dependency updates

**Dependabot PR Handling:**
- **Security updates**: Fast-track review and merge
- **Minor updates**: Validate changelog, approve if CI passes  
- **Major updates**: Manual review required (blocked by configuration)

### **Backporting Workflow**
**For stable release fixes:**
1. **Label merged PR** with `backport/stable-X.Y`
2. **Mergify creates backport PR** automatically to target branch
3. **Manual review and merge** backport PR (no auto-merge for backports)

**Conflict Resolution:**
```bash
# When backport has conflicts
git fetch origin
git checkout backport/stable-3.0/pr-123
# Resolve conflicts manually
git add . && git commit -m "resolve backport conflicts"
git push origin backport/stable-3.0/pr-123
# Request manual review and merge
```

## **6. Advanced Troubleshooting**

### **Mergify Edge Cases**
```bash
# PR not auto-merging despite approvals
# → Check PR comments for Mergify bot status
# → Verify all CI workflows completed (not just passed)
# → Ensure no draft status or blocking labels

# Mergify conditions unclear
# → Visit Mergify dashboard for detailed rule evaluation
# → Check .github/mergify.yml for current configuration
# → Use Mergify simulator for rule testing
```

### **CI/CD Advanced Issues**
```bash
# Workflow not triggering
# → Check path filters in .github/workflows/*.yml
# → Verify branch name matches workflow triggers
# → Check for syntax errors in workflow YAML

# Notebook execution environment issues
# → Verify requirements.txt has all dependencies
# → Check for hardcoded paths or environment assumptions
# → Validate parameter cell structure and tags
```

### **Development Environment Recovery**
```bash
# Complete pre-commit reset
pre-commit clean && pre-commit uninstall
pre-commit install

# Dependency environment reset
pip install --force-reinstall -r requirements-dev.txt

# Notebook kernel issues
pip install ipykernel
python -m ipykernel install --user --name python3
jupyter kernelspec list  # Verify installation
```

## **7. Emergency Procedures**

### **Hotfix Process**
For critical security issues or production-breaking bugs:

1. **Create hotfix branch**: `git checkout stable-3.0 && git checkout -b hotfix/critical-fix`
2. **Make minimal fix** (only address the specific issue)
3. **Emergency review**: Single reviewer, focus on fix correctness
4. **Override Mergify**: Add `do-not-merge` label, merge manually
5. **Release immediately**: Tag and create GitHub release
6. **Apply to main**: Create separate PR for main branch if needed

### **Release Planning**
For release planning, versioning, and release procedures, see: [`docs/maintainers/release-strategy.md`](docs/maintainers/release-strategy.md)

## **8. Quick Reference**

### **Essential Commands**
```bash
# Setup
git clone https://github.com/opendatahub-io/data-processing.git
cd data-processing && pip install -r requirements-dev.txt
pre-commit install

# Daily workflow
git checkout main && git pull origin main
git checkout -b feature/JIRA-123-description
make test-all
git add . && git commit -m "feat(component): JIRA-123 description"
git push origin feature/JIRA-123-description

# Quality assurance
make test-notebook-parameters  # Validate notebooks
make format-notebooks          # Clean outputs
pre-commit run --all-files    # Format and lint
```

### **Key Integrations**
- **Repository**: https://github.com/opendatahub-io/data-processing
- **Docling Framework**: https://docling-project.github.io/docling/
- **ODH Platform**: https://opendatahub.io/
- **Mergify Dashboard**: Check repository settings for team access

### **Configuration Files**
- **Dependencies**: `requirements-dev.txt`
- **Python config**: `pyproject.toml` (ruff, pytest settings)
- **Pre-commit hooks**: `.pre-commit-config.yaml`
- **CI workflows**: `.github/workflows/`
- **Mergify rules**: `.github/mergify.yml`
- **Dependabot**: `.github/dependabot.yml`
- **Tests**: `tests/`