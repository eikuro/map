# Map

Shared base configuration for the [language maps](../pymap/README.md).

`map` owns the configuration that does not depend on the project language. Both
[PyMap](../pymap/README.md) and [JSMap](../jsmap/README.md) read it through
Porter, so one edit here reaches every repository in the organisation that
consumes a map: a map materialises these files as its own tracked outputs, and
every consumer inherits them from the map.

## Purposes

- Hold one source per shared file, so a change to the editor policy, the
  markdown lint profile, the ignore rules, the ignore attributes, or the licence
  is one commit rather than one commit per language.
- Keep the language maps focused on what is language-specific: their spelling
  configuration, package toolchain, and CI assets.
- Make drift detectable. Each map's `porter config --dry` reports a stale output,
  and a map that edits a shared file by hand is reported as drift on the next run.

## Capabilities

| File | Porter class in a map | Why it is shared |
| --- | --- | --- |
| [`LICENSE`](LICENSE) | copy | Organisation licence policy; it never depended on the component or its `private` flag |
| [`.editorconfig`](.editorconfig) | copy | Editor indentation and encoding policy for every language |
| [`.markdownlint.yaml`](.markdownlint.yaml) | copy | Markdown rules the organisation disables, not language rules |
| [`.gitattributes`](.gitattributes) | copy | Which generated files are binary, and the notebook diff and filter attributes |
| [`.gitignore`](.gitignore) | rendered body | The shared body above the `# Repo-specific Entries` boundary; a consumer keeps its own entries below it |

The `.gitignore` body is a **union** of what both languages need. Every rule is
inert in the language it does not belong to — measured across the workspace, no
tracked file in a Python repository matches a Node pattern and none in a Node
repository matches a Python pattern — and the shared whitelist re-includes the
paths a repository may commit on purpose. Two consequences are load-bearing:

- A repository that commits a file the body ignores re-includes it below its own
  `# Repo-specific Entries` boundary. `cspell.config.yaml` is ignored because a
  Python consumer receives it as a Porter link, while a Node consumer receives and
  commits a copy.
- `__pycache__` is re-ignored immediately after the `!__*/` whitelist, because
  the `_*` blacklist is what otherwise keeps bytecode out of every repository.

## Compatibility boundaries

- **The map owns no toolchain for its consumers.** It carries no CI and no
  package manifest, and it does not own the maps' spelling configuration:
  `cspell.config.yaml` is language-typed, so a union there would make each map
  load the other language's dictionary and accept its words. This repository's
  own configuration is the exception — it loads both dictionaries because its
  files quote both languages' terms (`*nbqa*.py` and `*.vsix`, `nbstripout`).
- **[`pyproject.toml`](pyproject.toml) is deliberately empty.** Porter derives a
  `pyproject.toml` merge for any repository that declares a `python:` section, so
  the base must provide the file even though the shared Python baseline lives in
  PyMap. See [DECISIONS.md](specs/DECISIONS.md), "PyMap consumption".
- **`.ci/` and `.infra/` stay in PyMap.** The pipelines are target-typed (ADO and
  Azure) rather than language-typed, so a non-Python consumer would not reuse
  them, and a second source would break the one-base model.
- **A map never consumes another map.** Consumers point at one base; the map is
  that base for the maps, and it is the only repository here that reads none.
- **This repository is the base, not a template.** It carries no `template.yaml`:
  a scaffold is created from a map, never from the shared base.

## Consumption

```text
map/.editorconfig
  -> pymap/.editorconfig        (porter config, copy)
      -> consumer/.editorconfig (Porter symlink or copy, per the consumer's manifest)
```

A consumer therefore never names `map`. It names its map, which names the base:

```yaml
# pymap/porter.yaml, jsmap/porter.yaml
config:
  base: ../map
  checkout: working-tree
```

## Assets

The hero mark is not created yet: there is no `chest/logo/map.png` master, and
inventing artwork for the base of every configuration would decide a visual
identity that belongs to the owner. The README carries no hero image until one is
provided; the `repo-init` handoff (`hero-logo.py`) writes both the chest master
and the repository copy in one step.
