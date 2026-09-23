# CodeFusion workflow

This repository runs the WHO CodeFusion command-line tool in Docker. It matches
medical codes and translated medical terminology against ICD and other WHOFIC
classifications.

The workflow is manually triggered. It reads files from `input/`, applies the
settings in `parameters.json`, writes generated files to `output/`, and opens a
pull request for manual quality assurance.

## Repository layout

- `input/` contains the source `.txt` and `.xlsx` files.
- `output/` contains generated CodeFusion results after an output pull request
	is merged.
- `parameters.json` contains the shared CodeFusion command-line settings.
- `.github/workflows/run-CodeFusion.yaml` runs CodeFusion and creates the output
	pull request.

The repository does not include a native CodeFusion executable. The workflow
uses the versioned `whoicd/codefusion:1.1.1` Docker image.

## Prepare input

1. Add one or more `.txt` or `.xlsx` files under `input/`.
2. Commit and push the input files to the repository's default branch.
3. Update `parameters.json` when different matching settings are required.

Subdirectories under `input/` are supported. The workflow preserves that
structure under `output/`.

CodeFusion expects tab-separated data in `.txt` files. Set `columnNo` and
`fileContainsHeader` in `parameters.json` to match the input structure.

An `.xlsx` input produces an Excel output. A `.txt` input produces both a
tab-separated text output and an Excel output. CodeFusion adds `-checked` to
standard output names. Mapping mode also produces `-mapping` files.

## Configure CodeFusion

Edit `parameters.json` to control the classification version, source, language,
matching mode, threshold, and other CodeFusion options.

When `mappingMode` is enabled, set `idColumn` to the input column containing the
external terminology identifier. `termTypeColumnNo` is optional. WHO recommends
using a linearization such as `MMS` for mappings so postcoordination rules can be
applied; `foundation` output does not include postcoordination.

The workflow overrides only `inputFile`, setting it to the current file under
the container's `/input` directory. Keep these settings enabled for automation:

```jsonc
"ui": false,
"exitWhenFinished": true
```

`ui: false` selects the command-line tool. `exitWhenFinished: true` allows the
container and GitHub Actions job to finish without interactive input.

See the [CodeFusion documentation](https://icd.who.int/docs/codefusion/en/) for
all available parameters and input requirements.

## Run the workflow

1. Open the repository's **Actions** tab on GitHub.
2. Select **Run CodeFusion**.
3. Select **Run workflow**.
4. Wait for the `codefusion` job to complete.

The workflow processes every supported file in `input/`. It also uploads the
results as the `codefusion-results` workflow artifact.

## Review the output

After a successful run, the workflow creates and pushes a branch named like
`output-20260923-143012-123456789` and opens a pull request against the default
branch. The pull request contains a manual QA checklist.

Before merging:

1. Compare each generated file under `output/` with its source under `input/`.
2. Review results not marked `GoodMatch`.
3. Confirm the selected codes and translated terminology are correct.
4. Merge the pull request only after completing manual QA.

Each run replaces the working `output/` directory with newly generated results.
Merging the pull request therefore makes that run's output the current reviewed
result set.

## Repository settings

The workflow requires permission to push its generated branch and open a pull
request. In GitHub, open **Settings > Actions > General**, select **Read and
write permissions**, and enable **Allow GitHub Actions to create and approve
pull requests**.

## Data scope

Use this repository only for medical codes and translations. Do not add personal
health information or other identifying patient data to input files, output
files, workflow logs, or artifacts.
