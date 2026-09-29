# Map

<!-- cspell:ignore pycache pyproject -->

Shared base configuration for the [language maps](../pymap/README.md).

`map` owns the configuration that does not depend on the project language. Both
[PyMap](../pymap/README.md) and [JSMap](../jsmap/README.md) read it through
Porter, so one edit here reaches every repository in the organisation that
consumes a map: a map materialises these files as its own tracked outputs, and
every consumer inherits them from the map.

## Purposes

- Hold one source per shared file, so a change to the editor policy, the
  markdown lint profile, the ignore rules, the ignore attributes, the licence, or
  the shared vocabulary is one commit rather than one commit per language.
- Keep the language maps focused on language-specific ignore and spelling rules,
  package toolchains, and CI assets.
- Make drift detectable. Each map's `porter config --dry` reports a stale output,
  and a map that edits a shared file by hand is reported as drift on the next run.

## Capabilities

| File | How a map uses it | Why it is shared |
| --- | --- | --- |
| [`LICENSE`](LICENSE) | copy | Organisation licence policy; it never depended on the component or its `private` flag |
| [`.editorconfig`](.editorconfig) | copy | Editor indentation and encoding policy for every language |
| [`.markdownlint.yaml`](.markdownlint.yaml) | copy | Markdown rules the organisation disables, not language rules |
| [`.gitattributes`](.gitattributes) | rendered body | Shared binary-file attributes; language maps add their lockfile attributes |
| [`.gitignore`](.gitignore) | rendered body | Common ignore rules; a language map extends the body before the consumer boundary |
| [`cspell.config.yaml`](cspell.config.yaml) | import | Shared language-neutral defaults, exclusions, and organisation vocabulary; maps add language coverage and dictionaries |
| [`dictionaries/`](dictionaries/) | mount | Shared vocabulary, one dictionary per domain; the base links it as `.cspell`, and a consumer's `.cspell` link resolves to it through its map, so the words a repository accepts are reviewed once |
| [`.pre-commit-config.yaml`](.pre-commit-config.yaml) | copy | Shared commit hooks, including the staged Betterleaks secret scan |

The `.gitignore` body contains only common rules. PyMap and JSMap add their
  language-specific patterns through `config[".gitignore"]` in `porter.yaml`; Porter
places these in the generated shared body, before its final exception rules.
Downstream consumers inherit that complete map body and retain local entries
below the `# Repo-specific Entries` boundary. The shared whitelist re-includes
paths a repository may commit on purpose. Two consequences are load-bearing:

Map's `.gitattributes` contains only language-neutral rules. PyMap adds `uv.lock`
and JSMap adds `pnpm-lock.yaml` through `config[".gitattributes"]` in their
manifests, so downstream consumers receive the complete map-owned attributes.
PyMap's notebook filters are declared separately and applied only to worker
components by its template.

- A repository that commits a file the body ignores re-includes it below its own
  `# Repo-specific Entries` boundary. `cspell.config.yaml` is ignored because a
  Python consumer receives it as a Porter link, while a Node consumer receives and
  commits a copy.
- Python's `__pycache__` rule stays in PyMap and follows the `!__*/` whitelist,
  because the `_*` blacklist is what otherwise keeps bytecode out.

## Compatibility boundaries

- **The map owns no toolchain for its consumers.** It carries no CI and no
  package manifest. [`cspell.config.yaml`](cspell.config.yaml) provides only
  language-neutral CSpell defaults and shared vocabulary; PyMap and JSMap add
  their own file coverage, built-in dictionaries, and language vocabularies.
- **The vocabulary is mounted, not materialised.** `.cspell` is the vocabulary
  path at every layer: this repository links [`dictionaries/`](dictionaries/) as
  `.cspell`, a map links its own `.cspell` here, and a consumer links its
  `.cspell` to its map's, so no copy exists below the base.
  [`cspell.config.yaml`](cspell.config.yaml) names the dictionaries, so a word
  and the file that loads it change together.
- **PyMap owns its `pyproject.toml` baseline.** Porter derives a Python TOML
  overlay from PyMap's manifest and does not require the config base to carry a
  placeholder `pyproject.toml`. See [DECISIONS.md](specs/DECISIONS.md),
  "PyMap consumption".
- **`.ci/` and `.infra/` stay in PyMap.** The pipelines are target-typed (ADO and
  Azure) rather than language-typed, so a non-Python consumer would not reuse
  them, and a second source would break the one-base model.
- **A map never consumes another map.** Consumers point at one base; the map is
  that base for the maps, and it is the only repository here that reads none.
- **This repository is the base, not a template.** It carries no `template.yaml`:
  a scaffold is created from a map, never from the shared base.

## Betterleaks

The shared pre-commit hook calls the installed Betterleaks CLI directly. Install
it once per machine with Homebrew, then verify the executable and version:

```sh
brew install betterleaks
command -v betterleaks
betterleaks --version
```

The hook was validated with Betterleaks 1.6.1. Upgrade deliberately and rerun
the hook check when changing the installed version.

The hook scans staged Git changes with redacted findings. To run the same scan
manually:

```sh
betterleaks git --pre-commit --redact --staged --log-level warn
```

Upgrade with `brew upgrade betterleaks`; remove it with
`brew uninstall betterleaks`. Repositories using this shared hook require the
CLI on `PATH` for every `pre-commit` run that executes the hook, including CI.

## Consumption

```text
map/.gitignore common body
  -> pymap/.gitignore + Python rules (porter config)
      -> Python consumer .gitignore (Porter template sync or copy)
  -> jsmap/.gitignore + JavaScript rules (porter config)
      -> JavaScript consumer .gitignore (Porter template sync or copy)
```

The vocabulary travels as a mount rather than a rendered file, so it keeps one
layer while the rest of the base is restated per map:

```text
map/dictionaries (linked as map/.cspell)
  -> pymap/.cspell link
      -> Python consumer .cspell link
  -> jsmap/.cspell link
      -> JavaScript consumer .cspell link
```

A consumer therefore never names `map`. It names its map, which names the base:

```yaml
# pymap/porter.yaml, jsmap/porter.yaml
config:
  base: ../map
  checkout: working-tree
  .gitignore: |-
    # language-specific patterns
```

## Assets

The hero mark is not created yet: there is no `chest/logo/map.png` master, and
inventing artwork for the base of every configuration would decide a visual
identity that belongs to the owner. The README carries no hero image until one is
provided; the `repo-init` handoff (`hero-logo.py`) writes both the chest master
and the repository copy in one step.
