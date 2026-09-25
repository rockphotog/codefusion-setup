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
- Both parameter files are strict JSON. Keep comments out of them so they can
  be parsed by CodeFusion and other JSON tools.

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

## GitHub Actions: Used and Ignored Settings

The standard workflows, `run-CodeFusion.yaml` and `run-CodeFusion-QA.yaml`, use
the standard configuration. The separate `run-CodeFusion-Experimental.yaml`
workflow uses the experimental configuration. All workflows otherwise process
files in the same way:

1. The standard workflows read `parameters.json`; the experimental workflow
  reads `parameters-experimental.json`.
2. For each file under `input/`, the selected file is copied to the temporary
  Docker configuration directory as `parameters.json` and its `inputFile` is
  replaced with the container path `/input/<filename>`.
3. The resulting configuration is passed to CodeFusion in the Docker image.
4. The input and configuration directories are mounted at `/input` and
  `/app/CodeFusionFiles`; result files are copied from `/input` into
  `output/`.

The action-specific behavior of each setting is:

| Setting | GitHub Actions behavior |
| --- | --- |
| `ui` | Used. Keep `false`; the workflow runs CodeFusion non-interactively. |
| `inputFile` | Used, but the checked-in value is always replaced for each input file. |
| `columnNo` | Used by CodeFusion. |
| `fileContainsHeader` | Used by CodeFusion. |
| `version` | Used by CodeFusion to select classification data. |
| `source` | Used by CodeFusion to select the classification or Foundation. |
| `subtreeFilter` | Passed to CodeFusion. The repository value is empty, so no project-specific subtree is selected. |
| `includeScoreInOutput` | Used by CodeFusion. |
| `matchThreshold` | Used by CodeFusion. |
| `mappingMode` | Used by CodeFusion. Mapping-specific settings only have an effect when this is `true`. |
| `idColumnNo` | Read only when `mappingMode` is `true`; otherwise inactive. |
| `termTypeColumnNo` | Read only for mapping output when `mappingMode` is `true`; otherwise inactive. |
| `useFreePostcoordinationMatching` | Used by CodeFusion. |
| `exitWhenFinished` | Used. Keep `true` so the container can finish without keyboard input. |
| `codeFusionFilesFolder` | Not useful in these runs. It applies to the Web UI file browser, while Actions sets `ui` to `false`; the workflow mount path is fixed independently at `/app/CodeFusionFiles`. |
| `language` | Used by CodeFusion. |

The workflow itself does not use `codeFusionFilesFolder` to choose files or
outputs. Changing that value does not change the Actions mounts. The
experimental workflow has its own classification cache key based on
`parameters-experimental.json`, so its cache is separate from the standard
workflows.

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

```json
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
external concept. Set `idColumnNo` to the column containing the shared external
concept identifier. WHO recommends using a linearization such as `MMS` for
mapping so its postcoordination rules can be applied.

### `idColumnNo`

Required when `mappingMode` is `true`. This 1-based column number contains the
identifier of the external terminology or classification. It is **not** the
term-type column. All rows with the same identifier are considered synonyms of
one external concept.

### `termTypeColumnNo`

Optional 1-based column containing labels such as `Title` or `Synonym`. When
provided in mapping mode, the term type is included in the mapping output. It
does not identify which rows belong to the same concept; use `idColumnNo` for
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

```json
"codeFusionFilesFolder": "/Users/example/CodeFusionFiles"
```

## Recommended Configurations

### Ordinary matching

Use this for one phrase per row:

```json
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

```json
{
  "ui": false,
  "inputFile": "input/external-terms.xlsx",
  "columnNo": 3,
  "fileContainsHeader": true,
  "version": "2026",
  "source": "MMS",
  "language": "en",
  "mappingMode": true,
  "idColumnNo": 1,
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
2. Confirm `columnNo`, `fileContainsHeader`, and, for mappings, `idColumnNo`.
3. Choose a classification `version`, `source`, and `language`.
4. Keep `ui: false` and `exitWhenFinished: true` for GitHub Actions.
5. Run the workflow and inspect every generated result in the pull request.
6. Merge the result only after manual review of non-`GoodMatch` rows.

## References

- [WHO CodeFusion overview](https://icd.who.int/docs/codefusion/en/)
- [WHO CodeFusion User Guide](https://icd.who.int/docs/codefusion/en/user-guide/)
- [Running CodeFusion with Docker](https://icd.who.int/docs/codefusion/en/docker/)
- [WHO CodeFusion release notes](https://icd.who.int/docs/codefusion/en/release-notes/)