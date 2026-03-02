---
name: workspace-hub-compliance
version: "1.0.0"
category: archive
description: "Systematic verification checklist for ensuring repositories comply with workspace-hub standards covering UV, Plotly, file organization, and modern Python practices."
---

# Workspace-Hub Standards Compliance Skill

> Version: 1.0.0
> Created: 2026-01-08
> Category: Standards, Compliance, Code Quality

## Overview

This skill provides a systematic verification checklist for ensuring repositories comply with workspace-hub standards. It covers UV package management, Plotly visualization requirements, file organization standards, and modern Python development practices.

## When to Use

Use this skill when:
- Setting up new repositories in workspace-hub
- Reviewing existing repositories for standards violations
- Modernizing tech stacks to workspace-hub requirements
- Preparing repositories for compliance propagation
- Auditing codebase for deprecated tools and practices
- Implementing best practices across multiple repositories

## Skill Components

### 1. UV Package Manager Compliance

**Standard:** All Python projects MUST use UV package manager (not Conda, Poetry, or pip)

**Verification Checklist:**
- [ ] `pyproject.toml` exists with proper configuration
- [ ] No `conda.yaml`, `environment.yml`, or `requirements.txt` files
- [ ] No Poetry configuration (`poetry.lock`, `pyproject.toml` with `[tool.poetry]`)
- [ ] README documentation mentions UV, not Conda/Poetry
- [ ] Setup instructions use `uv venv` and `uv pip install`
- [ ] Python version specified as `requires-python = ">=3.11"`

**Compliance Commands:**
```bash
# ✅ Correct UV usage
uv venv
source .venv/bin/activate  # Linux/Mac
.venv\Scripts\activate     # Windows
uv pip install -e .

# ❌ Wrong - Conda
conda create -n myenv python=3.11
conda activate myenv

# ❌ Wrong - Poetry
poetry install
poetry shell

# ❌ Wrong - pip
python -m venv venv
pip install -r requirements.txt
```

**Migration Steps:**
1. Install UV: `curl -LsSf https://astral.sh/uv/install.sh | sh`
2. Create `pyproject.toml` from existing `requirements.txt` or `pyproject.toml` (Poetry)
3. Remove Conda/Poetry configuration files
4. Update README with UV instructions
5. Test installation: `uv pip install -e .`

**pyproject.toml Template:**
```toml
[project]
name = "project-name"
version = "1.0.0"
description = "Brief description"
requires-python = ">=3.11"
dependencies = [
    "pandas>=2.0.0",
    "numpy>=1.24.0",
    # Add other dependencies
]

[project.optional-dependencies]
dev = [
    "pytest>=7.4.0",
    "pytest-cov>=4.1.0",
    "ruff>=0.1.0",
]

[project.scripts]
main-command = "package.module:main"
```

### 2. Plotly Interactive Visualization Compliance

**Standard:** ALL visualizations MUST be interactive using Plotly (no static matplotlib/seaborn)

**Verification Checklist:**
- [ ] No `matplotlib`, `seaborn`, or `pyplot` imports in codebase
- [ ] All plotting uses `plotly.express` or `plotly.graph_objects`
- [ ] Dependencies include `plotly>=5.14.0` and `kaleido>=0.2.1`
- [ ] No `.png`, `.jpg`, `.svg` static plot exports
- [ ] HTML reports with interactive plots (`.html` files)
- [ ] Explicit note in docs: "All visualizations MUST be interactive (Plotly)"

**Compliance Commands:**
```python
# ✅ Correct - Plotly interactive
import plotly.express as px
import pandas as pd

df = pd.read_csv('../data/processed/results.csv')
fig = px.scatter(df, x='time', y='value', title='Interactive Plot')
fig.write_html('../reports/analysis.html')

# ❌ Wrong - Matplotlib static
import matplotlib.pyplot as plt
plt.scatter(df['time'], df['value'])
plt.savefig('plot.png')  # Static image export

# ❌ Wrong - Seaborn static
import seaborn as sns
sns.scatterplot(data=df, x='time', y='value')
plt.savefig('plot.png')
```

