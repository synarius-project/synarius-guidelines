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

Linking documentation to code
-----------------------------

Complex technical knowledge — specifications, protocols, non-obvious invariants, and
cross-module contracts — must be documented and explicitly linked from the source code that
implements or depends on it.  Code that is not connected to its governing rationale is
incomplete: a reader cannot verify correctness, and a maintainer cannot safely change anything
without knowing what rules must be preserved.

The principle applies at three levels of severity, each with the same core rule: **write it
down, name it, and link it from the code**.

.. rubric:: Level 1 — Formal specifications and standards

When a function, class, or module implements a published specification, a project-internal
protocol document, a file-format definition, or a non-trivial algorithm, the docstring must
state which document governs it and which section or paragraph applies.

.. code-block:: python

   def parse_dcm_record(data: bytes) -> DcmRecord:
       """Parse a single DCM record according to DCM2 spec §4.3 (key–value block).

       See ``docs/specifications/parameters_data_model_dcm2_cdfx_v0_5.rst``, section
       "Binary record layout".
       """

   class FmflExportVisitor:
       """Serialises the model graph to FMFL v0.1 format.

       Governing specification: ``docs/specifications/fmf_fmfl/fmfl_v0_1.rst``.
       Normative summary: ``docs/specifications/fmf_fmfl/normative_summaries_v0_1.rst``.
       """

The reference must be precise enough that a reader unfamiliar with the specification can find
the relevant passage without searching.  A bare *"implements the spec"* is not sufficient.

.. rubric:: Level 2 — Non-obvious internal invariants

When the correctness of a piece of code depends on a non-obvious constraint — an ordering
requirement, a coordinate convention, a data representation choice, a performance assumption —
that constraint must be named and explained in the docstring.  *Non-obvious* means: a
competent developer reading only the function body would not be able to reconstruct the
constraint from the code alone.

.. code-block:: python

   def _set_orthogonal_bends(self, value):
       """Set bends from absolute scene coordinates; stores them source-pin-relative.

       Storage format: even indices are x-offsets from the source-pin x, odd indices
       are y-offsets from the source-pin y.  The trailing y-coordinate of an even-length
       list is always stripped: routing derives it from the target pin's y, so storing
       it explicitly causes visual artefacts when placement is imprecise.

       See ``synarius-studio/docs/developer/connector_rendering.rst``,
       section "Speicherformat orthogonal_bends".
       """

For non-obvious invariants that span multiple functions in the same module, a module-level
docstring or an in-file section comment is the appropriate place, supplemented by a reference
in each participating function.

.. rubric:: Level 3 — Cross-module and cross-repository implicit contracts

Some behaviors require two independently maintained locations — often in different repositories
— to agree on the same value or formula.  No compile-time check can catch disagreements; only
a runtime integration test or a careful developer will notice.  These **implicit contracts**
are the most dangerous form of undocumented knowledge.

An implicit contract exists whenever:

* two modules independently compute the same quantity (e.g. a pixel dimension, a byte
  offset, a protocol field width), *or*
* one module stores data using a convention that another module must reverse correctly, *or*
* a module in repo A silently relies on an assumption that is only enforced in repo B.

For implicit contracts, apply all three rules together:

**Name it.** Every affected function or class on *both* sides of the boundary carries a
docstring that states the contract by name, identifies the counterpart, and describes the
symptom of a violation.

.. code-block:: python

   # Good
   def elementary_lib_block_pin_diagram_xy(inst, pin_name):
       """Pin attachment point in scene coordinates.

       Must reproduce the exact pixel position that ``FmuBlockItem.connection_point()``
       returns in synarius-studio.  A discrepancy causes connector bends to drift by the
       difference on every drag-release cycle.

       See ``synarius-studio/docs/developer/connector_rendering.rst``,
       section "Die Dual-Path-Invariante".
       """

   # Bad — states that matching is required but gives no context
   def elementary_lib_block_pin_diagram_xy(inst, pin_name):
       """Must match the studio layout."""

**Document it.** Create a developer documentation page that: (a) states the invariant
precisely; (b) explains why a violation causes a bug; (c) lists a **checklist** of every code
location that must be updated when the contract changes; (d) describes how to verify
correctness (test, formula, manual check).  Reference this page from every participating
source location with a consistent phrase: *"See <path/to/doc>, section '<Name>'."*

**Test it.** Add a unit test that asserts the contract holds.  If a headless test is not
feasible (e.g. requires a running Qt application), document that gap and describe what manual
verification looks like.

.. rubric:: When to write the document first

For Level 1, the governing specification already exists — reference it immediately when
writing the implementation.  For Levels 2 and 3, write the document *before* implementing or
modifying the affected code when the relationship is complex enough that a reader could not
derive it from the code alone.  A one-sentence docstring is often sufficient for simple
invariants; a separate ``docs/developer/`` page is appropriate when the constraint spans
multiple functions, files, or repositories.

.. rubric:: Rationale

The connector-bend drift bug (fixed 2026-04) is a concrete example of the cost of skipping
these rules.  Two geometry computations — one in ``synarius-core``, one in ``synarius-studio``
— had to agree on block widths in pixels.  The contract was invisible in both codebases.
Introducing three new block types violated it silently.  The resulting bug — a 37 px
horizontal jump after every drag-release — was reproducible only in the running application,
and the root cause was spread across two repositories, four files, and one approximation
function.  Explicit naming, a governing document, cross-references, and a checklist would
have prevented the regression.

AI assistant configuration
--------------------------

Two files at the **workspace root** (the directory that contains all Synarius repositories
as subdirectories, e.g. ``~/Programmierung/Synarius/``) instruct AI coding assistants to
follow these guidelines.  They are intentionally kept outside any individual repository
because they govern the entire multi-repo workspace.

.. list-table::
   :header-rows: 1
   :widths: 25 75

   * - File
     - Tool
   * - ``CLAUDE.md``
     - Claude Code (Anthropic CLI / VS Code extension): read automatically from the
       working directory and all parent directories on session start.
   * - ``.cursorrules``
     - Cursor: read automatically from the workspace root on project open.

Both files contain the same instruction and should be kept in sync:

.. code-block:: markdown

   # Synarius — Instructions for <Tool>

   ## Coding guidelines

   All code written or modified in this workspace must conform to the Synarius programming
   guidelines:

   **`synarius-guidelines/docs/programming_guidelines.rst`**

   Read this file before making non-trivial changes.  Pay particular attention to:

   - **Linking documentation to code** — specifications, non-obvious invariants, and
     cross-repository implicit contracts must be named in docstrings and linked to the
     governing document.  This includes the rule that every implementation of a formal
     specification (protocol, file format, algorithm) must reference the relevant section
     of that specification in its docstring.
   - **Method structure** — orchestration pattern, extraction criteria, length limits.
   - **Repository boundaries** — simulation logic belongs in `synarius-core`; Studio and
     Apps handle UI and integration only.

When these guidelines are updated in ways that affect what AI assistants should prioritise,
update the highlighted bullet points in both files accordingly.  The files themselves stay
minimal: they point to this document as the single source of truth rather than duplicating
rules.

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
