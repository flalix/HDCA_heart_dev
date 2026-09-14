# Review — `pyproject.toml`, HDCA_heart_dev

Reviewed against the repository layout at `github.com/rmauron/HDCA_heart_dev`
(top level: `code/`, `environments/`, `README.md` — no `src/`, no `__init__.py`,
no `LICENSE`).

**Headline:** this file is squidpy's `pyproject.toml` with `[project].name`
changed. 14 lines still reference squidpy, including the wheel packaging target,
the coverage source, four project URLs and five ruff per-file-ignore paths. As
written it does not build, and four separate config tables are silently inert.

Severity key — **B** blocks a build or install, **S** silently does nothing,
**M** metadata/correctness, **D** dependency hygiene.

---

## B1 — Wheel build fails: packaging target does not exist

```toml
build.targets.wheel.packages = [ "src/squidpy" ]
```

There is no `src/` directory in this repository and no importable package
anywhere in it. `hatchling` will abort with "Unable to determine which files to
ship". `[project].name` also implies an import package that does not exist.

Decide which of two shapes you want, because they need different files:

- **Analysis repository** (matches the repo as it stands — a script collection
  run across several conda environments). Drop `[build-system]` and `[project]`
  entirely; keep `[dependency-groups]` and `[tool.pixi]`. PEP 735 groups are
  valid without a `[project]` table, so `pixi` still builds every environment.
  This is what I've written in the corrected file.
- **Installable package.** Create `src/hdca_heart_dev/__init__.py`, move shared
  helpers out of `code/` into it, and set
  `build.targets.wheel.packages = ["src/hdca_heart_dev"]`.

Until one of those is true, `pip install .` and `pixi install` both fail on the
editable self-reference below.

## B2 — Editable self-install points at squidpy

```toml
pypi-dependencies.squidpy = { path = ".", editable = true }
```

This tells pixi that the current directory *is* squidpy and installs it editable
under that name. Two consequences: the real squidpy from conda-forge is
shadowed, and the install fails for the same reason as B1. If you want squidpy
as a dependency, declare it as one (`dependencies.squidpy = ">=1.6"` under
`[tool.pixi]`); the analysis in this project uses `squidpy.gr` and `squidpy.pl`
and currently nothing pulls it in at all.

## S1 — The entire pytest configuration is ignored

```toml
[tool.pytest]      # ← wrong table
```

pytest reads `[tool.pytest.ini_options]`. Under `[tool.pytest]` every key here
(`addopts`, `filterwarnings`, `markers`, `testpaths`, `python_files`) is
discarded without warning. Practical effect: `docs/` is not ignored,
`NumbaPerformanceWarning` is not escalated to an error, and `internet`/`gpu`
markers are unregistered.

Two notes for when you rename it. `strict = true` is not a pytest ini option —
the real options are `--strict-markers` and `--strict-config`, so after the
rename this key raises an error under `--strict-config`. And in
`filterwarnings`, later entries win; keep `error::numba.NumbaPerformanceWarning`
*after* the broad `ignore::UserWarning` or the escalation can be masked.

## S2 — setuptools tables are dead config

```toml
[tool.setuptools]        # backend is hatchling
[tool.setuptools_scm]    # versioning is hatch-vcs
```

`package-dir = {"" = "src"}` and `include-package-data` are setuptools-only keys
and hatchling ignores them — which is part of why B1 is not caught earlier. The
`setuptools_scm` table is likewise superseded by
`[tool.hatch].version.source = "vcs"`. Delete both.

## S3 — Coverage measures nothing

```toml
run.source = [ "squidpy" ]
paths.source = [ "squidpy", "*/site-packages/squidpy" ]
```

Coverage will report 0% for this project, or error with "No data was collected".

## S4 — `ban-relative-imports` is configured but not enforced

```toml
lint.flake8-tidy-imports.ban-relative-imports = "all"
```

The comment above it says "Disallow all relative imports", but `TID` is not in
`lint.select` (`B, BLE, C4, E, F, I, UP, W`), so the rule never fires. Add
`"TID"` to `select` if you want the ban; otherwise remove the setting and the
comment, which currently asserts a guarantee you don't have.