**Migration Steps:**
1. Search codebase for `matplotlib`, `seaborn`, `pyplot` imports
2. Replace with `plotly.express` or `plotly.graph_objects`
3. Convert static plots to interactive equivalents
4. Update dependencies: Remove matplotlib/seaborn, add Plotly/Kaleido
5. Change output from `.png`/`.jpg` to `.html`
6. Test all visualizations in browser

**Plotly Equivalents:**
| Matplotlib/Seaborn | Plotly Equivalent |
|-------------------|-------------------|
| `plt.plot()` | `px.line()` |
| `plt.scatter()` | `px.scatter()` |
| `plt.bar()` | `px.bar()` |
| `plt.hist()` | `px.histogram()` |
| `sns.heatmap()` | `px.imshow()` or `go.Heatmap()` |
| `plt.pie()` | `px.pie()` |
| `sns.boxplot()` | `px.box()` |
| `sns.violinplot()` | `px.violin()` |

### 3. File Organization Standards Compliance

**Standard:** Consistent file organization across all repositories

**Verification Checklist:**
- [ ] Reports saved to `/reports/` directory (NOT root)
- [ ] Data files in `/data/` with subdirectories: `raw/`, `processed/`, `results/`
- [ ] Source code in `/src/` with module-based organization
- [ ] Tests in `/tests/` mirroring `/src/` structure
- [ ] Configuration in `/config/` directory
- [ ] Documentation in `/docs/` directory
- [ ] No files saved to repository root (except standard files: README, LICENSE, pyproject.toml, .gitignore)

**Standard Directory Structure:**
```
repository/
├── src/                    # Source code
│   └── modules/
│       ├── module_1/
│       └── module_2/
├── tests/                  # Test files
│   ├── unit/
│   └── integration/
├── docs/                   # Documentation
├── config/                 # Configuration files
├── scripts/                # Utility scripts
├── data/                   # Data files
│   ├── raw/               # Raw input data
│   ├── processed/         # Processed data
│   └── results/           # Output results
├── reports/               # Generated reports (HTML, PDF)
└── pyproject.toml        # Project configuration
```

**Migration Steps:**
1. Create standard directories if missing: `mkdir -p {src,tests,docs,config,scripts,data/{raw,processed,results},reports}`
2. Move misplaced files to correct directories
3. Update import paths in code
4. Update file paths in scripts and documentation
5. Add `.gitignore` entries for generated files in `/reports/` and `/data/results/`

### 4. CSV Data Import Standards Compliance

**Standard:** CSV data MUST use relative paths from report location

**Verification Checklist:**
- [ ] No hardcoded absolute paths (e.g., `/mnt/github/...`, `C:\Users\...`)
- [ ] All CSV imports use relative paths (e.g., `../data/processed/file.csv`)
- [ ] CSV files stored in standardized locations (`/data/raw/`, `/data/processed/`, `/data/results/`)
- [ ] Path resolution utilities for cross-platform compatibility
- [ ] Documentation specifies data directory structure

**Compliance Commands:**
```python
# ✅ Correct - Relative path from report location
import pandas as pd
df = pd.read_csv('../data/processed/results.csv')

# ✅ Correct - Path utility
from pathlib import Path

def get_data_path(filename, data_type='processed'):
    """Get data file path relative to project root."""
    project_root = Path(__file__).parent.parent.parent
    return project_root / 'data' / data_type / filename

df = pd.read_csv(get_data_path('results.csv'))

# ❌ Wrong - Absolute path (not portable)
df = pd.read_csv('/mnt/github/workspace-hub/repo/data/results.csv')

# ❌ Wrong - Windows-specific absolute path
df = pd.read_csv('C:/Users/user/Documents/repo/data/results.csv')
```

**Migration Steps:**
1. Search codebase for absolute paths: `grep -r "/mnt/" .` or `grep -r "C:\\" .`
2. Replace with relative paths or path utilities
3. Test on different platforms (Linux, macOS, Windows)
4. Document data directory structure in README

