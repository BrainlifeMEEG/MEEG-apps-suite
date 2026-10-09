# Brainlife.io MNE Apps - Instructions

This is the single source of conventions for every app in this repo. The agents in
`.claude/agents/` enforce or apply these rules and link back to the sections here rather than
restating them; when a rule changes, change it here only.

## Project Overview

This is a collection of Brainlife.io applications for neuroimaging data processing, specifically focused on MEG/EEG data analysis using the MNE-Python library. Each folder contains a separate app with a standardized structure for processing neuroimaging data through containerized Python scripts.

## Project Structure

### App Organization
- Each folder contains a complete Brainlife.io application
- Apps follow a consistent pattern for neuroimaging data processing workflows
- Apps are designed to run in Docker/Singularity containers via the Brainlife.io platform

### Standard App Components

Every app typically contains:

1. **`main`** - Bash script that:
   - Sets up PBS/SLURM job parameters
   - Executes the Python script via Singularity container (image: see "Docker images" below)
   - The Python entrypoint invoked must be named exactly `main.py` — no alternate entrypoint filenames are permitted

2. **`main.py`** - Main Python script that:
   - No ad-hoc definitions for "main" or "generate-report" or "apply_filter" functions, but rather a single main.py that handles all processing steps for the app
   - Loads configuration from `config.json`
   - Ensures only the output directories the app actually writes to exist — typically some subset of `out_dir`, `out_figs`, `out_report`. Do not create unused output directories
   - Processes neuroimaging data using MNE-Python
   - Saves outputs to designated directories
   - Generates reports and visualizations as needed
   - Creates `product.json` for Brainlife.io interface as needed

3. **`config.json`** - Configuration file containing example:
   - Input file paths
   - Processing parameters such as
    - Channel selections
    - Filter settings
    - Event mappings
   - No comment fields
   - Required for every app that exposes user-configurable parameters; apps with no parameters may omit it, but must still call `load_config()` with sane defaults
   - A `config.json.example` alone is never sufficient — if example values are meant to be used for testing, `config.json` itself must also exist

4. **`README.md`** - Must follow the "App README policy" section below.

5. **`brainlife_utils/`** - Shared utility library containing:
   - must be a real git submodule: git submodule add git@github.com:BrainlifeMEEG/brainlifeMEEG_utils.git brainlife_utils. A plain copied `brainlife_utils/` directory (no `.git`) is non-compliant — it cannot receive upstream fixes and will drift
   - Configuration handling (`config_utils.py`)
   - File operations (`file_utils.py`)
   - Data processing helpers (`data_utils.py`)
   - Report generation (`report_utils.py`)
   - Plotting utilities (`plot_utils.py`)
   - No local helper/utility module of any name (`helper.py`, `brainlife_apps_helper/`, or similar) is permitted. All shared logic must live in `brainlife_utils`; if a helper contains logic not yet in `brainlife_utils`, upstream it there first, then remove the local copy.

## Common Patterns

### Data Flow
1. Input: EEG/MEG data files: always `.fif` except for conversion apps (e.g. egi2mne)
2. Processing: MNE-Python analysis functions
3. Output: Processed data files, reports, and visualizations

### Container Usage
- Apps use the images listed under "Docker images" below.
- Executed via Singularity for HPC compatibility

### Output Structure
- `out_dir/` - Primary data outputs (e.g., `raw.fif`, `epo.fif`)
- `out_figs/` - PNG plots and visualizations
- `out_report/` - HTML reports (MNE Report html)
- `product.json` - Metadata for Brainlife.io interface

### Configuration Handling
- JSON configurations with parameter validation
- `brainlife_utils.config_utils` provides None-value conversion for config parameters

## Docker images

Use the lean `brainlifemeeg/*` images; the older `brainlife/mne:*`, `brainlife/mne-freesurfer:*` and
third-party personal images are being phased out.

| Image | Use for |
|---|---|
| `docker://brainlifemeeg/mne:1.12.1` | Default for every app that only needs MNE-Python |
| `docker://brainlifemeeg/mne-freesurfer:1.12.1-7.4.1` | Apps that call FreeSurfer binaries or need Qt/pyvista offscreen 3D rendering (coreg, forward, source-estimate, label-timecourse, make-bem, source-space) |
| `docker://brainlifemeeg/mne-fsaverage:1.12.1` | Apps that need the `fsaverage` template subject baked in (gedai, gedai-epo) |

