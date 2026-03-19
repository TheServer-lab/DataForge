# 🔩 DataForge

A **minimal, deterministic, LLM-friendly** line-based file manipulation DSL.

[![PyPI](https://img.shields.io/pypi/v/dfgscript?label=PyPI)](https://pypi.org/project/dfgscript/)
[![License: SOCL](https://img.shields.io/badge/license-SOCL--1.0-lightgrey)](https://github.com/TheServer-lab/Celes/blob/main/LICENSE)
[![Python 3.9+](https://img.shields.io/badge/python-3.9%2B-blue)](https://python.org)

```dfg
create-file "notes.txt"

1+Hello world
2+This is a test file
3+It has some content
4+End of file
```

---

## Why DataForge?

Scripts and LLMs need a way to create and edit files that is **safe by default**. Direct filesystem access is too powerful — a single bad write can corrupt a project silently.

DataForge gives them a small, composable set of line-level primitives instead:

- **Deterministic** — same script, same result, every time
- **Safe** — conditional deletes never destroy data on mismatch
- **Atomic** — writes go to a temp file first, then rename over the original
- **Readable** — a human or an LLM can understand any `.dfg` file at a glance

---

## Installation

```bash
pip install dfgscript
```

Requires Python 3.9+. No external dependencies.

---

## CLI

```bash
# Execute a script
dataforge script.dfg

# Preview without writing anything
dataforge script.dfg --dry-run

# Save a timestamped backup before writing
dataforge script.dfg --backup-dir .backups

# Module invocation
py -m dataforge script.dfg
```

All three entry points do the same thing: `dataforge`, `build`, `py -m dataforge`.

---

## DSL Reference

### File-level commands

| Command | Behaviour |
|---|---|
| `create-file "path"` | Create new file — error if it already exists |
| `replace-file "path"` | Overwrite unconditionally |
| `change-file "path"` | Patch existing file (creates if absent) |

### Line-level operations

| Operation | Meaning |
|---|---|
| `N+text` | Write or replace line N (expands file with empty lines if needed) |
| `N-"text"` | Delete line N only if content matches exactly — warns and skips on mismatch |
| `N>text` | Insert text after line N — shifts the rest of the file down |
| `$+text` | Append as new final line |

### Comments and blank lines

```dfg
# Lines starting with # are comments and are ignored
# Blank lines in a .dfg are also ignored
```

---

## Examples

### Create a file

```dfg
create-file "notes.txt"

1+Hello world
2+This is a test file
3+It has some content
4+End of file
```

### Edit a file

```dfg
change-file "notes.txt"

# Delete line 2 only if it still matches the original
2-"This is a test file"
2+This is an edited file

# Delete line 3 only if it matches
3-"It has some content"

# Insert a new line after line 1
1>Inserted line here

# Append to end
$+--- EOF ---
```

**Result:**

1. Hello world
2. Inserted line here
3. This is an edited file
4. End of file
5. --- EOF ---

### Replace a file

```dfg
replace-file "log.txt"

1+Log created
2+Operation successful
```

---

## Python API

```python
from dataforge import parse, run

source = open("edit.dfg").read()
script = parse(source)                        # → DfgScript

lines  = run(script, dry_run=True)            # preview only
lines  = run(script, backup_dir=".backups")   # write + backup
```

---

## CLI Flags

| Flag | Description |
|---|---|
| `--dry-run` / `--preview` | Show result without writing any files |
| `--backup-dir PATH` | Save a timestamped .bak copy before each write |
| `--log PATH` | Append log output to a file |
| `--verbose` / `-v` | Debug-level detail for every operation |
| `--version` | Print version and exit |

---

## Security

- Path traversal (`..`) is blocked by default
- No shell code is ever executed
- Atomic rename semantics prevent partial writes
- Transactional apply — original is only replaced on full success

---

## Project Structure

```
dataforge_pkg/
├── pyproject.toml          ← pip install .
├── README.md
├── README.celes
├── tests.py
└── dataforge/
    ├── __init__.py         ← public Python API
    ├── __main__.py         ← py -m dataforge
    ├── parser.py           ← .dfg → DfgScript AST
    ├── interpreter.py      ← executes the AST
    └── cli.py              ← argparse CLI
```

---

## License

[Server-Lab Open-Control License (SOCL) 1.0](https://github.com/TheServer-lab/Celes/blob/main/LICENSE) — Copyright © 2025 Sourasish Das.
