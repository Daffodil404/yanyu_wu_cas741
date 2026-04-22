# Integrating NCRF into an Existing TRF Pipeline

Developer: Yanyu Wu

Project start: January 2026

This repository contains the documentation deliverables for a CAS 741 project on integrating Neuro-Current Response Functions (NCRFs) into an existing Temporal Response Function (TRF) analysis pipeline. The implementation is not maintained as standalone source code in this repository. Instead, the project work extends the existing `TRF-Tools` codebase by adding NCRF as an estimator option inside the current TRF experiment workflow.

The runnable code and environment-sensitive demo are in the separate `TRF-Tools` repository. This README explains that relationship because the project is an integration task, not built from scratch.

## Project Scope

Traditional TRF workflows often estimate source activity first and then fit a temporal response model on the reconstructed source signals. This project targets the integration of NCRF into the existing pipeline so that spatial and temporal response properties can be estimated jointly.

The integration is designed to preserve the current TRF pipeline while adding NCRF support through estimator selection. In the intended workflow, users can choose between the existing boosting estimator and the new NCRF estimator without changing the overall experiment entry point.

## Repository Roles

This repository:

- Documents the problem, requirements, design, verification plan, and project traceability for the NCRF integration.
- Explains the expected inputs, outputs, assumptions, and environment.
- Records the design rationale for adding NCRF to an existing pipeline rather than building a separate tool.

The external `TRF-Tools` repository:

- Contains the Python implementation.
- Contains the demo scripts for running the estimator workflow.
- Requires a specific branch and Python environment.
- Depends on external datasets that are not included in either repository.

## Implementation Repository

Use the `feature/test-pipeline` branch of:

```text
https://github.com/Daffodil404/TRF-Tools.git
```

The estimator integration used for the demo is branch-specific. Other branches may not contain the NCRF estimator workflow described by this project.

Clone the implementation repository with:

```bash
git clone -b feature/test-pipeline https://github.com/Daffodil404/TRF-Tools.git
cd TRF-Tools
```

Verify the branch:

```bash
git branch --show-current
```

Expected output:

```text
feature/test-pipeline
```

## Required Environment

The demo depends on the `TRF-Tools` environment and specific versions of key scientific packages.

From the `TRF-Tools` checkout:

```bash
mamba env create -f env-trf.yml
mamba activate trf
mamba install -c conda-forge -c conda-forge/label/eelbrain_dev "eelbrain=0.42.0a5"
pip install -U "git+https://github.com/Eelbrain/neuro-currentRF.git@master"
pip install -e .
```

The editable install is important because `env-trf.yml` may install an upstream copy of `TRF-Tools`. Running `pip install -e .` ensures Python imports the local `feature/test-pipeline` checkout.

Optional check:

```bash
python -c "import trftools; print(trftools.__file__)"
```

The printed path should point to the local `TRF-Tools` clone.

## Required Data

The demo data are not included in this documentation repository or in the implementation repository. Users must provide:

- an Appleseed dataset organized in BIDS format
- the stimulus `.wav` files used to create the acoustic-envelope predictor

Download locations:

- Latest version of the Appleseed dataset: [MacDrive download link](https://macdrive.mcmaster.ca/f/52d637bc76ab4dfaad28/?dl=1)
- Stimulus `.wav` files for generating the predictor: [Brodbeck Lab SharePoint stimuli folder](https://mcmasteru365.sharepoint.com/sites/BrodbeckLab/Shared%20Documents/Forms/AllItems.aspx?csf=1&web=1&e=HjgRML&ovuser=44376307-b429-42ad-8c25-28cd496f4772%2Cwu1064%40mcmaster.ca&OR=Teams-HL&CT=1776758626135&clickparams=eyJBcHBOYW1lIjoiVGVhbXMtRGVza3RvcCIsIkFwcFZlcnNpb24iOiI1MC8yNjAzMTIyMzAyMCIsIkhhc0ZlZGVyYXRlZFVzZXIiOmZhbHNlfQ%3D%3D&CID=df130ca2-2041-6000-bff5-54e56046bd04&cidOR=SPO&FolderCTID=0x012000DE68ADDE19558842A0CA0CFFB6CA8F46&id=%2Fsites%2FBrodbeckLab%2FShared+Documents%2FGeneral%2FDataset%2FAppleseed%2Fstimuli)

The demo scripts include example local data paths, but users must replace them with the actual dataset and stimulus paths on their own machine. The current demo paths are similar to:

```text
/Users/yanyuwoo/Data/Appleseed_BIDS_20251216
/Users/yanyuwoo/Data/stimuli
```

Before running the demo, update the path constants in the implementation repository if they do not match your local machine:

```text
trftools/pipeline/estimator/demo.py
trftools/pipeline/estimator/make_demo_predictor.py
```

Expected BIDS-style layout:

```text
DATA_ROOT/
  sub-XXXX/
    meg/
    anat/
  derivatives/
    predictors/
    ica/
    freesurfer/
    trans/
    eelbrain/
```

## Running the NCRF Demo

After cloning the correct branch, creating the environment, downloading the data, and setting local paths, create the predictor:

```bash
python trftools/pipeline/estimator/make_demo_predictor.py
```

This writes the acoustic-envelope predictor to:

```text
<DATA_ROOT>/derivatives/predictors/Appleseed~acoustic_envelop.pickle
```

Then run the demo:

```bash
python trftools/pipeline/estimator/demo.py
```

To explicitly run the NCRF path:

```bash
python -c "from trftools.pipeline.estimator.demo import run_ncrf_demo; run_ncrf_demo()"
```

To run the baseline boosting path:

```bash
python -c "from trftools.pipeline.estimator.demo import run_boosting_demo; run_boosting_demo()"
```

## Quick Command Summary

```bash
git clone -b feature/test-pipeline https://github.com/Daffodil404/TRF-Tools.git
cd TRF-Tools

mamba env create -f env-trf.yml
mamba activate trf
mamba install -c conda-forge -c conda-forge/label/eelbrain_dev "eelbrain=0.42.0a5"
pip install -U "git+https://github.com/Eelbrain/neuro-currentRF.git@master"
pip install -e .

# Download the Appleseed BIDS data and stimulus WAV files.
# Then update local data paths in:
# - trftools/pipeline/estimator/demo.py
# - trftools/pipeline/estimator/make_demo_predictor.py

python trftools/pipeline/estimator/make_demo_predictor.py
python trftools/pipeline/estimator/demo.py
```

## Documentation Map

The main documentation deliverables in this repository are:

- `docs/ProblemStatementAndGoals/ProblemStatement.tex`: problem statement, stakeholders, inputs, outputs, assumptions, and project goals
- `docs/SRS/SRS.tex`: software requirements specification
- `docs/Design/SoftArchitecture/MG.tex`: module guide and architecture-level design
- `docs/Design/SoftDetailedDes/MIS.tex`: module interface specification
- `docs/VnVPlan/VnVPlan.tex`: verification and validation plan
- `docs/VnVReport/VnVReport.tex`: verification and validation report
- `docs/ReflectAndTrace/ReflectAndTrace.tex`: reflection and traceability

The `src/` and `test/` folders are retained from the course template, but the project implementation itself is integrated into the external `TRF-Tools` repository described above.

## Troubleshooting

If the demo does not run, check the following first:

- The `TRF-Tools` checkout is on `feature/test-pipeline`.
- The active environment is `trf`.
- `Eelbrain` is version `0.42.0a5`.
- `ncrf` was installed from `Eelbrain/neuro-currentRF` on `master`.
- `pip install -e .` was run inside the local `TRF-Tools` checkout.
- The Appleseed BIDS dataset path is correct.
- The stimulus `.wav` directory path is correct.
- `make_demo_predictor.py` has generated `Appleseed~acoustic_envelop.pickle`.