Image sources live in `docker-mne/`, `docker-mne-freesurfer/` and `docker-mne-fsaverage/`; read
the README there before adding or bumping an image. Extra Python packages not in the image are
`pip install --user`-ed at run time from `main` (meegflow-app and gedai pattern), not baked into a new image.

## App Categories

### Data Conversion Apps (`*2mne`)
- Convert various formats to MNE-compatible `.fif` files
- Examples: `bdf2mne`, `edf2mne`, `ctf2mne`

### Preprocessing Apps (`filter-*`, `*-filter`)
- Apply temporal filters
- Examples: `filter-raw`, `filter-epo`, `filter-chpi`

### Projector computation for artifact removal (`ICA-*`, `SSP-*`)
- Independent Component Analysis and Signal Space Projection
- Examples: `ICA-fit`, `ICA-apply`, `SSP-projectors-ECG`

### Denoising (`gedai*`)
- Examples: `gedai`, `gedai-epo`

### Epoching and Events (`epoch*`, `events*`)
- Event detection and epoch extraction
- Examples: `epoch`, `events`

### Analysis Apps (`psd`, `peak-*`)
- Spectral analysis and feature extraction
- Examples: `psd`, `peak-amplitude`, `peak-frequency`

### Source modelling
- Examples: `make-bem`, `coreg`, `source-space`, `forward`, `noise-covariance`, `inverse-operator`, `source-estimate`, `label-timecourse`

## Development Guidelines

### When Creating New Apps:
1. Follow the standard directory structure
2. Use an image from "Docker images"
3. Implement proper error handling and validation
4. Generate MNE Reports for quality control
5. Create informative `product.json` outputs
6. Write the README to the "App README policy"

### Code Conventions:
- Import MNE-Python and standard scientific libraries
- Use shared utilities from `brainlife_utils` package
- Use `load_config()` for configuration loading and preprocessing
- Use `setup_matplotlib_backend()` for headless execution
- Use `ensure_output_dirs()` for creating output directories
- Build `product.json` via the `product_items` accumulator pattern — see "Product Metadata Convention" below
- Generate base64-encoded images for web display

### Output file naming conventions:
- Raw data files should be called raw.fif
- Epoched data files should be called epo.fif
- Evoked data files should be called ave.fif
- ICA solutions should be called ica.fif
- SSP/ECG/EOG projectors should be called proj.fif
- Any other derived artifact not covered above should use `<type>.fif` with the MNE-conventional suffix — never a custom filename
- reports are all called report.html
- Exception: apps that accept a list of input files and refine each independently (rather than merging them into one output, e.g. `fif2mne`) may name outputs `<type>_<n>.fif` (e.g. `raw_1.fif`, `raw_2.fif`) when there is more than one, falling back to the plain `<type>.fif` name for a single input. This is the one sanctioned deviation from the single-fixed-name rule above.

### Shared Utilities Usage:
```python
import sys
import os
sys.path.insert(0, os.path.join(os.path.dirname(__file__), 'brainlife_utils'))

from brainlife_utils import (
    load_config, 
    setup_matplotlib_backend, 
    ensure_output_dirs,
    create_product_json,
    add_image_to_product,
    add_info_to_product,
    add_raw_info_to_product
)

# Set up environment
setup_matplotlib_backend()
config = load_config()
ensure_output_dirs('out_dir', 'out_figs', 'out_report')  # only the dirs this app actually writes to

# Build product.json — see "Product Metadata Convention" below for the full pattern
```

## Product Metadata Convention (Required)

For all apps, always build `product.json` using an explicit list accumulator.

Required pattern:
```python
product_items = []
add_info_to_product(product_items, "message")
add_raw_info_to_product(product_items, raw)
add_image_to_product(product_items, "Figure title", filepath="out_figs/plot.png")
create_product_json(product_items)
```

Rules:
- Initialize `product_items = []` before any `add_*_to_product` call.
- Pass `product_items` as the **first argument** to all relevant helpers:
  - `add_info_to_product`
  - `add_raw_info_to_product`
  - `add_image_to_product`
  - `add_plotly_to_product`