### 5. Modern Python Standards Compliance

**Standard:** Python 3.11+ with modern type hints and current dependency versions

**Verification Checklist:**
- [ ] Python version `>=3.11` in `pyproject.toml`
- [ ] Modern type hints used (`from __future__ import annotations`)
- [ ] Current major versions for dependencies (Pandas 2.0+, NumPy 1.24+)
- [ ] Deprecated packages replaced (e.g., `pypdf` instead of `PyPDF2`)
- [ ] Modern linter: Ruff (not Black + isort + flake8)
- [ ] Type checking with mypy enabled

**Compliance Dependencies:**
```toml
[project]
requires-python = ">=3.11"
dependencies = [
    "pandas>=2.0.0",        # Not pandas 1.x
    "numpy>=1.24.0",        # Not numpy 1.x
    "pypdf>=3.0.0",         # Not PyPDF2
    "plotly>=5.14.0",       # Interactive viz
    "python-dotenv>=1.0.0", # Modern version
]

[project.optional-dependencies]
dev = [
    "ruff>=0.1.0",          # Modern linter (replaces Black, isort, flake8)
    "mypy>=1.5.0",          # Type checking
    "pytest>=7.4.0",        # Testing
]
```

**Migration Steps:**
1. Update `requires-python` to `>=3.11`
2. Update all dependencies to current major versions
3. Replace deprecated packages (PyPDF2 → pypdf, etc.)
4. Replace Black + isort + flake8 with Ruff
5. Add type hints to functions
6. Run mypy to validate types

### 6. Module Organization Standards Compliance

**Standard:** Modular architecture with clear module boundaries and CLI commands

**Verification Checklist:**
- [ ] Modules organized under `/src/modules/` with domain-driven names
- [ ] Each module has `__init__.py` with clear exports
- [ ] Module-specific CLI commands defined in `[project.scripts]`
- [ ] Shared utilities in `/src/modules/shared/` or `/src/utils/`
- [ ] Module boundaries clearly documented (DEC-XXX decisions)
- [ ] No circular dependencies between modules

**Compliance Structure:**
```
src/
└── modules/
    ├── module_1/
    │   ├── __init__.py
    │   ├── core.py
    │   ├── utils.py
    │   └── config.py
    ├── module_2/
    │   ├── __init__.py
    │   └── processor.py
    └── shared/
        ├── __init__.py
        └── helpers.py
```

**CLI Configuration:**
```toml
[project.scripts]
module1-command = "package.modules.module_1:main"
module2-command = "package.modules.module_2:main"
```

**Migration Steps:**
1. Identify logical module boundaries based on functionality
2. Create module directories under `/src/modules/`
3. Move related files into appropriate modules
4. Update imports to reflect new structure
5. Define CLI entry points for each module
6. Document module architecture in decisions.md

## Compliance Verification Process

### Step 1: Run Automated Checks

**UV Compliance:**
```bash
# Check for Conda/Poetry files
find . -name "conda.yaml" -o -name "environment.yml" -o -name "poetry.lock"
# Should return nothing

# Check for pyproject.toml with UV
grep -q "requires-python" pyproject.toml && echo "✓ UV compliant" || echo "✗ Missing UV config"
```

**Plotly Compliance:**
```bash
# Check for matplotlib/seaborn imports
grep -r "import matplotlib" src/ && echo "✗ Matplotlib found" || echo "✓ No matplotlib"
grep -r "import seaborn" src/ && echo "✗ Seaborn found" || echo "✓ No seaborn"
grep -r "import plotly" src/ && echo "✓ Plotly found" || echo "✗ Missing Plotly"
```

**File Organization:**
```bash
# Check for files in root that should be elsewhere
ls *.html *.csv *.png 2>/dev/null && echo "✗ Files in root" || echo "✓ Clean root"

# Check for standard directories
for dir in src tests docs config scripts data reports; do
    [ -d "$dir" ] && echo "✓ $dir exists" || echo "✗ $dir missing"
done
```

