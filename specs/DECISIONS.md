---
name: decisions
description: Review accepted base and ownership choices
type: spec
---

# Map Decisions

<!-- cspell:ignore ipykernel nbqa nbstripout pyproject -->

## DECIDED

### Extract a shared base instead of duplicating rules across the maps

Scope: `.`, `../pymap/porter.yaml`, `../jsmap/porter.yaml`
Type: `technical`

The maps' shared files differed by one to three lines each — `.editorconfig`'s
indent width and rule group, one `.markdownlint.yaml` rule, shared
`.gitattributes` rules, the language examples and about ten rules in
`.gitignore`, and the whole of `LICENSE` — and every one of those formats is
one Porter already copies or merges. Each map therefore declares `config: {base: ../map,
checkout: working-tree}` and materialises these files as its own tracked outputs;
consumers keep one base, their map, and name nothing here.

The maps materialise shared files as tracked outputs. Their `.gitignore` files
also extend the inherited body through `config..gitignore`, a Porter field that
keeps language additions before the consumer boundary while preserving each
repository's own tail. PyMap and JSMap similarly extend `.gitattributes` through
`config[".gitattributes"]`; Map retains only language-neutral attributes. The
extraction rewrites two published repositories (the
maps), which is why it was deferred until a third shared change landed.

Reopen if the maps' shared files diverge by a rule a copy cannot express, or if a
third map appears whose language needs a genuinely different shared body.

### Keep Markdownlint configuration common and unconditional

Scope: `.markdownlint.yaml`, `../pymap/porter.yaml`, `../jsmap/porter.yaml`
Type: `technical`

Markdown is present in nearly every repository, so both maps copy the same
Markdownlint rules without branching on `languages` in Porter. Reopen if a map
serves a repository family that does not use Markdown or needs incompatible
Markdown rules.

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

### Use four spaces as the shared EditorConfig default

Scope: `.editorconfig`
Type: `convention`

One common default keeps the shared file minimal. EditorConfig-aware editors and
formatters can use four spaces for future edits; for example, JSMap's Prettier
configuration does not set `tabWidth`, so its formatting may follow this value.
The setting never rewrites existing bytes. Reopen if a language map requires a
different default or its formatter checks reject four-space indentation.

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

### Keep language-specific ignore rules in each language map

Scope: `.gitignore`, `.gitattributes`, `../pymap/porter.yaml`, `../jsmap/porter.yaml`
Type: `technical`

Map owns only language-neutral ignore rules. PyMap and JSMap extend the inherited
`.gitignore` body through the nested `config..gitignore` manifest field; Porter
inserts those rules before the base's final exception rules and before the
`# Repo-specific Entries` boundary. Each map materialises the result as its own
tracked file, and downstream consumers inherit it while retaining their local
tail. Reopen if another map needs an incompatible insertion point or pattern
ordering that this contract cannot express.

### Keep lockfile attributes in their language maps

Scope: `.gitattributes`, `../pymap/porter.yaml`, `../pymap/template.yaml`,
`../jsmap/porter.yaml`, `../porter`
Type: `technical`

Map owns language-neutral Git attributes, including `.vscode/settings.json`.
PyMap adds `uv.lock binary`, and JSMap adds `pnpm-lock.yaml binary` through
`config[".gitattributes"]`. PyMap's notebook diff/filter rules are declared in
its template and apply only to worker components. Each map materializes the
completed shared file for downstream consumers. Reopen if Porter cannot express
a new map-owned attribute without replacing the shared template.

Two ownership details remain important:

- `/cspell.config.yaml` stays in the body because a Python consumer receives it
  as a Porter link and the link must stay untracked. A Node consumer receives and
  commits a copy, so JSMap, Falconer and Quest re-include it below their own
  `# Repo-specific Entries` boundary.
- `/justfile` moved out of the shared body and into PyMap's own tail. The
  justfile is not a Porter output in any consumer, and every consumer commits it,
  so the shared ignore only forced a `git add -f` on each new scaffold.

### Keep CSpell defaults shared and vocabulary language-specific

Scope: `cspell.config.yaml`, `../pymap/cspell.config.yaml`, `../jsmap/cspell.config.yaml`
Type: `architecture`

Map owns language-neutral CSpell defaults, exclusions, and shared organisation
vocabulary. PyMap and JSMap import that baseline, then declare their own file
coverage, built-in dictionaries, and language-specific organisation
dictionaries. This keeps a valid Python-only or JavaScript-only word from
weakening the other map's spelling gate. Reopen if a dictionary or file pattern
is genuinely shared and duplicated maintenance becomes material.

For the same reason `.ci/` and `.infra/` stay in PyMap: they are ADO and Azure
assets (container, function and bicep) rather than language assets, and a second
source would break the one-base model.

### PyMap consumption

Scope: `../pymap/porter.yaml`
Type: `technical`

Both maps consume this base. Porter now permits the derived Python TOML overlay
to start without a base `pyproject.toml`, so PyMap owns that baseline itself.

A `python:` section makes a component *Python*, and Porter then derives a
`pyproject.toml` merge. PyMap declares one (`version`,
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

### The vocabulary is a directory in the base, not a repository

Scope: `dictionaries/`, `.cspell`, `cspell.config.yaml`, `../pymap/porter.yaml`,
`../jsmap/porter.yaml`
Type: `technical`

The dictionaries were a component repository of their own (`eikuro/.cspell`)
that only repositories already naming a map consumed, while `cspell.config.yaml`
here was the only tracked file that reached them through a sibling path
(`../.cspell/dictionaries/`). The base every other repository reads was
therefore the one repository that read outside its own tree.

The vocabulary now lives at [`dictionaries/`](../dictionaries), flattened to one
`.dic` per domain, and the base links it as `.cspell` — the name every member
already uses — so the vocabulary path is `.cspell/<name>.dic` everywhere: this
repository's link reaches its own directory, a map links its `.cspell` to
`../map/.cspell`, and a consumer links its `.cspell` to its map's. Those links
are name-preserving, so `local: [.cspell]` expresses them and lets Porter
materialise the chain instead of it being hand-made.

History is carried from `eikuro/.cspell` as the second parent of the absorbing
commit, so its commits stay reachable through
`git log e460eb6^2 -- dictionaries/<name>.dic`; the source repository is deleted.
A path-limited log over
`.cspell/**` stops at the merge, because the absorbed paths are prefixed rather
than rewritten. The absorbed component manifest was not carried: this repository
already declares the identity, and its `private` flag stays the owner's decision.

Reopen if a domain map takes ownership of `finance` or `geospatial`, which would
move those dictionaries out of the base, or if a repository appears that needs
the vocabulary without naming a map.