- Do not use `product_json = create_product_json()` as an accumulator.
- Call `create_product_json(product_items)` only after all product items are added.

### Testing Considerations:
- A missing or invalid required config key must produce an `add_info_to_product` warning and a clean exit — never an uncaught stack trace.
- Every processing parameter read from `config.json` must be validated (type/range/allowed values) before use.
- Output `.fif` files must load successfully with the corresponding MNE reader (`mne.io.read_raw_fif`, `mne.read_epochs`, etc.) before the app exits successfully.

## App README policy

Every app's `README.md` follows this structure, in this order. It documents what the code
actually does; take parameter names, types and defaults from `main.py` and `config.json`, never
from memory.

1. **Title** — `# <Human-readable app name>` (e.g. "Filter Raw MEG/EEG Data"), not the repo name and
   never another app's title copied over.
2. **Badges**, directly under the title:
   - Run on Brainlife.io, linking the app's DOI:
     `[![Run on Brainlife.io](https://img.shields.io/badge/Brainlife-bl.app.NNN-blue.svg)](https://doi.org/10.25663/brainlife.app.NNN)`.
     If the app has no DOI yet, link `https://brainlife.io/app/<app_id>` instead.
3. `## Description` — one or two paragraphs on what the app does and which MNE function/method it
   uses, then "The app generates:" with a bullet list of outputs.
4. `## Inputs` — one bullet per input, named by its **config key** and datatype:
   ``- **`mne`** (`neuro/meeg/mne/raw`): continuous data to filter (required)``.
   Never name an input by a filename like "meg.fif".
5. `## Outputs` — one bullet per output, with its path and datatype, including
   `out_report/report.html` and figures when the app produces them.
6. `## Configuration Parameters` — a table with one row per config key that `main.py` reads:
   `| key | type | default | description |`. Keys, types and defaults must match `main.py` and
   `config.json`.
7. `## Usage` — `### Running on Brainlife.io` (numbered steps) and `### Local Testing`. All shell
   commands go inside a fenced code block; a `# comment` outside a fence renders as a stray H1.
8. Optional: `## Technical Details`, `## Limitations`, `## Pipeline Position`.
9. `## Authors` — `- Name (https://github.com/handle)`.
10. `## Citations` — always:
    - Hayashi, S., Caron, B.A., Heinsfeld, A.S. et al. brainlife.io: a decentralized and open-source cloud platform to support neuroscience research. Nat Methods 21, 809–813 (2024). https://doi.org/10.1038/s41592-024-02237-2
    - Gramfort, A. et al. MEG and EEG data analysis with MNE-Python. Front. Neurosci. 7, 267 (2013). https://doi.org/10.3389/fnins.2013.00267
    - plus the paper for the specific algorithm, where one applies.
11. `## Funding Acknowledgement` — the standard sentence plus these six badges:
    ```markdown
    [![NSF-BCS-1734853](https://img.shields.io/badge/NSF_BCS-1734853-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1734853)
    [![NSF-BCS-1636893](https://img.shields.io/badge/NSF_BCS-1636893-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1636893)
    [![NSF-ACI-1916518](https://img.shields.io/badge/NSF_ACI-1916518-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1916518)
    [![NSF-IIS-1912270](https://img.shields.io/badge/NSF_IIS-1912270-blue.svg)](https://nsf.gov/awardsearch/showAward?AWD_ID=1912270)
    [![NIH-NIBIB-R01EB029272](https://img.shields.io/badge/NIH_NIBIB-R01EB029272-green.svg)](https://grantome.com/grant/NIH/R01-EB029272-01)
    [![NIH-NIBIB-R01EB030896](https://img.shields.io/badge/NIH_NIBIB-R01EB030896-green.svg)](https://grantome.com/grant/NIH/R01-EB030896-01)
    ```
12. `## License` — `Copyright (c) <year> MEEG Brainlife team. Licensed under AGPL-3.0, see [license.txt](license.txt).`
    Apps wrapping a differently-licensed library (e.g. gedai, PolyForm Noncommercial) state that
    license here instead and keep the notice block near the top.

