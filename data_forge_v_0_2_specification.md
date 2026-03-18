# DataForge — v0.2 Specification

> A minimal, deterministic, LLM-friendly, line-based file manipulation DSL (`.dfg`).

---

## 1. Goals

- Provide a *safe* interface for scripts and LLMs to create and edit text files without direct arbitrary filesystem access.
- Be **simple**, **deterministic**, and **readable** for both humans and LLMs.
- Favor *safety-first* defaults (avoid silent destructive actions).
- Keep a small, composable set of line-level primitives that cover most edit use-cases.

---

## 2. Execution model

- A `.dfg` document is executed by a host tool using the CLI command:

```
build file.dfg
```

- Each `.dfg` currently targets **one file**. The target file is declared inline (see File-level operations).
- Execution is **strictly top-to-bottom**. The interpreter applies each operation sequentially.
- By default the interpreter performs a real run. Implementations **must** provide a dry-run/preview mode (example CLI flag `--dry-run`) that shows expected output without writing files.
- Implementations **should** attempt transactional semantics for `change-file` operations (apply to a temporary copy, then atomically replace the original on success). If atomic replace is not possible on a platform, the interpreter must at least preserve a backup copy before changes.

---

## 3. File-level operations (commands)

All commands are plain text lines at the top-level of the `.dfg`. Whitespace and blank lines are allowed for readability. Comments may be added with `#` as the first non-space character on a line.

### 3.1 `create-file "<path>"`
- Creates a **new** file at `<path>` and writes the lines that follow (until another file-level command or end-of-file) as the initial file contents.
- If the target file **already exists**, `create-file` **fails** and the interpreter must return an error code (non-zero). No write should occur.
- Use this when you want to ensure a file is only created if it does not exist.

### 3.2 `replace-file "<path>"`
- Unconditionally writes the provided lines into `<path>`, overwriting any existing content.
- Use this to forcefully replace file contents.

### 3.3 `change-file "<path>"`
- Applies a sequence of **line-level operations** to an existing file. If the file does not exist, the interpreter **should** create an empty file first and then apply operations (this keeps `N+` semantics consistent). Implementations may optionally return a warning if the file did not previously exist.
- `change-file` operations should be applied transactionally when possible (write to temp, validate, then atomically replace).

> **Note:** For v0.2, a `.dfg` should contain one file-level command at top-level. Future versions may support multiple target blocks.

---

## 4. Line-level operations (syntax)

Line operations are applied in the order they appear. Lines that are part of file content (for `create-file` / `replace-file`) SHOULD use the `N+` style for explicitness, but plain content lines (without `N+`) may be supported by implementations as an alternative for simple file creation blocks (see examples).

### 4.1 `N+<text>` — write/replace or create
- **Semantics:** Ensure line number `N` contains `<text>`.
  - If line `N` exists → replace its content with `<text>`.
  - If line `N` does not exist → expand the file with **empty lines** (`""`) up to line `N` and place `<text>` at line `N`.
- Behavior is deterministic and positional. Line numbers start at `1`.
- Example:
  - If file currently is `[
    "A",
    "B"
    ]` and operation is `3+Hello`, result → `["A","B","Hello"]` (line 3 created).

### 4.2 `N-"<text>"` — conditional delete (safe-delete)
- **Semantics:** Delete line `N` **only if** its content equals exactly `<text>`.
- If the line `N` exists but does not match `<text>`, the interpreter must **warn** and skip the delete (no change to the file).
- If the line `N` does not exist, the interpreter must **warn** and skip the delete.
- Example:
  - `2-"This is a test file"` removes line 2 only when an exact match occurs.

> **Important:** Matching is exact and **case-sensitive**. No trimming or normalization of whitespace should be performed during the match — the stored lines are raw strings. (Specifies empty line is `""`.)

### 4.3 `N>` — insert after
- **Syntax:** `N>Some text`
- **Semantics:** Insert `Some text` *after* line number `N`. All existing lines at positions `> N` are shifted down by one.
- If line `N` does not exist, the interpreter **expands with empty lines** up to `N` then inserts the new line after that (resulting in the same effect as an `N+` followed by shifting the remainder).

### 4.4 `$+<text>` — append to end
- **Semantics:** Append `<text>` to the end of the file as a new final line.
- Equivalent to writing to line `L+1` where `L` is current number of lines, but explicit and clearer.

