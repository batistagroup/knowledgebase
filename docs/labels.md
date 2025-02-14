# Labels Guide

This document describes the labels used in our knowledgebase repository and their corresponding commit prefixes.

## Labels

### Core Science

| Label                | Commit Prefix(es) |
| -------------------- | ----------------- |
| `quantum-mechanics`  | `quantum:`, `qm:` |
| `quantum-computing`  | `qc:`             |
| `quantum-dynamics`   | `qd:`             |
| `molecular-dynamics` | `md:`             |
| `ai-ml`              | `ai:`, `ml:`      |

### Programming

| Label    | Commit Prefix(es) |
| -------- | ----------------- |
| `git`    | `git:`            |
| `python` | `python:`, `py:`  |

### Educational

| Label       | Commit Prefix(es) |
| ----------- | ----------------- |
| `tutorials` | `tutorial:`       |
| `lectures`  | `lecture:`        |

### Publications

| Label          | Commit Prefix(es) |
| -------------- | ----------------- |
| `publications` | `pub:`            |

### Maintenance

| Label         | Commit Prefix(es) |
| ------------- | ----------------- |
| `maintenance` | `maint:`          |

## Version Impact

Labels affect the version number of the next release as follows:

### Major Version Bump

- Manual label `major` for major updates

### Minor Version Bump

- All core science labels
- Educational content labels
- Publications label

### Patch Version Bump

- Maintenance label
- Default for unlabeled changes
