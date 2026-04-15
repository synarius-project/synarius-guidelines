Programming guidelines
========================

Python version
--------------

**Develop and test with Python 3.11** (aligned with CI in the Synarius repositories). Use a virtual environment created with that interpreter; avoid mixing binary wheels from different Python minor versions (e.g. ``cp311`` packages under Python 3.12).

The **synarius-core**, **synarius-apps**, and **synarius-studio** packages declare ``requires-python = ">=3.11,<3.12"`` in each ``pyproject.toml``. Installing with the wrong interpreter (for example Python 3.12) should fail at dependency resolution instead of leaving incompatible wheels.

Use ``python -m pip ...`` with the same interpreter that owns the virtual environment. When you change Python **minor** versions, **recreate the venv** and reinstall; do not copy or reuse ``site-packages`` from another Python.

Repositories and boundaries
---------------------------

Synarius is split into separate Git repositories; each can be cloned and used according to your needs.

**synarius-core**

* GUI-less simulation and data backend.
* Must **not** depend on PySide/Qt or Synarius Studio.

**synarius-apps**

* Desktop tools and shared libraries: DataViewer, ParaWiz, ``synariustools`` (e.g. plot widgets).
* Depends on **synarius-core**; does **not** require Synarius Studio. Useful for users who only need viewers or parameter workflows.

**synarius-studio**

* Full graphical modeling and simulation IDE.
* Depends on **synarius-core** and **synarius-apps** (via declared dependencies). Keep simulation logic in **core**, not in Studio.

**Rule of thumb:** simulation logic belongs in **synarius-core**; Studio and Apps focus on UI, tooling, and integration.

General Principle
------------------
* Prefer clarity over cleverness.
* Prefer flat structure over deeply nested structure.
* Prefer well-named small methods over large monolithic ones.

Code style
----------

* Follow **PEP 8** and project conventions in existing code.
* Prefer small, focused functions and explicit behavior.
* Add docstrings where they help maintainers (public APIs, non-obvious behavior).

Complexity and Nesting
----------------------
* Avoid deep nesting.
* Prefer early returns (guard clauses) over nested if statements.
* A method should generally not exceed 3 levels of indentation.
* Methods with complex branching, nested loops, or nested try/except blocks should be split into smaller methods.

Loops
-----
* More than one loop in a method is a signal to review structure.
* Multiple simple loops may still be acceptable if they belong to a single coherent task.

Extraction Criteria
-------------------
Extract code into a separate method if one or more of the following applies:
* The code can be given a clear, meaningful name.
* It isolates error handling.
* It separates parsing, validation, or transformation logic.
* It improves readability by making the caller read like a sequence of domain-level steps.

Method Length
-------------
* If a method is longer than 20 lines it should be condsidered to split it into smaller methods.
* If a method has more than 1 loops it should be condsidered to split it into smaller methods.
* Methods, which are only used by one other method and do only server limited technical purpose of the calling 
  method and are not used in other places, should be inlined.
* Inlined methods should be structured in this way: The first method should be a "run" method, which calls the other methods. The other methods should be private and only used by the "run" method.
* Sumbethods of concructors must not be inlined, but should be named like "init_<name>" and follow the main contructor in the order of execution.

Method Structure (Orchestration Pattern)
----------------------------------------
* Public methods should act as orchestrators, calling smaller private methods.
* The structure should read like a sequence of clearly named steps.
* Private methods should represent meaningful sub-operations of the algorithm.

Typing and tooling
------------------

* Use type hints where they clarify interfaces; match the style of the repository you edit.
* Formatting and linting: follow the checks configured in that repository’s CI when present.

Testing
-------

* New features should include tests where practical.
* Bug fixes should include a regression test when feasible.
* Run the relevant test suite (e.g. ``pytest``) before opening a pull request.

Code Coverage
-------------
* Code coverage is a measure of the proportion of code that is executed by tests.
* The current goal is to achieve the following coverage for the codebase.

.. list-table::
   :widths: 40 20
   :header-rows: 1

   * - Part of Synarius
     - Coverage to achieve
   * - synarius_core
     - 75%
   * - synarius_studio
     - 40%
   * - synarius_dataviewer
     - 10%
   * - synarius_parawiz
     - 10%
   * - synariustools
     - 15%

* These percentages will be adopted over time.

Pull requests
-------------

* Keep changes focused and reviewable.
* Describe what changed and why; reference issues when applicable.
* Ensure CI passes for the target repository.

Repository-specific notes
-------------------------

* **synarius-core:** run tests via ``pytest`` from the core package layout.
* **synarius-apps / synarius-studio:** follow each repo’s ``CONTRIBUTING.md`` for additional hints (e.g. Studio icon conventions).

For legal and process details (e.g. CLA), see each repository’s ``CONTRIBUTING.md`` and ``CLA.md``.
