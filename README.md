# 🔩 DataForge

> A minimal, deterministic, LLM-friendly, line-based file manipulation DSL.

[![PyPI version](https://img.shields.io/pypi/v/dfgscript)](https://pypi.org/project/dfgscript/)
[![Python](https://img.shields.io/badge/python-3.9%2B-blue)](https://python.org)
[![License](https://img.shields.io/badge/license-SOCL-lightgrey)](./LICENSE)

---

## Install

```bash
pip install dfgscript
```

Afterwards three entry points are available:

```bash
dataforge script.dfg          # canonical command
build     script.dfg          # alias matching the spec
py -m dataforge script.dfg    # module invocation
```

---

## Quick example

```dfg
create-file "notes.txt"
1+Hello world
2+This is a test file
3+It has some content
4+End of file
end-file
```

```bash
dataforge notes.dfg           # write the file
dataforge notes.dfg --dry-run # preview without writing
```

---

## Multi-file support (v0.3)

A single `.dfg` script can create and edit multiple files at once. Each block
is explicitly closed with `end-file`:

```dfg
create-file "notes.txt"
1+Hello
end-file

create-file "Hi.txt"
1+Hi
end-file
```

Rules:
- Every block must be closed with `end-file` before opening a new one
- A single block without `end-file` is still valid (v0.2 backwards compatibility)
- If a block fails fatally, execution stops and later blocks are not run

---

## DSL reference

### File-level commands

```
create-file "path"    # create new file — error if it already exists
replace-file "path"   # overwrite file unconditionally
change-file "path"    # patch existing file (creates if absent)
remove-file "path"    # delete a file — error if it does not exist
new-folder "path"     # create a folder — error if it already exists
set-folder "path"     # create a folder + parents silently (mkdir -p)
remove-folder "path"  # delete a folder and all its contents
end-file              # close the current block
```

### Line-level operations

```
N+<text>      # write/replace line N  (expands file with "" if N > length)
N-            # delete line N unconditionally (warns if N out of range)
N><text>      # insert <text> after line N  (shifts tail down)
$+<text>      # append as new final line
```

### Comments & blank lines

```dfg
# This is a comment — ignored by the parser
```

Blank lines in a `.dfg` are also ignored. To write an actual blank line into
a target file use `N+` with no text: `3+`

---

## Edit example

```dfg
change-file "notes.txt"

# Delete line 2 only if it still matches exactly
2-"This is a test file"
2+This is an edited file

# Delete line 3 only if it matches
3-"It has some content"

# Insert a new line after line 1
1>Inserted line here

# Append to end
$+--- EOF ---
end-file
```

**Result:**
1. `Hello world`
2. `Inserted line here`
3. `This is an edited file`
4. `End of file`
5. `--- EOF ---`

---

## Scaffold example

DataForge can bootstrap an entire project structure in one script:

```dfg
new-folder "myapp"
end-file

new-folder "myapp/src"
end-file

new-folder "myapp/tests"
end-file

create-file "myapp/src/main.py"
1+# entry point
end-file

create-file "myapp/README.md"
1+# myapp
end-file
```

```bash
dataforge scaffold.dfg
```

## Box variables

Declare a **box** to store a value and use `[name]` to interpolate it anywhere — in paths, line operations, even folder names.

```dfg
box name = "RICK"

create-file "[name].txt"
1+[name] is the name
end-file
```

Boxes can be used across multiple blocks in the same script:

```dfg
box project = "myapp"
box author  = "Sourasish"

new-folder "[project]"
end-file

new-folder "[project]/src"
end-file

create-file "[project]/README.md"
1+# [project]
2+Author: [author]
end-file
```

Rules:
- Declare with `box name = "value"` — values are always quoted strings
- Interpolate with `[name]` anywhere in paths or line content
- Re-declaring a box overrides its previous value
- Using an undeclared box is a **parse error**
- Inline comments after the value are allowed: `box name = "RICK" # the author`

## CLI flags

| Flag | Description |
|------|-------------|
| `--dry-run` / `--preview` | Show resulting file(s); do **not** write anything |
| `--backup-dir PATH` | Save a timestamped `.bak` copy before each write |
| `--log PATH` | Append log output to a file |
| `--verbose` / `-v` | Debug-level logging of every operation |
| `--version` | Print version and exit |

---

## Python API

```python
from dataforge import parse, run

# Multi-file — parse() returns a list of DfgScript blocks
scripts = parse(open("edit.dfg").read())
results = run(scripts, dry_run=True)   # → { "path": ["line", ...], ... }

# Single-file convenience
from dataforge import parse_one
script  = parse_one(open("single.dfg").read())
results = run(script, backup_dir=".backups")
```

---

## Security

- Path traversal (`..`) is blocked by default
- No shell code is ever executed
- Atomic rename semantics — original only replaced on full success
- Transactional apply — writes go to a temp file first, then rename over the original

---

## Project structure

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

## Changelog

### v0.3.4
- `N-` now has two forms:
  - `N-` — unconditional delete (great for scaffolds and scripts that own the file)
  - `N-"text"` — conditional delete (safe for LLMs editing existing files — errors if content has drifted)
- Both forms error if line N does not exist
- `[box]` interpolation works inside `N-"text"` conditions

### v0.3.3
- Added `box name = "value"` — declare a variable (box) anywhere in a script
- Added `[name]` interpolation — use box values in paths and all line operations
- Using an undeclared box is a parse error
- Inline comments on box declarations are supported
- DataForge is now a full scaffold template engine

### v0.3.2
- `N-` no longer requires a text match — it now deletes line N unconditionally
- `N-` warns and skips if line N is out of range
- Syntax simplified from `N-"text"` to just `N-`

### v0.3.4
- `N-` now has two forms:
  - `N-` — unconditional delete (great for scaffolds and scripts that own the file)
  - `N-"text"` — conditional delete (safe for LLMs editing existing files — errors if content has drifted)
- Both forms error if line N does not exist
- `[box]` interpolation works inside `N-"text"` conditions

### v0.3.3
- Added `box name = "value"` — declare a variable (box) anywhere in a script
- Added `[name]` interpolation — use box values in paths and all line operations
- Using an undeclared box is a parse error
- Inline comments on box declarations are supported
- DataForge is now a full scaffold template engine

### v0.3.2
- `N-` no longer requires a text argument — deletes line N unconditionally
- Out-of-range deletes warn and skip instead of raising a fatal error

### v0.3.4
- `N-` now has two forms:
  - `N-` — unconditional delete (great for scaffolds and scripts that own the file)
  - `N-"text"` — conditional delete (safe for LLMs editing existing files — errors if content has drifted)
- Both forms error if line N does not exist
- `[box]` interpolation works inside `N-"text"` conditions

### v0.3.3
- Added `box name = "value"` — declare a variable (box) anywhere in a script
- Added `[name]` interpolation — use box values in paths and all line operations
- Using an undeclared box is a parse error
- Inline comments on box declarations are supported
- DataForge is now a full scaffold template engine

### v0.3.2
- `N-` no longer requires quoted text — deletes line N unconditionally
- Out-of-range `N-` warns and skips instead of erroring

### v0.3.4
- `N-` now has two forms:
  - `N-` — unconditional delete (great for scaffolds and scripts that own the file)
  - `N-"text"` — conditional delete (safe for LLMs editing existing files — errors if content has drifted)
- Both forms error if line N does not exist
- `[box]` interpolation works inside `N-"text"` conditions

### v0.3.3
- Added `box name = "value"` — declare a variable (box) anywhere in a script
- Added `[name]` interpolation — use box values in paths and all line operations
- Using an undeclared box is a parse error
- Inline comments on box declarations are supported
- DataForge is now a full scaffold template engine

### v0.3.2
- Added `new-folder "path"` — create a folder; error if it already exists
- Added `set-folder "path"` — create a folder and all parents silently (`mkdir -p`)
- Added `remove-folder "path"` — delete a folder and all its contents
- Folder commands respect `--dry-run` and `--backup-dir`
- Line-level operations inside folder commands are a parse error
- DataForge is now a scaffold tool for LLMs, scripts, and project generation

### v0.3.1
- Added `remove-file "path"` — deletes a file; errors if it does not exist
- Respects `--backup-dir` and `--dry-run` like all other commands
- Line-level operations inside a `remove-file` block are a parse error

### v0.3.0
- Multi-file support — a single `.dfg` can now target multiple files
- New `end-file` terminator closes each block explicitly
- `parse()` now returns `list[DfgScript]` instead of a single `DfgScript`
- `parse_one()` added as a convenience wrapper for single-block scripts
- `run()` now accepts either a single `DfgScript` or a `list[DfgScript]`
- `run()` now returns `dict[path, list[str]]`
- Full backwards compatibility with v0.2 single-block `.dfg` files

### v0.2.0
- Initial release

---

## License

[Server-Lab Open-Control License (SOCL)](./LICENSE) — Copyright © 2025 Sourasish Das.