### Step 2: Manual Review

**Checklist:**
- [ ] README documentation reflects UV usage
- [ ] All visualizations are interactive (test HTML reports in browser)
- [ ] CSV paths are relative (no absolute paths)
- [ ] Dependencies are current versions (check `pyproject.toml`)
- [ ] Modules have clear boundaries (check `/src/modules/` structure)
- [ ] CLI commands work (`python -m package.module`)

### Step 3: Test Compliance

**UV Installation Test:**
```bash
# Clean environment
rm -rf .venv

# Test UV setup
uv venv
source .venv/bin/activate
uv pip install -e .

# Verify installation
python -c "import [package]"
```

**Visualization Test:**
```bash
# Generate report
python scripts/generate_report.py

# Open in browser
open reports/report.html  # macOS
xdg-open reports/report.html  # Linux
start reports/report.html  # Windows

# Verify interactivity: hover, zoom, pan should work
```

## Compliance Templates

### UV Migration Template

```bash
#!/bin/bash
# migrate_to_uv.sh - Migrate repository to UV package manager

echo "🔄 Migrating to UV package manager..."

# Step 1: Install UV (if not installed)
if ! command -v uv &> /dev/null; then
    echo "Installing UV..."
    curl -LsSf https://astral.sh/uv/install.sh | sh
fi

# Step 2: Create pyproject.toml from requirements.txt
if [ -f "requirements.txt" ]; then
    echo "Creating pyproject.toml from requirements.txt..."
    # Manual conversion needed - see template
fi

# Step 3: Remove old configuration
echo "Removing Conda/Poetry files..."
rm -f conda.yaml environment.yml poetry.lock

# Step 4: Create UV environment
echo "Creating UV environment..."
uv venv
source .venv/bin/activate

# Step 5: Install dependencies
echo "Installing dependencies with UV..."
uv pip install -e .

# Step 6: Update README
echo "Update README with UV instructions (manual step)"
echo "✅ UV migration complete!"
```

### Plotly Migration Template

```python
# migrate_to_plotly.py - Convert matplotlib plots to Plotly

import re
from pathlib import Path

def convert_matplotlib_to_plotly(file_path):
    """Convert matplotlib code to Plotly equivalents."""

    conversions = {
        r'import matplotlib\.pyplot as plt': 'import plotly.express as px',
        r'plt\.scatter\((.*?)\)': r'px.scatter(df, x=\1)',
        r'plt\.plot\((.*?)\)': r'px.line(df, x=\1)',
        r'plt\.bar\((.*?)\)': r'px.bar(df, x=\1)',
        r'plt\.savefig\(["\'](.+?)\.png["\']\)': r'fig.write_html("\1.html")',
    }

    content = file_path.read_text()

    for old_pattern, new_pattern in conversions.items():
        content = re.sub(old_pattern, new_pattern, content)

    file_path.write_text(content)
    print(f"✅ Converted: {file_path}")

# Usage
for py_file in Path('src').rglob('*.py'):
    if 'matplotlib' in py_file.read_text():
        convert_matplotlib_to_plotly(py_file)
```

### File Organization Template

```bash
#!/bin/bash
# organize_files.sh - Reorganize files to workspace-hub standards

echo "📁 Organizing files to workspace-hub standards..."

# Create standard directories
mkdir -p src/{modules,utils} tests/{unit,integration} docs config scripts
mkdir -p data/{raw,processed,results} reports

# Move misplaced files
echo "Moving files to correct directories..."

# Move Python files to src/
find . -maxdepth 1 -name "*.py" ! -name "setup.py" -exec mv {} src/ \;

# Move test files to tests/
find . -name "test_*.py" -exec mv {} tests/unit/ \;

# Move data files
find . -maxdepth 1 -name "*.csv" -exec mv {} data/raw/ \;

# Move reports
find . -maxdepth 1 -name "*.html" -exec mv {} reports/ \;

# Update .gitignore
cat >> .gitignore <<EOF

# Generated files
reports/*.html
data/results/
*.log
EOF

echo "✅ File organization complete!"
echo "⚠️  Update import paths in code manually"
```