### 4.5 Comments & blank lines
- Any line starting with `#` (after optional leading whitespace) is a comment and ignored.
- Blank lines in a `.dfg` are ignored for parsing; to create an actual empty line in a file via `create-file`/`replace-file`, use the `N+` syntax with an empty string or explicitly include a numbered empty line (e.g. `2+`).

---

## 5. Quoting, whitespace, and normalization

- File paths in file-level commands MUST be quoted using double-quotes: `"notes.txt"`.
- The `<text>` part of `N+<text>` and `N>text` is the raw string from after the operator until end-of-line.
- **Empty line definition:** an empty line is the empty string `""` (not a space). When the interpreter expands the file to reach a line number, it fills intermediate lines with `""`.
- **Matching for deletes** is exact — characters, whitespace and case all must match. Implementations may offer optional flags such as `--ignore-trailing-whitespace` but default behavior is exact match.
- Internally, the interpreter treats each stored line as the raw content **without** newline characters. (E.g., `"Hi"`, not `"Hi\n"`.)

---

## 6. Error handling and warnings

- `create-file` on existing file: **error** (non-zero exit). No writes occur.
- `replace-file`: overwrites without confirmation.
- `change-file`: if any unconditional fatal parse/runtime error occurs, interpreter should abort and return a non-zero exit code and restore original file (if possible).
- For non-fatal mismatches (e.g., `N-"text"` where content differs), interpreter **must** emit a warning and continue with the next operation.
- Implementations should provide logging of actions and warnings (console output and optional log file).

---

## 7. Examples

### 7.1 Create notes.txt
```dfg
create-file "notes.txt"

1+Hello world
2+This is a test file
3+It has some content
4+End of file
```

### 7.2 Change notes.txt (edit, delete, insert, append)
```dfg
change-file "notes.txt"

# Replace line 2 only if it matches the original text
2-"This is a test file"
2+This is an edited file

# Delete line 3 only if it matches
3-"It has some content"

# Insert after line 1
1>Inserted line here

# Append to end
$+--- EOF ---
```

**Expected `notes.txt` after change-file** (applied to the create-file example above):

1. `Hello world`
2. `Inserted line here`
3. `This is an edited file`
4. `End of file`
5. `--- EOF ---`

> Note: If any conditional delete (like `3-"It has some content"`) does not match, the delete is skipped and a warning emitted.

### 7.3 Replace file
```dfg
replace-file "log.txt"

1+Log created
2+Operation successful
```

---

## 8. CLI & implementation suggestions

- CLI: `build file.dfg [--dry-run] [--backup-dir path] [--log path] [--verbose]`
- Always create a backup copy (timestamped) before making in-place changes unless `--no-backup` is explicitly provided.
- Implement atomic replace using `rename` semantics where possible (write to temporary file, fsync, then rename over original).
- Provide a `--preview` / `--dry-run` mode that prints resulting file content without writing.

---

## 9. Security considerations

- The interpreter must validate target paths to avoid path traversal by default. For example, by default only allow relative paths that do not contain `..` or absolute drives. Provide an explicit `--allow-paths` flag for advanced use.
- Run the interpreter with least privilege; it should not execute arbitrary shell code.
- When used as a layer between an LLM and a filesystem, ensure the host process enforces limits (max file size, max number of lines, execution timeouts).

---

## 10. Future / vNext ideas

- Support multi-file `.dfg` (block-per-file) with explicit `[FILE "path"]` headers.
- Add pattern-based operations (regex-based delete/replace) behind explicit flags for power users.
- Add line ranges (e.g. `2..4-"text"`) for batch operations.
- Support metadata operations (permissions, timestamps).
- Integrate with an audit log format (signed operations) for untrusted LLM execution.

---

## 11. Glossary

- **Interpreter:** program that executes `.dfg` files.
- **Target file:** the file a `.dfg` block applies to (declared in the file-level command).
- **Empty line:** an entry equal to the empty string `""`.
- **Transactional apply:** apply all edits to a temporary copy and atomically replace the original on success.

---

## 12. Appendix — Quick reference (cheat sheet)

- `create-file "path"` — create new file; error if exists
- `replace-file "path"` — overwrite file unconditionally
- `change-file "path"` — apply line operations (creates file if missing)

Line ops:
- `N+Text` → write/replace (expand file with `""` if N missing)
- `N-"Text"` → delete line N only if exact match
- `N>Text` → insert after line N (shifts tail)
- `$+Text` → append at end

---

*End of DataForge v0.2 specification.*

