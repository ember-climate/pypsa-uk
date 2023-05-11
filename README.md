# PyPSA-UK

## Introduction

`PyPSA-UK` is a dataset that can be used with the `PyPSA` open energy modelling framework to analyze the mechanics of the UK power system. The model was created by Ember and described in a [briefing on UK 2030 clean power](https://ember-climate.org/insights/research/uk-gas-power-phase-out/).

## Citation and license

The recommended citation is:

`Ember. (2022). PyPSA-UK - An open energy model of the UK power sector based on the PyPSA framework. https://github.com/ember-climate/pypsa-uk`

`PyPSA-UK` is based on the [PyPSA](https://pypsa.readthedocs.io/en/latest/index.html) open energy modelling framework used by many research institutions around the world. Whenever using `PyPSA-UK`, please also give credit to the authors of `PyPSA` following the guidelines [described in the PyPSA documentation](https://pypsa.readthedocs.io/en/latest/citing.html).

[Similarly to PyPSA](https://pypsa.readthedocs.io/en/latest/introduction.html?highlight=license#licence), `PyPSA-UK` is distributed under the [MIT license](https://github.com/ember-climate/pypsa-uk/blob/release/LICENSE).

## Setup

To use the model you will need to have a basic understanding of `Python` and `conda`, an LP solver and a computer with enough memory (16 GB RAM). The number of nodes in the model was kept at a minimum to make it possible to run with the open solver [HiGHS](https://www.maths.ed.ac.uk/hall/HiGHS/).

Before you start working with `PyPSA-UK`, please follow the [installation guidelines for PyPSA](https://pypsa.readthedocs.io/en/latest/installation.html).

The basic installation steps can be summarized as follows:

1. Clone the `PyPSA-UK` repository
2. Create a new `conda` environment
3. Install an LP solver (make sure it is available in the `Python` terminal)
4. Install the `PyPSA` package through `conda`

## Getting started

To make the model as accessible as possible `PyPSA-UK` is contained in just one excel sheet and one Python file. The excel sheet contains all the input data, and the Python file is used to import them into the model, run the optimization and analyse the results.

After you complete the installation procedue, it is quite easy to get first results:

1. Open the `pypsa-uk-prototype.ipynb` file script in the `pypsa-uk` folder using your IDE.
2. Set your solver name in cell 5: `network.lopf(snapshots=network.snapshots, solver_name='highs', pyomo=False)`. 
3. Save the file and run it (if an error pops up, make sure the terminal points to the correct directory and Python can see your solver).
4. Once the optimization is finished, you can view selected results in the bottom of the Jupyter notebook file.

The notebook shows only basic outputs like electricity generation, but you there are several examples of processing results in the `PyPSA` documentation - such as plotting storage discharge profiles, SRMC, emissions, line loading etc.

## Model design

Please see [Ember's briefing on UK 2030 clean power](https://ember-climate.org/insights/research/uk-gas-power-phase-out/) for a full explanation of the assumptions behind the modelled scenarios. An excerpt is provided below.

The scenario provided with the model uses 2030 capacity and demand expectations from National Grid’s Future Energy Scenarios (Leading the Way pathway), expanding the geographical scope to cover Northern Ireland. 

The model is run for each hour of the year 2030, with 12 nodes representing the regions of the UK and connected by lines that represent the main lines in the UK's transmission network. Current and planned power station locations are used (based on DUKES data, planned hydro pumped-storage units, the REPD database, among others), with some generators being decommissioned and some upgraded. 

The input data includes 3 weather and demand profiles from ENTSO-E's Pan-European Climate Database (PECD) for the most critical climatic years: 1995, 2008 and 2009 (the baseline year). These need be switched manually in the Excel sheet.

By default interconnectors are switched off, with the UK being fully self-sufficient in terms of electricity supplies. The existing interconnector information is provided in the input data so the model can be tested with imports switched on as well.

Dispatching is optimised based on short-run marginal costs. National Grid’s fuel and CO2 price forecasts are provided in the input data, and running costs for non-fossil fuel generators were set to represent their dispatching priority in the merit order. CHP operation is not optimized but follows a fixed profile based on 2019 hourly generation. A minimum load of 40% for nuclear plants was assumed based on ENTSO-E’s ERAA input data. No ramp up/down limits were introduced due to their impact on model solving times, but these can be added in future works based on PyPSA-UK.

Please note that `PyPSA-UK` is not configured as a capacity expansion model (CEM), but rather a merit-order style dispatch model. The installed capacity for different technologies is provided manually as part of the scenario design. This decision was made to ensure that the implemented scenario is realistic and aligned with National Grid's. Running the model as a CEM means the capacity additions are set by the optimizer and do not always correspond to reality. It is possible to set the `capex` values for each technology and convert `PyPSA-UK` to a CEM.

## Changing input parameters

To change the scenario, access the Excel sheet `pypsa-uk-prototype.xlsx`.

A few categories of tabs can be seen:

- `load` - demand profiles for each region. The climatic year can be changed by switching  between the `Climatic year - YYYY` columns in the formulas for hourly demand values.
- `pv`, `wind`, `wind_offshore`, `chp` - solar, wind and chp generation profiles for all hours of the year. The climatic year can be changed by switching  between the `Climatic year - YYYY` columns in the formulas for regional load factors
- `lines`, `links`, `buses`, `cbf` - technical components of the model - the definition of the nodes, lines and interconnectors
- `prices` - price assumptions for fossil fuels and hydrogen
- `gen_XXX`, `st_XXX` - capacity assumptions for each generator and storage technology. Most parameters should be self-explanatory, the `p_nom` parameter is the installed capacity, for more explanation please refer to the `PyPSA` components documentation

## Further improvements

Many improvements can be made in the `PyPSA-UK` model, some of them include:

- Heating - including heat demand in the model. This would make the CHP generation profile more realistic as currently it is independent from climatic assumptions.
- Hydrogen - including electrolyzers in the model, which would make the total generation more comparable with National Grid's approach, and would make it easier to analyse the potential of using curtailed energy for electrolysis (it is currently done outside of the optimisation model)
- EU-wide network - including a full representation of the European-wide transmission network and the cross-border flows by adding at least a single node per country, with capacity expansion projections based on e.g. the NECP's
- Thermal unit flexibility - adding ramp-up/down and minimum utilization constraints for thermal units. This was removed from the model due to the significant increase in model solving times, but it is possible to implement these constraints in `PyPSA`.
- Demand disaggregation - the UK-wide demand profiles are disaggregated into regions based on historical regional consumption data. It is possible the spatial distribution of the demand will change in the future, which should be represented in the model.
- Unit maintenance schedule - the model does not include planned and unplanned outages of generation units, which could mean that the availability of especially older plants is higher than in reality. A maintenance schedule and a random outage algorithm could be included to represent the actual working conditions of generators.
- DSR - demand side response is implemented as a generator which is a simplification, could potentially be implemented as a storage unit with variable availability

You are welcome to expand and modify `PyPSA-UK` according to your needs (honoring the license - see `Citiation and license` section). You can also submit pull requests onto the `PyPSA-UK` repository, but due to a lack of resources we cannot ensure we will be able to review and approve those.

## References

Please look at the Ember briefing cited in the `Introduction` for a detailed list of sources and a description of the methodology, the scenario design assumptions, price parameters etc.

The most significant model references are listed below:

- PyPSA: T. Brown, J. Hörsch, D. Schlachtberger, PyPSA: Python for Power System Analysis, 2018, Journal of Open Research Software, 6(1), arXiv:1707.09913, DOI:10.5334/jors.188
- National Grid Future Energy Scenarios 2022: https://www.nationalgrideso.com/future-energy/future-energy-scenarios
- ENTSO-E Ten-year Network Development Plan (TYNDP) 2022: https://2022.entsos-tyndp-scenarios.eu/download/
- ENTSO-E European Resource Adequacy Assessment (ERAA) 2021: https://www.entsoe.eu/outlooks/eraa/2021/eraa-downloads/index.html
- Climate Change Committee Sixth Carbon Budget (2020): https://www.theccc.org.uk/publication/sixth-carbon-budget/
- PyPSA-PL: Czyżak, P., Mańko, M., Sikorski, M., Stępień, K., Wieczorek, B. (2021). PyPSA-PL - An open energy model of the Polish power sector based on the PyPSA framework. Instrat. https://github.com/instrat-pl/pypsa-pl

## Release notes

- 0.1.0 - initial release of PyPSA-UK model with a 2030 scenario described in [Ember's briefing on UK 2030 clean power](https://ember-climate.org/insights/research/uk-gas-power-phase-out/)