Not allowed:
- a second, trailing `## Citation` block repeating the Hayashi reference;
- the 2023 arXiv/PMC version of the Hayashi citation;
- the old "Documentation" style (a numbered "Input file is / Output files are" list instead of sections);
- repo names, branch names or docker images that don't match the app's current `main` and registration.

### README checklist

The auditor reports, and the inventory sheet's `Readme` column records, a README as compliant
only if every item passes:

1. H1 title present, not the repo name, not another app's name.
2. Run-on-Brainlife badge present, linking a DOI or a brainlife.io app id.
3. Sections `Description`, `Inputs`, `Outputs`, `Configuration Parameters` (unless the app reads no config keys), `Usage`, `Authors`, `Citations`, `Funding Acknowledgement`, `License` all present, in that order.
4. Every config key read in `main.py` appears in the Configuration Parameters table or as an input.
5. Citations include Hayashi 2024 (Nat Methods) and Gramfort 2013.
6. All six funding badges present, including both NIH grants.
7. No duplicate trailing `## Citation`, no 2023 Hayashi citation, no stray H1 outside the title.
8. Any docker image named in the README matches `main`.

## Repository Hygiene

- When an app is renamed or retired, remove its entry from the root `.gitmodules` — do not leave orphaned submodule declarations pointing at a directory that no longer exists.
- Superseded reports and status documents go to `docs/archive/`, not the repo root.

## CLI use and Brainlife documentation

### Documentation

- User/app docs live at https://brainlife.io/docs — sections cover Projects, Processes, Pipelines, Apps (registration, `config.json.schema`, `product.json`), Datatypes, Resources, Publications.
- Source for these docs (to check for very recent additions before they're indexed by search): https://github.com/brainlife/docs — e.g. `docs/user/*.md`, `docs/apps/*.md`, `docs/cli/*.md`, `docs/technical/api.md`.
- The docs site can lag behind the live platform by months — new UI features (e.g. the "save pipeline group as workflow" feature added July 2026) may only be discoverable by inspecting the deployed app directly (see "Direct API calls" below for how), not by reading the docs.

### CLI

- Install: `sudo npm install -g brainlife` (npm package name is `brainlife`, the binary is `bl`).
- Login: `bl login --ttl 7` (the `--ttl` is the token lifetime in days).
- Source: https://github.com/brainlife/cli — install/usage docs per subcommand at https://brainlife.io/docs/cli/ (`install`, `upload`, `download`, `app`, `group`, `update`).
- CLI coverage is intentionally limited — it wraps the Warehouse API for common tasks (upload/download datasets, submit/query apps, manage projects). For anything it doesn't support, fall back to direct API calls.
- `bl app` has only `query`/`run`/`wait`: no create, register or update subcommand. There is no `rule` subcommand at all. App registration (`POST app`), app updates such as changing `github_branch` (`PUT app/:id`) and pipeline rules (`POST rule`, `PUT rule/order/:projectId`) all go through the Warehouse API directly.

### Authentication

- A cached login token lives at `~/.config/brainlife.io/.jwt` (written by `bl login`). Read it into a shell variable for the `Authorization: Bearer` header; never print it or copy it into another file.
- Warehouse reads of public records (apps, datatypes) need no token. Writes always do, and `_canedit` in a read only shows up when a token is sent.
- If a request returns `HTTP 500` with `{"message":"UnauthorizedError: jwt expired"}`, stop and ask the user to run `bl login` themselves; an agent cannot log in on their behalf.
- Edit rights on an app come only from membership in its `admins` list, never from being its creator. This project's convention is `admins: ["670", "720", "1348"]`.

### Direct API calls

Brainlife is a set of microservices, each with its own base URL under `brainlife.io`. All of them expect `Authorization: Bearer <jwt>` (the JWT from `bl login`, or from the `auth` service directly).

| Service | Base URL | Purpose | API docs |
|---|---|---|---|
| Warehouse | `https://brainlife.io/api/warehouse` | Projects, datasets, apps, pipelines/rules, workflows, tasks | https://brainlife.github.io/warehouse/apidoc |
| Amaretti | `https://brainlife.io/api/amaretti` | Task submission/monitoring on compute resources | https://brainlife.github.io/amaretti/apidoc |
| Auth | `https://brainlife.io/api/auth` | Login, profile, JWT issuance | source is private; contact brainlife devs for details |
| Event | `https://brainlife.io/api/event` | Event bus (also exposed as a websocket) | — |

More detail: https://brainlife.io/docs/technical/api and the corresponding source in the docs repo (`docs/technical/api.md`).

Practical tip for reverse-engineering a feature that isn't documented yet: the warehouse UI (a Vue 2 app) is served at `brainlife.io` as `/static/js/app.<hash>.js` plus lazy-loaded numbered chunks (`/static/js/<n>.<hash>.js`, hash map is in `/static/js/manifest.<hash>.js`). Grepping the deployed bundle for a UI string (e.g. a button's tooltip text) is often faster than digging through GitHub when the public repo hasn't caught up to production yet.

