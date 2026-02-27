---
name: code-commenting
description: Comprehensive code commenting methodology for Python projects. Use when user asks to add comments, annotations, or documentation to Python code files. Provides structured approach with module docstrings, class/function documentation, section separators, and inline comments.
---

# Code Commenting Skill

Add structured, professional comments to Python code files.

## Commenting Hierarchy

Add comments in the following hierarchical structure:

### 1. Module-Level Docstring

Place at the top of the file using triple quotes with separator lines:

```python
"""
============================================================================
Module Name - Brief Description
============================================================================
Detailed description of the module.

This module provides:
    - Feature 1
    - Feature 2
    - Feature 3

Note:
    Any important notes for users.
"""
```

### 2. Import Section Comments

Group imports by category using separator lines:

```python
# =============================================================================
# Standard Library Imports
# =============================================================================
import os
import random

# =============================================================================
# Third-Party Library Imports
# =============================================================================
from tqdm import tqdm

# =============================================================================
# Project Internal Imports
# =============================================================================
from myproject.core import Module
```

### 3. Section Separators

Use fixed-length separator lines to divide code blocks:

```python
# =============================================================================
# Type Aliases
# =============================================================================
ParamKey = Tuple[int, int]  # (num_ue, b) - parameter combination key

# =============================================================================
# Main Functions
# =============================================================================
def main():
    pass
```

### 4. Class Documentation

```python
class Metrics:
    """
    Statistics Collector - Collect metrics during simulation.
    
    Uses Welford's online algorithm for delay statistics.
    
    Attributes:
        total_requests: Total number of requests.
        collision_failures: Number of collision failures.
    
    Note:
        Only data within the statistics window is recorded.
    """
    
    def __init__(self) -> None:
        """Initialize all statistics counters."""
        # Counters
        self.total_requests: int = 0
```

### 5. Function Documentation

Use detailed docstrings with workflow, parameters, returns, and examples:

```python
def run_simulation(
    num_ue: int,
    frame: Frame,
) -> Dict:
    """
    Run simulation - Brief description.
    
    Detailed explanation of the function's purpose and behavior.
    
    Workflow:
        1. Step one
        2. Step two
        3. Step three
    
    Args:
        num_ue: Number of UEs.
            Detailed explanation of the parameter.
        
        frame: Frame structure configuration.
            - Sub-attribute 1
            - Sub-attribute 2
    
    Returns:
        Dict: Result dictionary.
            - key1: Description
            - key2: Description
    
    Example:
        >>> result = run_simulation(num_ue=100, frame=Frame())
        >>> print(result['success_rate'])
    
    Note:
        Any important notes.
    """
```

### 6. Step-by-Step Inline Comments

For complex functions, use step-by-step structure:

```python
def complex_function():
    # =========================================================================
    # Stage 1: Initialization
    # =========================================================================
    
    # -------------------------------------------------------------------------
    # Step 1.1: Calculate Parameters
    # -------------------------------------------------------------------------
    # Explain why we calculate this way
    total = calculate_total()
    
    # -------------------------------------------------------------------------
    # Step 1.2: Create Instances
    # -------------------------------------------------------------------------
    instance = create_instance()
    
    # =========================================================================
    # Stage 2: Main Loop
    # =========================================================================
    
    for i in range(total):
        # ---------------------------------------------------------------------
        # Phase 1: Request Phase
        # ---------------------------------------------------------------------
        # Detailed explanation of what this phase does
        process_request()
```

### 7. Inline Comments

```python
# Variable explanation
num_slots = 10  # m: number of request slots (paper notation)

# Condition explanation
if frame_idx < warmup_frames:
    continue  # Skip statistics during warmup period

# Algorithm step
delta = value - mean  # Welford Step 1: Calculate delta
```

## Formatting Guidelines

### Separator Lengths

- **Major sections** (=): 77 characters total
  ```
  # =============================================================================
  ```

- **Minor subsections** (-): 77 characters total
  ```
  # -------------------------------------------------------------------------
  ```

### Best Practices

1. **Don't repeat the code** - Comments should explain "why", not "what"
2. **Be consistent** - Use the same commenting style throughout the project
3. **Hierarchical structure** - Use `=` for major blocks, `-` for sub-blocks
4. **Step numbering** - Use Step X.Y numbering for complex workflows
5. **Type annotations** - Add type explanations for important variables
6. **Paper references** - Note formula numbers when code implements paper equations

## Workflow

1. **Read the file** - First read the entire file to understand its structure
2. **Add module docstring** - Add module-level documentation
3. **Organize imports** - Group imports with separator lines
4. **Document classes** - Add docstrings to each class
5. **Document functions** - Add detailed docstrings to each function
6. **Add section separators** - Divide code blocks with separator lines
7. **Add inline comments** - Add inline comments for complex logic
8. **Review** - Ensure comments are complete and consistent

## Template Reference

For quick-reference templates, see [references/templates.md](references/templates.md).

## Example: Complete File Structure

```python
"""
============================================================================
Module Name - Brief Description
============================================================================
Module description.
"""

# =============================================================================
# Standard Library Imports
# =============================================================================
import os

# =============================================================================
# Project Imports
# =============================================================================
from project import Module

# =============================================================================
# Constants
# =============================================================================
MAX_VALUE = 100  # Maximum value limit

# =============================================================================
# Class Definitions
# =============================================================================
class MyClass:
    """Class description."""
    pass

# =============================================================================
# Main Functions
# =============================================================================
def main():
    """Main function."""
    # -------------------------------------------------------------------------
    # Step 1: Initialization
    # -------------------------------------------------------------------------
    pass

# =============================================================================
# Module Exports
# =============================================================================
__all__ = ["MyClass", "main"]
```
