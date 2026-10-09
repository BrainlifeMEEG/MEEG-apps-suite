# Brainlife.io MEG/EEG Apps Suite

A collection of [Brainlife.io](https://brainlife.io/) apps for MEG/EEG processing with
[MNE-Python](https://mne.tools/). This meta-repository holds every app as a git submodule; each
app is its own GitHub repository under [BrainlifeMEEG](https://github.com/BrainlifeMEEG), run on
brainlife.io from its `v1.0` branch.

**Repository**: https://github.com/BrainlifeMEEG/MEEG-apps-suite

## Getting the code

```bash
git clone --recurse-submodules git@github.com:BrainlifeMEEG/MEEG-apps-suite.git
cd MEEG-apps-suite
git submodule update --init --recursive   # if cloned without --recurse-submodules
```

Each app also carries its own `brainlife_utils/` submodule (shared helpers, from
[BrainlifeMEEG_utils](https://github.com/BrainlifeMEEG/BrainlifeMEEG_utils)).

## Apps

| Category | Apps |
|---|---|
| Format conversion to MNE `.fif` | `bdf2mne`, `brainvision2mne`, `ctf2mne`, `edf2mne`, `eeglab2mne`, `egi2mne`, `fif2mne` |
| MEG (FIF) specific | `maxwell-filter`, `head-pos` |
| Preparing raw data | `add-flat-chan`, `add-montage`, `average-channels`, `concat`, `crop-raw`, `interpolate-raw`, `mark-bad-raw`, `resampling` |
| Filtering | `filter-raw`, `filter-epo` |
| Artifact projectors (SSP / ICA) | `SSP-projectors-ECG`, `SSP-projectors-EOG`, `emptyroom-proj`, `SSP-apply`, `plot_proj_topomaps-raw`, `ICA-fit`, `ICA-fit-epo`, `ICA-apply`, `ICA-apply-epo`, `ICA-plot`, `autoreject` |
| Denoising | `gedai`, `gedai-epo` |
| Events and epochs | `events`, `eventslog`, `epoch`, `drop-bad-epo`, `interpolate`, `rereference`, `apply-baseline`, `plot-epochs` |
| Evoked responses | `evoked-averaged`, `average-erp` |
| Spectral features | `psd`, `epoch-psd`, `peak-amplitude`, `peak-frequency` |
| Data information | `info-raw`, `info-epo`, `info-evoked` |
| Source modelling | `make-watershed-bem`, `make-bem`, `coreg`, `source-space`, `forward`, `noise-covariance`, `inverse-operator`, `source-estimate`, `label-timecourse` |
| Full pipeline | `meegflow-app` |

Each app's own `README.md` documents its inputs, outputs and configuration parameters.

## Developing apps

- **[agent-instructions.md](agent-instructions.md)** is the single source of conventions: app
  structure, `brainlife_utils` usage, output naming, `product.json`, Docker images, the App
  README policy, and how to work with the brainlife.io CLI and API.
- **[.claude/agents/](.claude/agents/)** holds the Claude Code agents that apply those rules:
  auditing and fixing apps, syncing `brainlife_utils`, registering apps, switching the branch an
  app runs from, and setting up pipeline rules.
- **[docker-mne/](docker-mne/)**, **[docker-mne-freesurfer/](docker-mne-freesurfer/)** and
  **[docker-mne-fsaverage/](docker-mne-fsaverage/)** hold the container images the apps run in.
- **[local_scripts/](local_scripts/)** holds project-level tooling (pipeline reproduction,
  rule setup).
- **[docs/archive/](docs/archive/)** holds superseded reports, kept for history.

## Citation

If you use these apps in research, please cite:

```bibtex
@article{Hayashi2024,
  author  = {Hayashi, Soichi and Caron, Bradley A. and Heinsfeld, Anibal S. and others},
  title   = {brainlife.io: a decentralized and open-source cloud platform to support neuroscience research},
  journal = {Nature Methods},
  year    = {2024},
  volume  = {21},
  number  = {5},
  pages   = {809--813},
  doi     = {10.1038/s41592-024-02237-2}
}

@article{gramfort2013meg,
  title   = {MEG and EEG data analysis with MNE-Python},
  author  = {Gramfort, A. and Luessi, M. and Larson, E. and others},
  journal = {Frontiers in Neuroscience},
  volume  = {7},
  pages   = {267},
  year    = {2013},
  doi     = {10.3389/fnins.2013.00267}
}
```

## Authors

- Maximilien Chaumon (https://github.com/dnacombo)
- Guiomar Niso (https://github.com/guiomar)
- Kami Salibayeva (https://github.com/KSalibay)
- Saeed Zahran (https://github.com/zahransa/)
- Aurore Bussalb (https://github.com/abussalb)
- Franco Pestilli (https://github.com/francopestilli)

See individual app repositories for detailed contributor lists.

## License

All apps are licensed under AGPLv3 or later, as stated in [license.txt](license.txt) and in each
app repository, except where an app states otherwise (e.g. `gedai`, which wraps a
PolyForm Noncommercial library).

MNE-Python is Copyright (c) 2011-2026 MNE-Python developers, licensed under the 3-Clause BSD
License (https://mne.tools/stable/license.html).

## Resources

- Brainlife.io documentation: https://brainlife.io/docs
- MNE-Python documentation: https://mne.tools/
- Bugs: open an issue in the individual app repository.
