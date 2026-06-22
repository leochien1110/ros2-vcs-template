---
name: vcstool-workspace
description: Use when creating, editing, validating, importing, exporting, or troubleshooting ROS/ROS 2 vcstool .repos manifests and source checkouts.
---

# vcstool Workspace

Use `vcstool` for multi-repository ROS workspaces.

## Manifest Rules

- Keep `.repos` files at the workspace root.
- Import repositories into `src/`.
- Ask whether the user wants one manifest or multiple project-specific manifests when project count is ambiguous.
- For multiple projects, use names like `<project>.repos` and document shared import order.

Minimal manifest:

```yaml
repositories:
  local/path:
    type: git
    url: https://github.com/org/repo.git
    version: branch-or-tag
```

## Commands

```bash
mkdir -p src
vcs validate < <project>.repos
vcs import src < <project>.repos
cd src
vcs custom --git --args submodule update --init --recursive
cd ..
```

Update already-imported repositories:

```bash
vcs pull src
```

Export current checkouts:

```bash
vcs export src > <project>.repos
vcs export --exact src > <project>.repos
```

`vcs import` reads YAML from stdin. `vcs validate` also reads the manifest from stdin.
