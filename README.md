<div align="center">

# PyPSA-IMG: An open optimisation model for the design and assessment of islanded, hydrogen-integrated microgrids

### An islanded, hydrogen-integrated industrial microgrid modelled in PyPSA at full hourly resolution, with techno-economic, life-cycle, uncertainty, and cost-emissions trade-off analysis

[![License: MIT](https://img.shields.io/badge/License-MIT-blue.svg)](LICENSE)
[![Python 3.12](https://img.shields.io/badge/Python-3.12-3776AB.svg?logo=python&logoColor=white)](https://www.python.org/)
[![PyPSA 0.20.1](https://img.shields.io/badge/PyPSA-0.20.1-1f77b4.svg)](https://pypsa.org/)
[![Solver: GLPK](https://img.shields.io/badge/Solver-GLPK-6aa84f.svg)](https://www.gnu.org/software/glpk/)
[![DOI](https://img.shields.io/badge/DOI-pending-lightgrey.svg)](#citation)
[![Paper](https://img.shields.io/badge/Paper-in%20review-orange.svg)](#citation)

<!--
  Banner image. Drop the microgrid schematic (PNG) into docs/ or figures/ and
  point the src below at it, e.g. docs/microgrid_schematic.png. It renders as a
  full-width banner under the title. A width of ~820px reads well on GitHub.
-->
<!--
<img src="docs/microgrid_schematic.png" alt="Islanded wind-hydrogen-CHP microgrid schematic" width="820">
-->

</div>

---

## Overview

This repository contains the model underlying a study of an islanded wind-hydrogen-combined-heat-and-power (CHP) microgrid serving a chemical and pharmaceutical site in Teesside, United Kingdom. The model is implemented in PyPSA and solved at full hourly resolution (8,760 snapshots) as a linear optimal power flow. Beyond the core techno-economic assessment (TEA) and life-cycle assessment (LCA), it includes a Monte Carlo uncertainty analysis, a one-at-a-time sensitivity ranking, and a carbon-price sweep that traces the cost against operational-emissions frontier and its marginal abatement cost.

The repository serves as the code and data availability record for the associated publication. Its purpose is to reproduce the reported results and to allow the model to be adapted to other sites. The full publication reference is added on publication.

## Headline results

The optimised islanded microgrid comprises 18.3 MW of wind, a 2.6 MW / 10.2 MWh battery, a 4.2 MW PEM electrolyser, a 317 MWh hydrogen store, a hydrogen-fuelled CHP unit, a backup gas boiler, and waste-heat recovery. It delivers a total annualised cost of £4.53 M/yr, a levelised cost of energy of £181.85/MWh, and a levelised cost of hydrogen of £7.63/kg.

Against the grid-plus-boiler baseline, the system reduces operational emissions by 72.8 percent. On a full life-cycle basis, after accounting for the annualised embodied-carbon burden, the net reduction is 48.9 percent. The one-off embodied carbon is repaid within 7.7 years, and over the 25-year horizon the system returns approximately 2.0 tonnes of CO2e avoided per tonne invested in manufacturing (net carbon return on investment).

On its own, the islanded base case does not clear a 6 percent real hurdle rate, returning an internal rate of return of minus 0.9 percent. The Economic Viability Roadmap quantifies the support levers that would render the project viable, of which private-wire power monetisation is the dominant one; under the full roadmap the internal rate of return reaches 12.3 percent.

A carbon-price sweep traces the cost against operational-emissions frontier. It shows that operational emissions can be driven down to a near-zero floor of roughly 94 percent abatement, but only at the frontier's over-build extreme, at close to double the annualised cost. The study therefore adopts a pragmatic near-zero design that retains a small gas-boiler backup for resilience rather than pursuing that uneconomic limit.

All figures above are reproduced by running the notebook against the pinned environment. Each is documented cell by cell, with independent validation checks.

## Citation

The associated paper citation will be added on publication. A permanent archived version of this code will be deposited with Zenodo, and the DOI added here at that time. Until then, please cite this repository directly and refer back for the full reference.

```
Wafad, A. A. (2026). PyPSA-IMG: An open optimisation model for the design and assessment of islanded, hydrogen-integrated
microgrids (v1.0) [Software]. GitHub. (Zenodo DOI to follow.)
```

## Repository contents

```
.
├── <notebook>.ipynb      Main analysis notebook (59 cells; run top to bottom)
├── requirements.txt      Version-pinned Python environment
├── LICENSE               MIT License
├── README.md             This file
├── figures/              Exported publication figures (PNG and vector PDF) - coming soon
└── docs/                 Schematics and supporting images - coming soon
```

The analysis is contained in a single notebook, organised into the sections listed under Notebook structure below. Running it top to bottom reproduces every headline figure and table, each guarded by an internal validation suite.




## Requirements and environment

The model runs in a version-pinned Python environment to guarantee reproducibility of the published results. The core dependencies are PyPSA 0.20.1 (frozen, because later releases alter component cost attributes), NumPy below 2.0, pandas below 2.3, Pyomo below 6.7, and SciPy below 1.12, together with the GLPK linear-programming solver, which is invoked with pyomo set to False. Development was carried out on Google Colab using Python 3.12. Local execution is supported through the accompanying requirements file. System-level dependencies, namely GLPK and the PROJ and GEOS libraries used by the optional mapping figures, are installed through the host system's package manager.

On first import, the frozen environment emits a Pyomo deprecation notice and a number of pandas future-warnings that originate inside PyPSA 0.20.1. These are expected on every execution and have no effect on the numerical results.

## Data access

The model requires two external datasets, which the user obtains separately.

The first is the wind resource, taken from the ERA5 hourly single-level reanalysis (100 m wind components) and retrieved from the Copernicus Climate Data Store. A free Climate Data Store account and a personal API key are required. The key is supplied through a Colab secret or an environment variable named CDS_API_KEY and is never committed to the repository. The notebook downloads the study-year data for the grid cell containing the site.

The second is the industrial demand profile, an hourly electricity and heat demand series of 8,760 rows for the site, supplied as a CSV with the columns Timestamp, electricity_demand_kW, and heat_demand_kW. Users adapting the model to their own site substitute a demand file of the same format.

## Notebook structure and run order

Run the cells in order from the top. After the environment-setup cell, restart the runtime once and run again from the top, so that the pinned package versions are the ones loaded. The notebook is organised into the following sections; the per-cell listing is given in the table of contents.

1. Computational environment (setup, imports, central configuration)
2. Study site (location maps)
3. Wind resource data acquisition (Copernicus Climate Data Store, ERA5)
4. Wind resource characterisation (power curve, net capacity factor, Weibull check)
5. Network construction (PyPSA buses, snapshots, demand loads)
6. Baseline case, status quo (grid import plus gas boiler; solve, KPIs, schematic)
7. Proposed case, wind-hydrogen-CHP microgrid (master parameters, component build, LOPF)
8. Physics and accounting validation (heat coupling, energy balances, islanding, lifetimes)
9. Proposed KPIs, carbon, and master results (headline cost and hydrogen figures; carbon reduction)
10. Independent validation suite (first-principles sanity, price realism, green-hydrogen benchmark)
11. Dispatch and schematic figures (representative weeks; microgrid schematic)
12. Techno-economic and viability analysis (discounted cash flow, NPV, IRR; Economic Viability Roadmap; sanity suite; sensitivity and WACC stress test)
13. Life-cycle assessment (embodied against operational deep-dive; carbon payback; carbon return on investment)
14. Monte Carlo and tornado analysis (parametric uncertainty; sensitivity ranking; reliability)
15. Cost against operational-emissions trade-off (carbon-price sweep; Pareto frontier; marginal abatement cost and the techno-economic wall)

Two computationally heavy stages dominate runtime: the full-resolution proposed-case LOPF solve over 8,760 snapshots, and the Monte Carlo uncertainty analysis, which checkpoints to disk periodically and resumes automatically after an interruption. The Monte Carlo stage and the carbon-price sweep are provided for transparency; reproducing the published deterministic results does not require re-running them, as both restore their completed results from checkpoints where available.

## Running on Google Colab

The notebook was developed on Colab and runs there directly, but a few controls are worth knowing about before a long run.

**Downloading figures and tables.** File saving is controlled by a single flag in the configuration cell, `CONFIG.auto_download`. When it is set to `True` and the notebook is running on Colab, every figure and table is both saved to the working directory and downloaded to your machine. When it is `False` (the default), the same files are still written to the working directory but are not downloaded; you can retrieve them from the Colab file browser. Set the flag to `True` if you want the outputs pushed to your computer automatically.

**Checkpoints and Colab disconnects.** The two long stages, the Monte Carlo analysis and the carbon-price sweep, periodically save their progress to a checkpoint file so that an interrupted run can resume rather than restart. By default the checkpoint is written to a session-local folder (`./mc_checkpoints` and `./tradeoff_checkpoints`). This works, but it carries an important caveat on Colab: the session-local filesystem is erased when the runtime disconnects or is recycled, so a checkpoint saved there is lost along with it, and the run would start again from the beginning.

If you intend to run a long stage on Colab and want it to survive a disconnect, mount your own Google Drive and point the checkpoint path at it. The engine cells expose the path as a single adjustable variable near the top, with the exact Drive snippet shown in a comment beside it. In outline:

```python
from google.colab import drive
drive.mount('/content/drive')
CHECKPOINT_PATH = "/content/drive/MyDrive/your-folder/mc_checkpoint.pkl"
```

With the checkpoint on your own Drive, a disconnect is harmless: re-run the cell and it resumes from the last saved point. This is your own Drive and your own folder; the repository does not mount any drive or depend on one.

**Running the long stages.** The Monte Carlo loop is gated behind `RUN_MONTE_CARLO`, which defaults to `False`, so that a full re-run is never triggered by accident. A completed checkpoint is always loaded regardless of this flag; the flag only controls whether a fresh or partial run is allowed to proceed. Set it to `True` to run or resume the loop.

**Changing the inputs.** Checkpoints are specific to the configuration and data they were produced with. The Monte Carlo checkpoint records the run's settings and refuses to resume if they differ, and the carbon-price checkpoint is ignored if the price grid has changed. If you adapt the model to a different site or dataset, set `CHECKPOINT_RESET = True` (or delete the checkpoint file) to force a clean recomputation, so that you never see results carried over from a previous configuration. Checkpoint files are not distributed with the repository for this reason: every user computes results against their own inputs.

## Configuration and adaptation

Project-wide settings are centralised in a single configuration dataclass, covering the real WACC, the carbon price, the project lifetime, the solver, the Monte Carlo sample count, and an optional flag that controls whether figures and tables are downloaded when the notebook is run on Colab. Site-specific inputs, namely coordinates, the demand file, technology-inclusion flags, and the wind loss sub-factors, are set in clearly marked cells and documented for reuse at a different site. Numerical values that a user may reasonably change are marked in the code with an inline note.

## Reproducibility

The linear optimal power flow is solved at full hourly resolution, that is 8,760 time steps. Temporal down-sampling is not supported, because it systematically under-represents low-wind periods and would flatter both the reliability and the economics. The reported results are guaranteed against the pinned environment and the tagged release. Every results cell carries an internal validation suite that reduces the analysis to the deterministic base case and checks it against the locked reference values before the numbers are used.

## Warnings and limitations

This model is a research tool released to support an associated publication, and it carries assumptions that a user should understand before relying on it.

The economic viability roadmap is a proposed policy analysis, not modelled physics. It quantifies how far hypothetical, contingent support levers (a private-wire tariff, hydrogen production support, a capital grant, oxygen byproduct sales) would move the internal rate of return, with explicit friction applied to each. These levers are illustrative and should not be read as a forecast or a financing plan.

The optimisation is solved at full hourly resolution over the whole year. Temporal down-sampling is deliberately not offered, because coarser time steps under-represent low-wind periods and would understate both the reliability challenge and the true cost. Any adaptation that reduces the resolution will produce optimistic and unreliable results.

The embodied-carbon factor for the hydrogen storage tank is based on the published range for the more widely characterised tank type, because a per-capacity embodied factor for the specific stationary high-pressure type is not available in a citable form. This is documented in the life-cycle cell. Its effect is immaterial, at under four percent of embodied carbon, but users substituting their own storage technology should revisit it.

Wind turbine availability (scheduled and unscheduled downtime) is distinct from the gross-to-net loss stack and is not separately applied in the current version. The net capacity factor reflects the loss stack described in the wind resource cells.

The single-site results are specific to the study location, its wind resource, and its industrial demand profile. Applying the model to another site requires substituting the wind data and the demand series, and re-running from scratch rather than reusing any saved checkpoint.

## Contributing

Issues, corrections, and suggestions are welcome through the repository's issue tracker. If you build on the model or adapt it to another site, feedback on what worked and what needed changing is genuinely useful and helps improve later versions.

## License

The code in this repository is released as free software under the MIT License; see the LICENSE file. Under the MIT License anyone is free to use, copy, modify, and distribute the code, including for commercial purposes, provided the copyright notice is retained. Copyright is held by the author.

The input data is a separate matter and is not covered by this license. The wind resource is obtained from the Copernicus Climate Data Store (ERA5) under the Copernicus licence terms; the industrial demand profile, energy prices, greenhouse-gas conversion factors, carbon price, and technology cost catalogues are obtained from their respective providers under those providers' own terms of use. The repository distributes none of this data. Users obtain each dataset directly from its source, as described in the Data access section, and are responsible for complying with the terms attached to it.

## Acknowledgements

To be completed on publication.

## Nomenclature and abbreviations

This is the consolidated list of abbreviations and symbols used throughout the notebook. Per-cell abbreviation blocks have been removed in favour of this single reference.

### Techno-economic and viability

- **TEA**, Techno-Economic Assessment
- **LCA**, Life-Cycle Assessment
- **TAC**, Total Annualised Cost (£/yr)
- **CAPEX**, Capital Expenditure (up-front investment)
- **OPEX**, Operating Expenditure (annual running cost)
- **FOM**, Fixed Operation and Maintenance cost
- **VOM**, Variable Operation and Maintenance cost (£/MWh)
- **CRF**, Capital Recovery Factor (annualises a capital sum)
- **WACC**, Weighted Average Cost of Capital (real discount rate)
- **NPV**, Net Present Value (£)
- **IRR**, Internal Rate of Return (percent)
- **DCF**, Discounted Cash Flow
- **LCOEn**, Levelised Cost of Energy (£/MWh delivered)
- **LCOH₂**, Levelised Cost of Hydrogen (£/kg)
- **MAC**, Marginal Abatement Cost (£ per tonne CO₂e avoided)
- **EVR**, Economic Viability Roadmap (proposed support levers, not modelled in the LP)
- **PPA**, Power Purchase Agreement
- **PW**, Private-wire (direct electrical connection to an adjacent site)
- **HAR1**, Hydrogen Allocation Round 1 (UK green-hydrogen support auction)
- **HPBM**, Hydrogen Production Business Model (UK support scheme)
- **LCHA**, Low-Carbon Hydrogen Agreement
- **NZHF**, Net Zero Hydrogen Fund (capital-grant support)
- **DH**, District Heating

### Uncertainty and reliability

- **Monte Carlo**, repeated sampling of uncertain parameters to obtain result distributions
- **Tornado**, one-at-a-time sensitivity ranking of the outputs to each input factor
- **Paired non-EVR / EVR**, the base and roadmap cases evaluated on the same random draw
- **Fixed-capacity**, the uncertainty analysis perturbs a fixed design rather than re-sizing it
- **LPSP**, Loss of Power Supply Probability (unserved energy as a fraction of demand)
- **VOLL**, Value of Lost Load (the shedding penalty; reported separately, excluded from the economics)
- **Pareto frontier**, the set of designs for which neither cost nor emissions can improve without worsening the other
- **Weighted-sum sweep**, a carbon-price sweep that recovers the supported (convex) frontier
- **Techno-economic wall**, the abatement level beyond which the marginal abatement cost rises sharply

### Technologies and components

- **CHP**, Combined Heat and Power
- **PEM**, Proton-Exchange-Membrane (electrolyser)
- **H2-ICE**, Hydrogen Internal-Combustion Engine (the CHP prime mover)
- **BESS**, Battery Energy Storage System
- **WHR**, Waste-Heat Recovery (process heat exchanger)
- **HX**, Heat Exchanger
- **LOPF**, Linear Optimal Power Flow
- **PyPSA**, Python for Power System Analysis (the modelling framework)

### Carbon and life-cycle

- **Embodied carbon**, one-off manufacturing emissions (steel, turbines, cells, tanks)
- **Operational carbon**, emissions from running the system, here the backup gas boiler
- **EF**, Emission Factor (tCO₂e per MW or per MWh of capacity, or per MWh burnt)
- **Annualised embodied**, one-off embodied spread over each component's own life
- **Whole-life embodied**, embodied carbon including pro-rated mid-life replacements over the horizon
- **Carbon payback**, the years taken for operational abatement to repay the embodied-carbon debt
- **Net-LCA abatement**, operational abatement minus the annualised embodied burden
- **Crossover**, the year in which cumulative net emissions reach zero, that is the embodied debt is repaid
- **CROI**, Carbon Return on Investment (lifetime CO₂e avoided per unit embodied CO₂e)
- **SMR**, Steam Methane Reforming (the conventional grey-hydrogen route)

### Units

- **tCO₂e**, tonnes of CO₂-equivalent (all greenhouse gases expressed as equivalent CO₂)
- **MWh / MWh_th**, megawatt-hour electrical / megawatt-hour thermal
- **MW / MW_el**, megawatt of capacity / megawatt electrical
- **O₂ / H₂**, oxygen / hydrogen
