<!-- markdownlint-disable MD033 MD041 -->
<p align="center">
 <img alt="cspell" src="cspell.png" width="160">
</p>

<h3 align="center">cspell</h3>
<p align="center">Shared spelling dictionaries for Aurora repositories.</p>
<!-- markdownlint-enable MD033 MD041 -->

## Purposes

This README is the entry point to Aurora's shared cspell vocabulary.

The repository keeps approved terms separate from consumer configuration:
consumer repositories own their file coverage, language settings, and ignore
paths, while this repository owns the reusable dictionaries those checks load.

## Capabilities

Run `cspell --config cspell.config.yaml <files>` from a consumer repository to
load the mounted dictionaries below.

| Dictionary | Coverage | Path |
| --- | --- | --- |
| Finance | Trading, financial-data, and security-identifier vocabulary | [`dictionaries/finance.dic`](dictionaries/finance.dic) |
| Geospatial | Geographic datasets, raster formats, and remote-sensing vocabulary | [`dictionaries/geospatial.dic`](dictionaries/geospatial.dic) |
| Library | Python packages, APIs, and test/build configuration terms | [`dictionaries/lib.dic`](dictionaries/lib.dic) |
| Tooling | Agent, editor, process, and project-specific vocabulary | [`dictionaries/tooling.dic`](dictionaries/tooling.dic) |

## Compatibility

- **PyMap**, **JSMap**, and **Quest** reference these dictionaries from their
consumer `cspell.config.yaml` files.
- **Porter** materialises `.cspell` into consumer repositories as the shared
dictionary mount.
- cspell owns vocabulary only; consumer repositories own file selection,
language configuration, ignore paths, and application code.

## Surface

- [`cspell.png`](cspell.png) is the repository identity asset.
- [`dictionaries/`](dictionaries/) contains the maintained word lists.
- Consumer repositories provide the `cspell.config.yaml` invocation surface.