### Inspecting a process's task logs (slurm-*.err, product.json, outputs)

A `brainlife.io/project/<id>/process/<id>` URL's `process` ID is an Amaretti
**instance** ID — Warehouse has no "process"/"instance" model of its own,
this concept lives entirely in Amaretti (source: `brainlife/amaretti`,
`api/controllers/{instance,task}.js`). An instance groups one task per
pipeline step. No `bl` CLI subcommand covers this (`bl app run`/`bl app wait`
only submit/wait on new runs) — it's direct-API-only:

```bash
TOKEN=$(cat ~/.config/brainlife.io/.jwt)   # populated by `bl login`

# 1. instance -> per-step task IDs + statuses (config.summary[].task_id)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://brainlife.io/api/amaretti/instance?find=%7B%22_id%22%3A%22<processId>%22%7D"

# 2. task detail (status, status_msg e.g. TIMEOUT, resource, env, timings)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://brainlife.io/api/amaretti/task/<taskId>"

# 3. list files in the task's working directory (slurm-*.log/.err, product.json, out_dir/...)
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://brainlife.io/api/amaretti/task/ls/<taskId>"

# 4. download a specific file
curl -s -H "Authorization: Bearer $TOKEN" \
  "https://brainlife.io/api/amaretti/task/download/<taskId>/slurm-<jobid>.err" -o slurm.err
```

The same workdir also exists on disk on this cluster, under
`/network/iss/brainlife/users/brlife/workdir/<instanceId>/<taskId>/` — but
it's owned by the `brlife` service account and not readable by other users,
so the API route above is the reliable path.

### Reducing noise in slurm-*.err logs

`.err` is just the job's raw stderr — SLURM doesn't filter it by severity, so
it collects whatever every layer writes to fd 2. In practice that's three
independent sources, each fixable separately:

- **Apptainer/Singularity's own INFO/WARNING chatter** (`INFO: Using cached
  SIF image`, `WARNING: passwd file doesn't exist in container...` — the
  latter is benign/expected for rootless containers on shared HPC). Suppress
  with a *global* flag placed before the subcommand, not after:
  `singularity --silent exec docker://... python3 main.py` (`--silent` =
  errors only; `--quiet` = still shows warnings). This only affects
  Apptainer's own messages, not anything the contained process writes.
- **joblib/scikit-learn parallel progress spam**
  (`[Parallel(n_jobs=1)]: Done N out of N | elapsed: ...`), written by code
  MNE calls internally (e.g. `Report.add_evokeds`, `autoreject`) — this comes
  from MNE's own log-level default, not an explicit `verbose=` passed by app
  code. Fix at the source: call `mne.set_log_level('WARNING')` early in
  `main.py` (or centrally once inside `brainlife_utils`, e.g. in
  `setup_matplotlib_backend()` or `load_config()`, so every app gets it for
  free) — this collapses MNE's internal joblib verbosity to 0.
- **Genuine Python warnings** (e.g. `RuntimeWarning: This filename ... does
  not conform to MNE naming conventions`) — these are real signal, not noise;
  don't blanket-suppress `warnings.filterwarnings('ignore')`. Fix the
  underlying cause instead (in that example: MNE expects evoked files to end
  in `-ave.fif`/`_ave.fif`, which conflicts with this repo's own
  `ave.fif`-only convention — worth resolving one way or the other rather
  than living with the warning on every run).
