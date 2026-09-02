<div align="center">

# PyPSA-IMG

**An open optimisation model for the design and assessment of islanded, hydrogen-integrated industrial microgrids**

Multi-carrier (electricity · hydrogen · heat) microgrid optimisation in PyPSA at full hourly resolution, with built-in techno-economic, life-cycle, uncertainty, sensitivity, and cost-emissions trade-off analysis.

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB.svg?logo=python&logoColor=white)](https://www.python.org/)
[![PyPSA 0.20.1](https://img.shields.io/badge/PyPSA-0.20.1-1f77b4.svg)](https://pypsa.org/)
[![Solver: GLPK](https://img.shields.io/badge/Solver-GLPK-6aa84f.svg)](https://www.gnu.org/software/glpk/)
[![Paper](https://img.shields.io/badge/Paper-ESD%202026-blue.svg)](https://doi.org/10.1016/j.esd.2026.102133)
[![DOI](https://img.shields.io/badge/Zenodo-pending-lightgrey.svg)](#citation)

<!--
  Banner image. Export the three-bus microgrid schematic from the notebook
  (Cell 44), save it to docs/microgrid_schematic.png, and uncomment the line
  below. ~820px wide reads well on GitHub.
-->
<!-- <img src="docs/microgrid_schematic.png" alt="Islanded wind-hydrogen-CHP microgrid schematic" width="820"> -->

</div>

---

## Table of contents

- [Overview](#overview)
- [What the model does](#what-the-model-does)
- [Quick start](#quick-start)
- [Adapting the model to your own site](#adapting-the-model-to-your-own-site)
- [Requirements and environment](#requirements-and-environment)
- [Running on other platforms](#running-on-other-platforms)
- [Data access](#data-access)
- [Configuration](#configuration)
- [Long runs and checkpoints](#long-runs-and-checkpoints)
- [Reproducibility](#reproducibility)
- [Development status and roadmap](#development-status-and-roadmap)
- [Notebook structure](#notebook-structure)
- [Limitations](#limitations)
- [Contributing and contact](#contributing-and-contact)
- [Citation](#citation)
- [License](#license)
- [Nomenclature and abbreviations](#nomenclature-and-abbreviations)

---

## Overview

PyPSA-IMG is an open-source optimisation model for designing and assessing **islanded, hydrogen-integrated industrial microgrids**. It couples electricity, hydrogen, and heat within a single [PyPSA](https://pypsa.org/) network and solves the system as a linear optimal power flow (LOPF) at full hourly resolution (8,760 snapshots), sizing and dispatching each component at least cost.

Around this core dispatch, the model provides an integrated analysis pipeline: a techno-economic assessment, a life-cycle carbon assessment, a Monte Carlo uncertainty analysis, a deterministic sensitivity ranking, and a cost-emissions trade-off study. It is written to be re-pointed at any industrial site by substituting the wind resource and demand profile.

The model was developed and validated for the chemical and process industrial cluster in Teesside, United Kingdom; that application and its results are reported in the associated paper (see [Citation](#citation)). Although developed for this case study, PyPSA-IMG is built as a general framework for islanded industrial microgrids and can be applied to other industrial sites and clusters by substituting the wind resource and demand profile (see [Adapting the model to your own site](#adapting-the-model-to-your-own-site)). This repository is the code and data-availability record for the associated publication and the maintained home of the model.

## What the model does

The microgrid integrates onshore wind, a PEM electrolyser, hydrogen storage, a battery, a hydrogen-fuelled CHP unit, waste-heat recovery, and a backup gas boiler. On top of the LOPF dispatch, the model runs:

| Analysis | What it produces |
|---|---|
| **Techno-economic assessment (TEA)** | Endogenous component sizing, annualised cost, levelised cost of energy and hydrogen, discounted cash flow, NPV, IRR |
| **Economic Viability Roadmap (EVR)** | How contingent policy levers (private-wire supply, hydrogen support, capital grants, oxygen by-product sales) move viability, a basis for industrial-decarbonisation policy analysis |
| **Life-cycle carbon assessment (LCA)** | Operational and embodied emissions, component-replacement accounting, carbon payback, carbon return on investment |
| **Monte Carlo uncertainty** | Probabilistic stress-testing over 1,000 iterations, paired base and roadmap cases per draw |
| **Sensitivity ranking** | One-at-a-time (tornado) ranking of the dominant cost and emission drivers |
| **Cost-emissions trade-off** | A carbon-price sweep tracing the cost against operational-emissions frontier and its marginal abatement cost |

Every analysis runs at full 8,760-hour resolution, and every results cell is guarded by an internal validation suite that reconciles its outputs against locked reference values before they are used.

## Quick start

The model runs end to end in one notebook, `PyPSA-IMG_v1.0.ipynb`. Google Colab is the fastest route; local and other platforms are covered [below](#running-on-other-platforms).

**Google Colab (recommended)**

1. Open `PyPSA-IMG_v1.0.ipynb` in Colab.
2. Run the first cell (environment setup); it installs the pinned dependencies. **Restart the runtime when prompted**, then run again from the top so the pinned versions are the ones loaded.
3. *(Optional)* To retrieve fresh wind data, add a Copernicus Climate Data Store API key as a Colab secret named `CDS_API_KEY`. The wind file included in this repository can be used directly instead.
4. Run the cells in order. The deterministic results (TEA, LCA, trade-off) reproduce in a few minutes.

**Local**

```bash
pip install -r requirements.txt                            # Python 3.12 environment
sudo apt-get install glpk-utils libproj-dev libgeos-dev    # Debian/Ubuntu; see below for Windows/macOS
```

Then launch Jupyter, open `PyPSA-IMG_v1.0.ipynb`, and run the cells in order from the top.

> **Runtime.** The deterministic results (baseline, proposed case, TEA, LCA, trade-off) complete in a few minutes. The full Monte Carlo analysis takes several hours and is gated off by default; it is not required to reproduce the published deterministic results (see [Long runs and checkpoints](#long-runs-and-checkpoints)).

## Adapting the model to your own site

The model is designed to be re-pointed at a different industrial site:

1. **Demand.** Replace `chemical_industry_load_profile_8760h.csv` with your own 8,760-row hourly series using the same columns (`Timestamp`, `electricity_demand_kW`, `heat_demand_kW`), and update the CSV path in the demand-loading cell (Cell 18). Timestamps must align with the network snapshots; the cell asserts this and converts kW to MW automatically. Any site-specific preparation of the profile (a scaling factor, or a heat-to-power ratio used to derive the heat series) should be done before this cell.
2. **Wind resource.** Set the site coordinates in the marked configuration cell; the notebook retrieves the matching ERA5 grid cell from the Copernicus Climate Data Store (free API key required), or supply your own ERA5 `.nc` file in the same format.
3. **Parameters.** Adjust the real WACC, carbon price, project lifetime, technology-inclusion flags, and wind loss sub-factors in the `ProjectConfig` dataclass. Values a user may reasonably change are flagged inline with an `adjustable` note in the code.
4. **Reset checkpoints.** Set `CHECKPOINT_RESET = True` (or delete the checkpoint files) so results are recomputed against your inputs, not carried over.
5. **Re-run from the top.** Results are specific to the wind resource and demand profile, so a full re-run is required; checkpoints from another configuration must not be reused.

## Requirements and environment

The model runs in a version-pinned environment to guarantee reproducibility. Core dependencies are **PyPSA 0.20.1** (frozen, because later releases alter component cost attributes), **NumPy < 2.0**, **pandas < 2.3**, **Pyomo < 6.7**, and **SciPy < 1.12**, with the **GLPK** solver (invoked with `pyomo=False` via PyPSA's native `linopy` path). Development was on Google Colab with **Python 3.12**; local execution is supported through `requirements.txt`. System-level dependencies (GLPK, and PROJ/GEOS for the optional maps) are installed through the host package manager, not pip.

<details>
<summary><strong>Expected warnings (all harmless, click to expand)</strong></summary>

On first import, the frozen environment emits a Pyomo deprecation notice and several pandas future-warnings that originate **inside PyPSA 0.20.1** (for example at `pypsa/components.py:310`). These are expected on every run and have no effect on the numerical results; the network builds successfully regardless.

On Google Colab, `pip check` additionally reports version conflicts against Colab's own pre-installed packages (for example `jax`, `opencv`, `rasterio`, `spopt`), which require newer NumPy or SciPy than the versions pinned here. Those packages are **not used by the model**, so the warnings are a consequence of Colab's large base environment rather than a problem with the model's dependencies, and can be safely ignored. On a clean local or JupyterLab environment built only from `requirements.txt`, these conflict warnings do not appear.

</details>

## Running on other platforms

The notebook is standard Jupyter and runs anywhere a Python 3.12 kernel and the pinned dependencies are available. The only Colab-specific pieces are the optional file auto-download and the `google.colab` imports, which are guarded by an `IN_COLAB` flag and skipped automatically off Colab.

- **JupyterLab / classic Jupyter (local or server).** Install with `pip install -r requirements.txt jupyterlab`, then `jupyter lab`. Files are written to the working directory; leave `CONFIG.auto_download = False` (the default), as browser download is Colab-only.
- **VS Code / Cursor.** Open the folder, select a Python 3.12 interpreter with the pinned dependencies installed, and run through the built-in Jupyter support. No other changes needed.
- **Kaggle Notebooks.** Upload the notebook and data files as a dataset, or clone the repository, then install the pinned versions in the first cell. Kaggle has no `google.colab` module, so the Colab download branch is skipped automatically.
- **Other cloud notebooks (Binder, SageMaker, Deepnote, …).** Any Python 3.12 kernel works. Install `requirements.txt` and the system-level solver/geospatial libraries, then run top to bottom.

**System libraries by platform.** The inline `!apt-get` installs in the setup cell assume a Debian/Linux environment (Colab, most cloud notebooks, Linux local), where they work out of the box. On **Windows or macOS local** machines, `apt-get` is unavailable, install the system libraries with conda before running, and skip the `apt-get` lines in the setup cell:

```bash
conda install -c conda-forge glpk proj geos
```

**Solver.** GLPK is the only required solver and is free on every platform. If GLPK is unavailable, any LP solver PyPSA 0.20.1 supports (for example CBC) can be selected in the configuration cell; results are solver-independent for this linear problem.

## Data access

Two datasets are included so the published results reproduce directly:

- **Wind resource** (`era5_wind_100m_2023_8760h.nc`): ERA5 hourly 100 m wind components for the study grid cell (2023), from the Copernicus Climate Data Store. To retrieve fresh or alternative-site data, a free Climate Data Store account and API key are required, supplied through a Colab secret or the environment variable `CDS_API_KEY` (never committed to the repository).
- **Industrial demand profile** (`chemical_industry_load_profile_8760h.csv`): an 8,760-row hourly electricity and heat demand series (`Timestamp`, `electricity_demand_kW`, `heat_demand_kW`). Users adapting the model substitute a file of the same format.

Energy prices, greenhouse-gas conversion factors, the carbon price, and technology cost catalogues used as parameters are drawn from the sources documented in the paper and are not redistributed here. Redistribution of the included datasets is subject to their original providers' terms (see [License](#license)).

## Configuration

Project-wide settings are centralised in a single `ProjectConfig` dataclass (instantiated as `CONFIG` in Cell 3): the project lifetime, real WACC (`wacc_real`), central carbon price (`carbon_price_central`), solver (`solver_name`, default `glpk`), Monte Carlo sample count (`mc_samples`), and the Colab download flag (`auto_download`, default `False`). Site-specific inputs, coordinates, the demand file, the technology-inclusion scenario, and the wind loss sub-factors, are set in clearly marked cells. Numerical values a user may reasonably change are flagged inline with an `adjustable` note.

## Long runs and checkpoints

The Monte Carlo analysis and the carbon-price sweep periodically save progress to a checkpoint file so an interrupted run resumes rather than restarts.

- **Gating.** The Monte Carlo loop is gated behind `RUN_MONTE_CARLO` (default `False`) so a full re-run is never triggered by accident; a completed checkpoint is always loaded regardless. Set it to `True` to run or resume.
- **Default paths.** `./mc_checkpoints/mc_checkpoint.pkl` and `./tradeoff_checkpoints/`. On Colab the session-local filesystem is erased on disconnect, so to survive a disconnect, mount your own Google Drive and point the path at it:

  ```python
  from google.colab import drive
  drive.mount('/content/drive')
  CHECKPOINT_PATH = "/content/drive/MyDrive/your-folder/mc_checkpoint.pkl"
  ```

- **Changing inputs.** Checkpoints are specific to the configuration and data that produced them; the Monte Carlo checkpoint refuses to resume if settings differ, and the carbon-price checkpoint is ignored if the price grid changes. When adapting to a new site, set `CHECKPOINT_RESET = True` (or delete the checkpoint) so results are never carried over. Checkpoint files are not distributed, so every user computes against their own inputs.

## Reproducibility

The LOPF is solved at full hourly resolution (8,760 time steps); temporal down-sampling is not supported (see [Limitations](#limitations)). Results are guaranteed against the pinned environment and the tagged release, and every results cell carries an internal validation suite that checks its outputs against locked reference values before they are used.

## Development status and roadmap

PyPSA-IMG is under active development. Version 1.0 (tagged `v1.0-paper`) corresponds to the published paper and reproduces its results exactly against the pinned environment. The framework is intended to grow beyond the single-site case study; planned directions include:

- **Multi-site cluster optimisation**, extending the single-site model to co-optimise several industrial sites with shared assets and inter-site energy exchange.
- **Grid-connected scenarios**, as a configurable alternative to the fully islanded design.
- **Additional technologies and support mechanisms**, broadening the component library and the Economic Viability Roadmap levers.

Each release is tagged and archived, so results remain reproducible against the specific version that produced them. Suggestions and contributions toward these directions are welcome (see [Contributing and contact](#contributing-and-contact)).

## Notebook structure

<details>
<summary>The model is a single notebook of 60 numbered cells across 15 sections (click to expand)</summary>

Run the cells in order from the top. After the environment-setup cell (Cell 1), restart the runtime once and run again from the top so the pinned versions load. Sections are interleaved with validation and diagnostic cells that reconcile results against locked reference values.

1. **Computational environment**: setup and dependency pinning, library imports and verification, central configuration (`ProjectConfig`), logging, and random-seed control.
2. **Study site**: location maps.
3. **Wind resource data acquisition**: Copernicus Climate Data Store (CDS) API client, authentication, ERA5 100 m retrieval, and dataset loading.
4. **Wind resource characterisation**: site wind-speed extraction, seasonal and monthly variability, power curve and net capacity factor, Weibull cross-validation, and the capacity-factor distribution.
5. **Network construction**: PyPSA network initialisation, technology scenario configuration, buses and snapshots, and the fixed electricity and heat demand loads.
6. **Baseline case (status quo)**: grid import plus natural-gas boiler; LOPF solve, techno-economic KPIs, weekly dispatch, and schematic.
7. **Proposed case (wind-hydrogen-CHP microgrid)**: master parameters (single source of truth), component build, and the full-resolution LOPF solve.
8. **Physics and accounting validation**: CHP heat-output coupling, energy-balance and islanding checks, binding-bounds diagnostics, and post-solve lifetime checks.
9. **Proposed KPIs, carbon, and master results**: headline cost and hydrogen figures, physical diagnostics, and carbon reduction on operational, total, and net bases.
10. **Independent validation suite**: first-principles KPI sanity, price-realism, and green-hydrogen checks.
11. **Dispatch and schematic figures**: representative low-, mid-, and high-wind weeks; the three-bus microgrid schematic.
12. **Techno-economic and viability analysis**: discounted cash flow, NPV and IRR, the Economic Viability Roadmap, an EVR sanity suite, and a sensitivity/WACC stress test.
13. **Life-cycle assessment**: embodied against operational deep-dive, carbon payback, and carbon return on investment.
14. **Monte Carlo and tornado analysis**: methodology schematic, the Monte Carlo engine, one-at-a-time tornado ranking, a whole-pipeline sanity check, and results tables and figures.
15. **Cost-CO₂ trade-off**: methodology schematic, the carbon-price-sweep engine, a sanity check, the frontier figure and tables, and the marginal abatement cost and techno-economic wall.

The full-resolution LOPF solve and the Monte Carlo analysis dominate runtime; see [Long runs and checkpoints](#long-runs-and-checkpoints).

</details>

## Limitations

This is a research tool released to support an associated publication; users should understand its assumptions before relying on it.

- **The Economic Viability Roadmap is a proposed policy analysis, not modelled physics.** It quantifies how far contingent support levers would move the IRR, with explicit friction on each. The levers are illustrative and are not a forecast or a financing plan.
- **Full-resolution only.** Down-sampling is deliberately not offered; coarser steps under-represent low-wind periods and understate both the reliability challenge and the true cost.
- **Hydrogen-tank embodied factor.** The embodied-carbon factor for the hydrogen storage tank uses the published range for the more widely characterised tank type, because a per-capacity factor for the specific stationary high-pressure type is not available in a citable form (documented in the LCA cell). Its effect is immaterial (under 4% of embodied carbon), but users substituting their own storage technology should revisit it.
- **Wind availability.** Scheduled and unscheduled turbine downtime is distinct from the gross-to-net loss stack and is not separately applied in this version; the net capacity factor reflects the loss stack described in the wind-resource cells.
- **Single-site specificity.** Results are specific to the study location, its wind resource, and its demand profile. Applying the model elsewhere requires substituting the wind data and demand series and re-running from scratch.

## Contributing and contact

Contributions and feedback are welcome:

- **Bugs and questions**: open an issue on the [issue tracker](../../issues).
- **Improvements and site adaptations**: pull requests are welcome. If you adapt the model to another site, a note on what worked and what needed changing genuinely helps improve later versions.
- **Contact**: Ammar Ahmed Wafad. For correspondence relating to the paper, please use the contact details in the [publication](https://doi.org/10.1016/j.esd.2026.102133).

## Citation

If you use this model, please cite both the paper and the software.

**Paper**

```
Wafad, A. A., Sher, F., Khzouz, M., & Ioannou, A. (2026). Integrated wind hydrogen
multicarrier microgrids for industrial decarbonisation and net zero transition.
Energy for Sustainable Development, 95, 102133.
https://doi.org/10.1016/j.esd.2026.102133
```

**Software**

```
Wafad, A. A. (2026). PyPSA-IMG: An open optimisation model for the design and
assessment of islanded, hydrogen-integrated microgrids (v1.0) [Software]. Zenodo.
https://doi.org/10.5281/zenodo.XXXXXXX
```

A `CITATION.cff` file is included, so GitHub's **"Cite this repository"** button generates these entries automatically.

## License

The code is released under the **MIT License** (see [LICENSE](LICENSE)): anyone may use, copy, modify, and distribute it, including commercially, provided the copyright notice is retained. Copyright is held by the author.

Input data is not covered by this licence. The wind resource derives from the Copernicus Climate Data Store (ERA5) and remains subject to the Copernicus licence terms; the industrial demand profile is provided in a representative form for reproducibility. Energy prices, greenhouse-gas conversion factors, the carbon price, and technology cost catalogues are obtained from their respective providers under those providers' own terms and are not redistributed here. Users redistributing or adapting any dataset are responsible for complying with the terms attached to the original source.

## Nomenclature and abbreviations

<details>
<summary>Consolidated list of abbreviations and symbols (click to expand)</summary>

**Techno-economic and viability**

| Term | Definition |
|---|---|
| TEA | Techno-Economic Assessment |
| LCA | Life-Cycle Assessment |
| TAC | Total Annualised Cost (£/yr) |
| CAPEX | Capital Expenditure |
| OPEX | Operating Expenditure |
| FOM | Fixed O&M cost |
| VOM | Variable O&M cost (£/MWh) |
| CRF | Capital Recovery Factor |
| WACC | Weighted Average Cost of Capital (real) |
| NPV | Net Present Value (£) |
| IRR | Internal Rate of Return (%) |
| DCF | Discounted Cash Flow |
| LCOEn | Levelised Cost of Energy (£/MWh delivered) |
| LCOH₂ | Levelised Cost of Hydrogen (£/kg) |
| MAC | Marginal Abatement Cost (£/tCO₂e avoided) |
| EVR | Economic Viability Roadmap (proposed levers, not modelled in the LP) |
| PPA | Power Purchase Agreement |
| PW | Private-wire |
| HAR1 | Hydrogen Allocation Round 1 (UK) |
| HPBM | Hydrogen Production Business Model (UK) |
| LCHA | Low-Carbon Hydrogen Agreement |
| NZHF | Net Zero Hydrogen Fund |
| DH | District Heating |

**Uncertainty and reliability**

| Term | Definition |
|---|---|
| Monte Carlo | Repeated sampling of uncertain parameters for result distributions |
| Tornado | One-at-a-time sensitivity ranking |
| Paired non-EVR / EVR | Base and roadmap cases on the same random draw |
| LPSP | Loss of Power Supply Probability |
| VOLL | Value of Lost Load (reported separately, excluded from economics) |
| Pareto frontier | Designs where neither cost nor emissions improves without worsening the other |
| Techno-economic wall | The abatement level beyond which MAC rises sharply |

**Technologies and components**

| Term | Definition |
|---|---|
| CHP | Combined Heat and Power |
| PEM | Proton-Exchange-Membrane (electrolyser) |
| H2-ICE | Hydrogen Internal-Combustion Engine (CHP prime mover) |
| BESS | Battery Energy Storage System |
| WHR | Waste-Heat Recovery |
| HX | Heat Exchanger |
| LOPF | Linear Optimal Power Flow |
| PyPSA | Python for Power System Analysis |

**Carbon and life-cycle**

| Term | Definition |
|---|---|
| Embodied carbon | One-off manufacturing emissions |
| Operational carbon | Emissions from running the system (here the backup gas boiler) |
| EF | Emission Factor (tCO₂e per MW/MWh of capacity, or per MWh burnt) |
| Whole-life embodied | Embodied carbon including pro-rated mid-life replacements |
| Carbon payback | Years for operational abatement to repay the embodied-carbon debt |
| Net-LCA abatement | Operational abatement minus the annualised embodied burden |
| CROI | Carbon Return on Investment (lifetime CO₂e avoided per unit embodied CO₂e) |
| SMR | Steam Methane Reforming |

**Units**

| Symbol | Meaning |
|---|---|
| tCO₂e | Tonnes of CO₂-equivalent |
| MWh / MWh_th | Megawatt-hour electrical / thermal |
| MW / MW_el | Megawatt of capacity / electrical |
| O₂ / H₂ | Oxygen / hydrogen |

</details>
