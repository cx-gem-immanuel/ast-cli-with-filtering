# Functionality Updates

## `--file-filter-ext` Flag

A new scan flag that accepts an ordered, comma-separated list of Apache Ant-style glob patterns controlling which files and directories are included in or excluded from the scan.

### Convention (Ant Standard)

| Pattern | Behaviour |
|---|---|
| Bare pattern | **Include** entries that match |
| `!` prefix | **Exclude** entries that match |
| No rules | Everything included |
| Include rules present | Default is **excluded** (opt-in) |
| Exclude rules only | Default is **included** (opt-out) |

### Wildcards

| Wildcard | Meaning |
|---|---|
| `*` | Any characters within a single path segment |
| `**` | Any characters across path separators (any depth) |
| `?` | Exactly one character |
| `{a,b}` | Alternation — matches either `a` or `b` |

> **Note:** `{a,b}` alternation is provided by the underlying `doublestar` library and is not part of the core Ant specification.

### Ordering

Rules are evaluated in order; the **last matching rule wins**. This enables patterns such as:

```
**/*.java,!**/Test*.java
```
Include all Java files, then exclude test classes.

```
!**/test/**,**/test/fixtures/**
```
Exclude all test directories, then re-include fixture data.

### Behaviour Details

- **Directory precedence:** a directory is pruned and not descended into unless a later rule may re-include a descendant, in which case traversal continues with per-entry evaluation.
- **Implicit depth anchoring:** patterns without `/` match at any depth — `*.java` behaves as `**/*.java`.
- **Directory-only rules:** a trailing `/` restricts a rule to directories only — `test/` matches a directory named `test` but not a file named `test`.

### Examples

```
# Exclude-only: include everything except test files and node_modules
!**/*.test.js,!**/node_modules/**

# Include-only: include only Java source files
**/*.java

# Ant idiom: include Java, exclude test classes
**/*.java,!**/Test*.java

# Exclude tests but keep fixtures
!**/test/**,**/test/fixtures/**

# Exclude specific path
!**/CICD/**

# Exclude test files across all JS/TS variants
!**/*.{test,spec}.{js,ts,jsx,tsx}
```

---

## `.git` Selective Filtering

Previously the entire `.git` directory was included in the zip with no filtering applied. It now operates selectively.

### Always Included

The following entries are always written to the zip **unconditionally**, regardless of any filter — including explicit exclusion patterns that target them directly:

| Entry | Purpose |
|---|---|
| `.git/HEAD` | Current branch pointer |
| `.git/packed-refs` | Packed references |
| `.git/objects/` | Object store (entire subtree) |
| `.git/refs/` | References (entire subtree) |
| `.git/config` | Remote urls, branches config |

### Normal Filtering Applied

Everything else inside `.git` is subject to the same filter pipeline as the rest of the source tree.