## S5 — Eighteen no-op docstring ignores

`D100`, `D104`, `D105`, `D107`, `D411` in `lint.ignore`, plus `D`-codes in seven
`per-file-ignores` entries, all reference pydocstyle rules. `D` is not selected,
so none of them do anything. Either add `"D"` to `select` (a real change — it
will surface a large backlog of missing docstrings across `code/`) or drop the
ignores. Keeping them suggests docstring linting is active when it isn't.

## S6 — per-file-ignores target paths that don't exist

`src/squidpy/_constants/_constants.py`, `src/squidpy/_constants/_pkg_constants.py`,
`src/squidpy/pl/_ligrec.py`, `.scripts/ci/download_data.py` — none are in this
repo. The `tests/*` and `docs/*` entries also point at directories that don't
exist yet.

## M1 — Missing `authors`, `license`, `license-files`

For a published atlas with a Zenodo DOI these are the fields people need in
order to cite and reuse the code, and PyPI will not display attribution without
them. There is no `LICENSE` file in the repository either — worth adding
alongside (BSD-3-Clause matches the scverse ecosystem; MIT is also common for
analysis code).

## M2 — All four URLs point to squidpy

`Bug Tracker`, `Documentation`, `Home-page`, `Source` all resolve to
`scverse/squidpy`. Bug reports for this project would land on squidpy's tracker.

## M3 — `Typing :: Typed` requires a shipped `py.typed`

The classifier is a promise to type-checkers. Without `py.typed` inside the
package, downstream `mypy` silently ignores your annotations. Drop the
classifier or ship the marker.

## M4 — `Development Status :: 5 - Production/Stable`

A judgment call, but for a paper-companion analysis repository with no tests and
no releases, `4 - Beta` describes the state more accurately.

## M5 — Name normalization

`HDCA_heart_dev` is legal but normalizes (PEP 503) to `hdca-heart-dev`, which is
what pip, PyPI URLs and lockfiles will show. Declaring
`name = "hdca-heart-dev"` avoids the mismatch.

## D1 — `leidenalg` declared three times

Bare in `dependencies` (line 1 of the list), again as `leidenalg>=0.11.0` at the
end, and a third time in `optional-dependencies.leiden`. Keep one pinned entry.
With the runtime dependency in place, the `leiden` extra only adds
`spatialleiden` — rename it to reflect that, or fold `spatialleiden` into a
broader extra.

## D2 — `napari-spatialdata` is a hard runtime dependency

This pulls the full napari/Qt GUI stack into every install, including headless
CI and cluster jobs where it cannot even initialize. It is an interactive viewer,
not a pipeline component. Move it to an optional extra
(`optional-dependencies.viz`). Same argument applies to `centrosome` and
`cp-measure`, which are the CellProfiler featurization stack and are only needed
for image feature extraction — an `image` extra keeps the base install
installable on a login node.

## D3 — `numba>=0.56.4` against a Python 3.14 default environment

`environments.default` resolves `py314`, but numba added Python 3.14 support in 0.63.0 (released 8 December 2025). The declared floor is
roughly four years below what the default environment can actually use. The
solver will resolve upward, so this isn't a break — but the floor no longer
communicates anything true. Bump to `numba>=0.63` if 3.14 is the primary target.

## D4 — Several floors are implausibly old for the rest of the pin set

`scikit-learn>=0.24` (Dec 2020), `matplotlib>=3.3` (2020), `dask>=2021.2`,
`statsmodels>=0.12`, `networkx>=2.6`, `pillow>=8` sit next to `pandas>=2.1`,
`zarr>=3`, `xarray>=2024.10` and `imagecodecs>=2025.8.2`. No solve will ever
pick the old ones, so they give a false impression of the supported range. For a
reproducibility-sensitive project, floors that match what you actually tested
are more useful than inherited ones.

## D4b — `pyyaml` is the distribution; `yaml` is the import name

If you hand-edit this list, note that the PyPI distribution is **`pyyaml`** and
only the *import* is `yaml`. There is no package named `yaml` on PyPI, so
declaring it produces:

