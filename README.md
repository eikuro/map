<!-- markdownlint-disable MD033 MD041 -->
<p align="center">
 <img alt="cspell" src="cspell.png" width="160">
</p>

<h3 align="center">cspell</h3>
<p align="center">Shared spelling dictionaries for Workspace repositories.</p>
<!-- markdownlint-enable MD033 MD041 -->

## Purposes

This README is the entry point to Workspace's shared cspell vocabulary.

The repository keeps approved terms separate from consumer configuration:
consumer repositories own their file coverage, language settings, and ignore
paths, while this repository owns the reusable dictionaries those checks load.

## Capabilities

Run `cspell --config cspell.config.yaml <files>` from a consumer repository to
load the mounted dictionaries below. Each dictionary owns one vocabulary
domain, and a consumer loads only the domains its sources need: the Python
baseline loads `aurora`, `finance`, `geospatial`, `platform`, and `python-lib`,
while the JavaScript baseline loads `aurora`, `js-lib`, and `platform`.

| Dictionary | Coverage | Path |
| --- | --- | --- |
| Aurora | Organisation, workspace component, and internal shorthand names | [`dictionaries/aurora.dic`](dictionaries/aurora.dic) |
| Finance | Trading, financial-data, and security-identifier vocabulary | [`dictionaries/finance.dic`](dictionaries/finance.dic) |
| Geospatial | Geographic datasets, raster formats, and remote-sensing vocabulary | [`dictionaries/geospatial.dic`](dictionaries/geospatial.dic) |
| JavaScript library | Node, TypeScript, and VS Code package vocabulary the built-in dictionaries do not cover | [`dictionaries/js-lib.dic`](dictionaries/js-lib.dic) |
| Platform | Operating systems, shells, editors, browsers, CLIs, hosting, and cross-language technical vocabulary | [`dictionaries/platform.dic`](dictionaries/platform.dic) |
| Python library | Python packages, APIs, test configuration, and data-visualisation vocabulary | [`dictionaries/python-lib.dic`](dictionaries/python-lib.dic) |

A word belongs to the dictionary that owns its meaning. The language
dictionaries (`js-lib`, `python-lib`) hold vocabulary that only one runtime
needs, `platform` holds vocabulary every repository shares, and `aurora` holds
names that only make sense inside this workspace. The domain dictionaries
(`finance`, `geospatial`) stay separate because only domain repositories need
them.

## Compatibility

- **PyMap** and its consumers reference `aurora`, `finance`, `geospatial`,
`platform`, and `python-lib` from their `cspell.config.yaml` files.
- **JSMap** and its consumers reference `aurora`, `js-lib`, and `platform`.
- **Porter** materialises `.cspell` into consumer repositories as the shared
dictionary mount.
- cspell owns vocabulary only; consumer repositories own file selection,
language configuration, ignore paths, and application code.

## Surface

- [`cspell.png`](cspell.png) is the repository identity asset.
- [`dictionaries/`](dictionaries/) contains the maintained word lists.
- Consumer repositories provide the `cspell.config.yaml` invocation surface.
