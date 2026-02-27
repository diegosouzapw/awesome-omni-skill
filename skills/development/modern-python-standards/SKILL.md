---
name: modern-python-standards
description: Strict adherence to modern (3.11+), idiomatic, and type-safe Python development.
---

# Modern Python Standards
_Strict adherence to modern (3.11+), idiomatic, and type-safe Python development._

## Knowledge
* #### The Python Philosophy (Vybz Edition)
      *   **PEP 20 (The Zen):** Explicit is better than implicit. Simple is 
          better than complex.
      *   **PEP 8 (Style):** Strict adherence to formatting.
      *   **PEP 257 (Docs):** Use **Google Style** docstrings for all functions 
          and classes.
      *   **PEP 484 & 585:** You implement rigorous type hinting to ensure code 
          self-documentation and IDE support.
* #### Modern Syntax Mandates (Python 3.11+)
      *   **Typing (PEP 585 & 604):** Use built-in collection types for hinting (`list[str]`, `dict[str, Any]`) and the pipe operator for unions (`str | None`). Avoid importing `List`, `Dict`, `Union` from `typing`.
      *   **Pathing:** Strictly use `pathlib.Path`. Do NOT use `os.path.join` or string manipulation for file paths.
      *   **Data Structures:** Prefer `@dataclass` with type hints over raw dictionaries or complex `__init__` boilerplate for data objects.
      *   **String Formatting:** Use f-strings exclusively.

## Abilities
* Refactoring complex nested logic into flat, readable 'Happy Paths' (Guard Clauses).
* Implementing Context Managers (`with` statements) for safe resource handling (files, locks).
* Writing self-documenting code where variable names explain the 'What' and comments explain the 'Why'.
* Fundamentals: Deep understanding of data structures, functions, generators, iterators, error handling, and concurrency (multithreading/async).
* Object Oriented Programming
* Utilizing `if __name__ == '__main__':` blocks for module testability.
