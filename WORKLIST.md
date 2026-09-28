# Worklist

## 1. Validate inputs and configuration before running CodeFusion

The workflows currently check only that at least one supported file exists. Add a preflight step that rejects malformed parameter JSON and checks each input against its selected configuration, including required columns, header settings, and mapping-mode columns. Report the file and reason for each failure before starting Docker.

**Done when:** Invalid files or settings fail the workflow with actionable diagnostics, and representative valid `.txt` and `.xlsx` inputs pass preflight.

## 2. Share the duplicated workflow implementation

`run-CodeFusion.yaml` and `run-CodeFusion-Experimental.yaml` repeat the same processing, artifact, and pull-request steps. Move the shared logic into a reusable workflow or checked-in script, passing the parameter file and cache namespace as inputs. Keep the standard and experimental entry points distinct.

**Done when:** Both entry points use the same implementation, preserve their separate configuration and cache behavior, and produce the same outputs and pull-request flow as before.

## 3. Make each run traceable and reproducible

Record the CodeFusion image identity and hashes of the selected parameters and processed inputs with each run. Pin the Docker image by digest (while retaining a documented update process) and include the provenance in the uploaded artifact or pull-request details.

**Done when:** A reviewer can identify the exact image, configuration, and inputs behind a result, and the documented image update process can be followed without silently changing the executed image.