```
Because there are no versions of yaml and you require yaml,
we can conclude that your requirements are unsatisfiable.
```

The original file had this right (`pyyaml>=6`). Verified by resolving both forms:
`pyyaml>=6` resolves; bare `yaml` fails with exactly the message above.

## B3 — uv needs `requires-python`, and needs the platform set narrowed

Two problems that only appear under `uv` (pixi is unaffected, which is why they
can go unnoticed):

**No `requires-python`.** uv reads it from `[project]`. With no such table it
warns and falls back to the interpreter it happens to find:

```
warning: No `requires-python` value found in the workspace. Defaulting to `>=3.14`.
```

That default is *stricter* than this project's real floor of 3.12, and it silently
changes per machine — a 3.12 box and a 3.14 box resolve different graphs from the
same file, which defeats the point of a lockfile.

**Unconstrained platform resolution.** By default uv solves for every platform
including win32, so a package with no Windows wheel fails a split you never
intend to support:

```
No solution found when resolving dependencies for split (markers: sys_platform == 'win32')
```

`[tool.pixi].workspace.platforms` declares `linux-64`/`osx-arm64` but uv does not
read it. The uv-side equivalent is `[tool.uv].environments`.

Fix — a metadata-only `[project]` table plus a narrowed environment list, which
does *not* turn the repo into an importable package (no `[build-system]`, so
nothing is built):

```toml
[project]
name = "hdca-heart-dev"
version = "0.0.0"
requires-python = ">=3.12"
dependencies = [ ]

[tool.uv]
environments = [ "sys_platform == 'linux'", "sys_platform == 'darwin'" ]
default-groups = [ "analysis" ]
```

Verified: `uv lock` on the corrected file resolves 260 packages with no warning,
and all six groups (`analysis`, `viz`, `image`, `test`, `docs`, `dev`) resolve.

## D5 — `tifffile!=2022.4.22` and a misplaced comment

The single-version exclusion guards a 2022 bug that a modern floor
(`tifffile>=2023.1`) covers more clearly. Separately, the comment

```
# due to https://github.com/scikit-image/scikit-image/issues/6850 breaks rescale ufunc
```

sits between `scikit-image` and `scikit-learn`, reading as if it applies to
`scikit-learn`. Move it onto the `scikit-image` line.

## D6 — Asymmetric pixi environments

`dev-py312`, `dev-py314`, `docs-py312`, `docs-py314`, `test-py314` — but no
`test-py312`, so the older interpreter is never tested. Either add it or drop
`py312` from the matrix and raise `requires-python`.

---

## Verified as correct

- `dynamic = ["version"]` with `hatch-vcs` and `version.source = "vcs"` — consistent.
- `requires-python = ">=3.12"` matches the 3.12/3.13/3.14 classifiers.
- pixi resolves `dev`/`test`/`docs` correctly: pixi interprets PEP 735 dependency groups as features of the same name, so those `features` references are valid even without explicit `[tool.pixi.feature.*]` tables.
- `matplotlib<3.11` in the `test` group only, with the runtime pin left open — correct
  way to cap a dependency for image-comparison baselines.
- `lint.unfixable` including `F401` — sensible, stops ruff deleting re-exports.
- `isort.required-imports = ["from __future__ import annotations"]` with `I`
  selected — active and correct.

## Suggested order of work

1. B1, B2 and B3 — decide package vs. analysis repo, then add the metadata-only
   `[project]` table (`requires-python`) and `[tool.uv].environments` if you use uv.
   Nothing else installs reliably until then.
2. S1 — rename the pytest table (drop `strict`, reorder `filterwarnings`).
3. S2, S3, S6 — delete setuptools tables, fix coverage source, fix ignore paths.
4. M1, M2 — add `authors`/`license` + a `LICENSE` file, repoint URLs.
5. S4, S5 — decide on `TID` and `D`, then make `select` and `ignore` agree.
6. D1, D2 — de-duplicate `leidenalg`, move GUI/image stacks to extras.
7. D3–D6 — refresh floors, fix the comment placement, balance the test matrix.