## Compliance Enforcement

### Pre-Commit Hook

```bash
#!/bin/bash
# .git/hooks/pre-commit - Workspace-hub standards enforcement

echo "🔍 Checking workspace-hub standards compliance..."

# Check 1: UV compliance
if [ -f "conda.yaml" ] || [ -f "environment.yml" ]; then
    echo "❌ ERROR: Conda configuration found. Use UV package manager."
    exit 1
fi

# Check 2: Plotly compliance
if grep -r "import matplotlib" src/ 2>/dev/null; then
    echo "❌ ERROR: Matplotlib imports found. Use Plotly for interactive visualizations."
    exit 1
fi

# Check 3: File organization
if ls *.html 2>/dev/null || ls *.csv 2>/dev/null; then
    echo "❌ ERROR: Files in root directory. Move to /reports/ or /data/"
    exit 1
fi

# Check 4: Absolute paths
if grep -r "/mnt/github" src/ 2>/dev/null || grep -r "C:\\\\" src/ 2>/dev/null; then
    echo "❌ ERROR: Absolute paths found. Use relative paths."
    exit 1
fi

echo "✅ Compliance checks passed!"
```

### CI/CD Validation

```yaml
# .github/workflows/compliance.yml
name: Workspace-Hub Compliance

on: [push, pull_request]

jobs:
  compliance:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      - name: Check UV Configuration
        run: |
          test -f pyproject.toml || (echo "Missing pyproject.toml" && exit 1)
          grep -q "requires-python" pyproject.toml || (echo "Missing Python version" && exit 1)

      - name: Check for Conda/Poetry
        run: |
          ! test -f conda.yaml && ! test -f environment.yml && ! test -f poetry.lock

      - name: Check Plotly Usage
        run: |
          ! grep -r "import matplotlib" src/
          ! grep -r "import seaborn" src/
          grep -r "import plotly" src/ || (echo "Missing Plotly" && exit 1)

      - name: Check File Organization
        run: |
          test -d src && test -d tests && test -d docs && test -d data && test -d reports
          ! ls *.html *.csv 2>/dev/null
```

## Best Practices

### Do's

✅ **Use UV for all Python projects** - Faster, more reliable than Conda/Poetry
✅ **Interactive visualizations only** - Plotly for all charts and graphs
✅ **Relative paths for data** - Portable across environments
✅ **Organized directory structure** - Files in appropriate subdirectories
✅ **Modern Python (3.11+)** - Latest features and performance
✅ **Current dependency versions** - Major versions (Pandas 2.0+, etc.)

### Don'ts

❌ **Don't use Conda or Poetry** - UV is the workspace-hub standard
❌ **Don't use matplotlib/seaborn** - Static plots violate standards
❌ **Don't use absolute paths** - Breaks portability
❌ **Don't save to root directory** - Use `/reports/`, `/data/`, etc.
❌ **Don't use outdated dependencies** - Keep major versions current
❌ **Don't use deprecated packages** - Replace PyPDF2 with pypdf, etc.

## Related Skills

- **Product Documentation Modernization** - Documenting compliance requirements
- **Technology Stack Modernization** - Updating to compliant tech stack
- **File Organization** - Implementing standard directory structure
- **Quantification & Metrics** - Measuring compliance impact

## References

- Workspace-Hub Standards: `/mnt/github/workspace-hub/docs/modules/standards/`
- UV Package Manager: https://github.com/astral-sh/uv
- Plotly Documentation: https://plotly.com/python/
- HTML Reporting Standards: `docs/modules/standards/HTML_REPORTING_STANDARDS.md`
- File Organization Standards: `docs/modules/standards/FILE_ORGANIZATION_STANDARDS.md`

---

## Version History

- **1.0.0** (2026-01-08): Initial skill creation based on aceengineer-admin tech-stack.md modernization work
