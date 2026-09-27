---
name: decisions
description: Review accepted base and ownership choices
type: spec
---

# Map Decisions

## DECIDED

### Extract a shared base instead of duplicating rules across the maps

Scope: `.`, `../pymap/porter.yaml`, `../jsmap/porter.yaml`
Type: `technical`

The maps' shared files differed by one to three lines each — `.editorconfig`'s
indent width and rule group, one `.markdownlint.yaml` rule, one lockfile line in
`.gitattributes`, the language examples and about ten rules in `.gitignore`, and
the whole of `LICENSE` — and every one of those formats is one Porter already
copies or merges. Each map therefore declares `config: {base: ../map,
checkout: working-tree}` and materialises these files as its own tracked outputs;
consumers keep one base, their map, and name nothing here.

No Porter change was needed: `copy` reproduces a file byte for byte, and the
`.gitignore` key's template rule already renders `base body + Repo-specific
Entries + the consumer's own tail`. The extraction rewrites two published
repositories (the maps), which is why it was deferred until a third shared
change landed.

Reopen if the maps' shared files diverge by a rule a copy cannot express, or if a
third map appears whose language needs a genuinely different shared body.

### The bare name `map`, not a `*map` prefix

Scope: `.`, `../charter/CONCEPTS.md`
Type: `convention`

The organisation convention names a shared configuration base with a
domain-specific prefix and the `map` suffix, because a prefix says *which domain*
a map serves. This repository is not a domain map: it is the base every domain
map reads, so a prefix would claim a domain it does not have and would read as a
sibling of `pymap` and `jsmap` rather than their base. The convention gains the
exception, and the seat row in `../charter/CONCEPTS.md` lists `map/` before the
domain maps.

### The shared `.editorconfig` sets the family policy, with Python at four

Scope: `.editorconfig`
Type: `convention`

One shared file can carry one default, so the union policy is measured rather
than inherited: `[*] indent_size = 2` is the modal width of the workspace's
tracked shell, CSS, HTML, YAML, JSON, D2 and markdown files, `[*.py]` stays at
four (PEP 8), and the four-wide `.ini`, `.just`, `justfile` and `Dockerfile`
family keeps four explicitly. PyMap's now-redundant
`[*.{yaml,yml,json,jsonc,geojson}] indent_size = 2` group and its `[*]` width of
four are both retired by the shared file. EditorConfig steers editors and format
tools on edit and never rewrites existing bytes, so no file changes shape
because of this decision.

### PolyForm Noncommercial 1.0.0 as the organisation licence

Scope: `LICENSE`
Type: `policy`

The workspace was split: PyMap, Butler and Speller carried PolyForm
Noncommercial 1.0.0, JSMap and Squire carried MIT, and twelve repositories
carried none — and the `private` flag never predicted which, since public PyMap
was PolyForm while private JSMap was MIT. The base now carries one licence for
the organisation, and it is the source-available one the public map already
published, so JSMap and Squire move MIT → PolyForm on their next
materialisation.

Reopen if the organisation publishes a component under a permissive licence
deliberately; the licence is then a per-component decision again and this file
stops being copied.

### Union the shared ignore files instead of a per-language variant

Scope: `.gitignore`, `.gitattributes`
Type: `technical`

Both shared ignore bodies reach both languages through their map, and Porter can
copy a file or render a body-plus-tail, so the affordable choice was a union body
rather than a per-language body. Each union rule is inert in the language it does
not belong to — verified across the workspace: no tracked file in a Python
repository matches a Node-only pattern, and none in a Node repository matches a
Python-only pattern — and the whitelist re-includes what a repository commits on
purpose. `.gitattributes` has no merge format in Porter at all, so a per-language
delta there would have needed a new Porter feature; the union needs none.

Two rules carry a consequence that a reader must know:

- `/cspell.config.yaml` stays in the body because a Python consumer receives it
  as a Porter link and the link must stay untracked. A Node consumer receives and
  commits a copy, so JSMap, Falconer and Quest re-include it below their own
  `# Repo-specific Entries` boundary.
- `/justfile` moved out of the shared body and into PyMap's own tail. The
  justfile is not a Porter output in any consumer, and every consumer commits it,
  so the shared ignore only forced a `git add -f` on each new scaffold.

### Keep the base free of toolchain configuration

Scope: `cspell.config.yaml`, `../pymap/porter.yaml`
Type: `architecture`

Spelling configuration stays per map. `files:`, `ignorePaths:` and the
`dictionaries:` list are language-typed — PyMap loads `python-lib`, JSMap loads
`js-lib` and the TypeScript built-in — so a union would make each map load the
other language's dictionary and accept its words as correct, which weakens the
gate it exists to provide. For the same reason `.ci/` and `.infra/` stay in
PyMap: they are ADO and Azure assets (container, function, and bicep) rather than
language assets, and a second source would break the one-base model.

Reopen if a shared dictionary need appears that is not language-typed, or when a
non-Python consumer needs a pipeline.

## DECIDED

### PyMap consumption

Scope: `../pymap/porter.yaml`
Type: `technical`

Both maps consume this base. JSMap materialises the five shared files with no
Porter change at all; PyMap needed one relaxation, and the shape of the problem
is worth keeping because it is not obvious from the manifest.

A `python:` section makes a component *Python*, and Porter then derives a
`pyproject.toml` merge from the base. PyMap declares one (`version`,
`influence`) because `porter python` requires a `type: config` component, and
that command is what keeps PyMap's Python-version targets authoritative. The
derived overlay used to remove the `notebook` dependency group from every
non-worker component (`_dependency_group_overlay`), and PyMap's
`pyproject.toml` is the baseline every Python consumer merges: a render on
2026-09-28 produced a file with **zero** occurrences of `notebook`, because
`deep_merge` pops a `None` overlay value. Every worker would have lost
`ipykernel`, `nbqa` and `nbstripout` on its next render, and the `nbstripout`
pre-commit hook would have failed for want of the tool. Declaring the group in
PyMap's manifest did not help (the overlay set it to `None` regardless), and
declaring PyMap a `worker` would have kept the group but `porter python` refuses
anything that is not a `config` component.

The resolution was the narrowest of the three candidates: a `config` component
now keeps the template's group (`declares_notebook_group` in Porter's model),
while the scaffold's notebook *directory* keeps following the narrower
worker-only rule, because a config component owns no notebook directory. Worker
manifests were left alone, so the dependency list stays in one place.

Two further defects surfaced while propagating, both fixed in Porter rather than
worked around here:

- A manifest's `.gitignore` entries were re-appended on every render whenever
  the managed symlink block had become unnecessary, because the suffix check
  then compared two different blocks. Every consumer with a `.gitignore:` key
  and a symlink grew a duplicated rule per run.
- A repo whose `.gitignore` had no `# Repo-specific Entries` heading had its
  whole body treated as repo entries and appended below the new boundary. Chest
  was in that state and now carries the heading.

`LICENSE` materialises in every consumer whose manifest declares it, so the
PolyForm text now reaches repositories that carried none (Miller, Smith, Saga)
and replaces MIT where a map had it (JSMap, Squire).
