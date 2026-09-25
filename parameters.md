# Parameters Guide
# CodeFusion Parameters Guide

This guide explains how to configure the `parameters.json` and
`parameters-experimental.json` files in this repository. CodeFusion uses these
settings to match medical phrases to ICD-11 and other WHO Family of
International Classifications (WHOFIC) entities.

The authoritative WHO documentation is available in the [CodeFusion User
Guide](https://icd.who.int/docs/codefusion/en/user-guide/). The repository runs
CodeFusion through GitHub Actions and Docker; see [README.md](README.md) for
that workflow.

## Before You Start

- Input files must be Excel files (`.xlsx`) or tab-separated text files
  (`.txt`). A text file must use tabs between columns, not commas.
- Column numbers are **1-based**: the first column is `1`, the second is `2`,
  and so on.
- Do not put patient-identifying or other sensitive health information in the
  repository, results, logs, or workflow artifacts.
- The files in this repository use JSON with comments (JSONC). The comments
  are useful documentation and are accepted by the repository workflow. A
  strict JSON parser may require the comments to be removed.

## How Configuration Is Selected

CodeFusion can be run in Web UI mode, with command-line options, or with a
`parameters.json` file next to the executable. The settings in this repository
are intended for the last two cases and for the Docker workflow.

For the repository workflow:

- Keep `ui` set to `false` so the process runs non-interactively.
- Keep `exitWhenFinished` set to `true` so the Docker container and workflow can
  finish automatically.
- The workflow supplies `inputFile` for each file under `input/`; the empty
  value in the checked-in `parameters.json` is therefore intentional.
- Results are written under `output/` by the workflow. A normal run appends
  `-checked` to the input filename. Mapping mode also creates files containing
  `-mapping`.

When running CodeFusion directly, set `inputFile` to a file path. WHO requires
forward slashes in this value, including on Windows, for example
`C:/data/phrases.xlsx`.

## Parameters

### `ui`

Boolean. Starts the Web UI when `true`; use `false` for command-line,
parameters-file, and automated runs. The UI is available at
`http://localhost:8384` when started according to the WHO instructions.

### `inputFile`

String path to the input `.xlsx` or tab-separated `.txt` file. This is the only
required value when running CodeFusion directly. In this repository it is
overridden by the workflow, so normally leave it as `""`.

### `columnNo`

Integer identifying the column containing the phrase to match. It defaults to
`1`. For example, use `2` when the first column contains an external ID and the
second contains the medical term.

### `fileContainsHeader`

Boolean. When `true`, CodeFusion ignores the first row as a header. Set it to
`false` when the first row is data. This repository explicitly uses `false` in
`parameters.json`; do not rely on the WHO command-line default, which is
`true`, when creating a new configuration.

### `version`

String identifying the classification release, such as `"2026"`, `"2025"`,
`"2023"`, or `"2023-01"`. Use `"beta"` for the daily unreleased version
available through the WHOFIC maintenance platform. The requested release must
be available for the selected source; WHO documents `ICHI` as available only
with `beta`.

### `source`

Classification or foundation used for matching. Supported values are:

- `MMS`: ICD-11 Mortality and Morbidity Statistics linearization.
- `ICF`: International Classification of Functioning, Disability and Health.
- `ICHI`: International Classification of Health Interventions; WHO documents
  this for `beta` versions.
- `foundation`: the ICD-11 Foundation.

The default and recommended source for this repository is `MMS`. A linearization
such as `MMS` provides classification codes and postcoordination rules. A
`foundation` search returns Foundation URIs and does not provide
postcoordination combinations.

### `language`

Two-character language code for the classification language, for example
`"en"` for English or `"es"` for Spanish. The repository uses `"en"`.

### `subtreeFilter`

Optional comma-separated list of Foundation URIs. Restricts matching to the
specified entities and their descendants. For example:

```jsonc
"subtreeFilter": "http://id.who.int/icd/entity/1435254666,http://id.who.int/icd/entity/1630407678"
```

If omitted, CodeFusion uses source-specific defaults. For `MMS` and `ICF`, the
default searches without extension codes; `MMS` also excludes the Traditional
Medicine and V chapters. Use this option when a project must limit matching to
known chapters or domains.

### `includeScoreInOutput`

Boolean. Adds the similarity score to the output when `true`. Scores range from
`0` to `1`, with higher values indicating a better match. The standard
`parameters.json` leaves this disabled; the experimental file enables it.

### `matchThreshold`

Number between `0` and `1`. Only matches meeting the minimum score are included.
The WHO default is `0.43`. Raising the threshold generally reduces weak matches
but can increase `NoMatch` results; lowering it produces more candidates that
need human review.

### `mappingMode`

Boolean. When `true`, CodeFusion creates the normal checked output and an
additional mapping output. Mapping mode groups multiple input phrases that
belong to the same external concept, which lets synonyms contribute to one
mapping suggestion.

Use mapping mode when the input contains several labels or synonyms for each
external concept. Set `idColumn` to the column containing the shared external
concept identifier. WHO recommends using a linearization such as `MMS` for
mapping so its postcoordination rules can be applied.

### `idColumn`

Required when `mappingMode` is `true`. This 1-based column number contains the
identifier of the external terminology or classification. It is **not** the
term-type column. All rows with the same identifier are considered synonyms of
one external concept.

### `termTypeColumnNo`

Optional 1-based column containing labels such as `Title` or `Synonym`. When
provided in mapping mode, the term type is included in the mapping output. It
does not identify which rows belong to the same concept; use `idColumn` for
that.

### `useFreePostcoordinationMatching`

Boolean. The normal matching process uses postcoordination combinations
suggested by the classification. When `true`, CodeFusion also searches for
other possible postcoordination combinations. This can improve mapping results,
especially for precoordinated concepts, but it may produce more alternatives
and requires careful review. Postcoordination is not available when
`source` is `foundation`.

### `exitWhenFinished`

Boolean. When `true`, exits after processing; when `false`, waits for a key press.
Use `true` in CI, Docker, and other automated runs. The repository uses `true`.

### `codeFusionFilesFolder`

Optional folder used by the Web UI file browser. The WHO default is
`~/CodeFusionFiles` on macOS and Linux and `Documents/CodeFusionFiles` on
Windows. This setting does not select the input for the repository workflow.
Example:

```jsonc
"codeFusionFilesFolder": "/Users/example/CodeFusionFiles"
```

## Recommended Configurations

### Ordinary matching

Use this for one phrase per row:

```jsonc
{
  "ui": false,
  "inputFile": "input/phrases.txt",
  "columnNo": 1,
  "fileContainsHeader": false,
  "version": "2026",
  "source": "MMS",
  "includeScoreInOutput": true,
  "matchThreshold": 0.43,
  "mappingMode": false,
  "useFreePostcoordinationMatching": false,
  "exitWhenFinished": true,
  "language": "en"
}
```

### Mapping external terminology

Use this when multiple rows share an external concept ID:

```jsonc
{
  "ui": false,
  "inputFile": "input/external-terms.xlsx",
  "columnNo": 3,
  "fileContainsHeader": true,
  "version": "2026",
  "source": "MMS",
  "language": "en",
  "mappingMode": true,
  "idColumn": 1,
  "termTypeColumnNo": 2,
  "useFreePostcoordinationMatching": true,
  "includeScoreInOutput": true,
  "exitWhenFinished": true
}
```

## Understanding Results

For an Excel input, CodeFusion creates an Excel result. For a tab-separated
text input, it creates both a tab-separated result and an Excel result. The
result keeps the source columns and adds matching information, including:

- `MatchLevel`: such as `GoodMatch`, `MatchHasAdditionalWords`, `FlexiMatch`,
  `GoodMatchWithFreePostcoordination`, or `NoMatch`.
- `Score`: included when `includeScoreInOutput` is enabled.
- `MatchType`: classification relationship such as `Real` or a
  postcoordination combination; it is unavailable for `foundation` searches.
- `BestMatchPhrase`, `LinearizationURI`, `Code`, and `FoundationURI`.

Review all results that are not `GoodMatch` before accepting them. Automated
coding must be reviewed by a human coder, especially for lower-confidence
matches.

Mapping mode additionally produces `.txt` and `.xlsx` mapping files. These
contain mapping rows grouped by external ID, detail rows for the matched labels,
and fields such as `MappingMatchLevel`, `bestScore`, `LabelBeingMatched`,
`BestMatchingPhrase`, `MatchLevel`, `score`, `MatchType`, and `Code`.

## Repository Checklist

1. Put `.txt` or `.xlsx` files in `input/`.
2. Confirm `columnNo`, `fileContainsHeader`, and, for mappings, `idColumn`.
3. Choose a classification `version`, `source`, and `language`.
4. Keep `ui: false` and `exitWhenFinished: true` for GitHub Actions.
5. Run the workflow and inspect every generated result in the pull request.
6. Merge the result only after manual review of non-`GoodMatch` rows.

## References

- [WHO CodeFusion overview](https://icd.who.int/docs/codefusion/en/)
- [WHO CodeFusion User Guide](https://icd.who.int/docs/codefusion/en/user-guide/)
- [Running CodeFusion with Docker](https://icd.who.int/docs/codefusion/en/docker/)
- [WHO CodeFusion release notes](https://icd.who.int/docs/codefusion/en/release-notes/)