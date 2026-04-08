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

Code style
----------

* Follow **PEP 8** and project conventions in existing code.
* Prefer small, focused functions and explicit behavior.
* Add docstrings where they help maintainers (public APIs, non-obvious behavior).

Typing and tooling
------------------

* Use type hints where they clarify interfaces; match the style of the repository you edit.
* Formatting and linting: follow the checks configured in that repository’s CI when present.

Testing
-------

* New features should include tests where practical.
* Bug fixes should include a regression test when feasible.
* Run the relevant test suite (e.g. ``pytest``) before opening a pull request.

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
