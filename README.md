# Aveiro Solar Potential

**Geospatial and economic analysis of rooftop solar PV potential for Renewable Energy Communities (CER) in Aveiro, Portugal.**

Group project developed for the Seminar course (2025/2026), MSc in Data Science, University of Aveiro.

Python · GeoPandas · PySAL (spreg, esda) · PuLP (MILP) · Streamlit

---

## Overview

The project estimates the technical and economic viability of Renewable Energy Communities in the municipality of Aveiro. The analysis works at two spatial levels: the 7-digit postal code (CP7) and the distribution transformer substation (PTD) service areas of the grid operator E-Redes.

The pipeline runs in four stages:

| Notebook | Topic | Main output |
|----------|-------|-------------|
| `04` | PV production and consumption per CP7 | `comparacao_final_limpa_cp7.csv` |
| `01` | Spatial routing (buildings to PTD) and physical sizing | `dashboard_ptd_capacidade_real.gpkg` |
| `02` | Economic analysis per PTD (NPV, IRR, payback) | `analise_economica_ptd.csv` |
| `03` | Spatial econometrics (Moran, SAR, SEM) and MILP optimisation | `milp_resultados_cp7_88k.csv` |

Run order: **04, 01, 02, 03**.

## Key results

- Estimated PV production potential: **292 GWh/year**
- Validated electricity consumption (overlap area): **427 GWh/year**
- Overall self-sufficiency rate: **52.8%**
- With a EUR 5M budget (MILP): **359 of 558 CP7 areas become self-sufficient (64.3%)**, funding 112 CP7 areas for EUR 4.89M
- Global Moran's I = 0.1703 (p < 0.001): significant positive spatial autocorrelation of self-sufficiency
- LISA: 105 significant CP7 areas (32 HH, 49 LL, 18 LH, 6 HL)
- SAR preferred over SEM (AIC 6494.9 vs 6502.0)

## Methodology in brief

- **PV production:** hourly tilted-plane irradiation from PVGIS (mean 1860 kWh/m²/year). Usable roof area = building footprint x 0.40; production = irradiation x usable area x panel efficiency x performance ratio.
- **Consumption:** robust annual extrapolation per CP7 from hourly E-Redes readings (minimum 70 hourly records).
- **Spatial matching (buildings to PTD):** centroid-based spatial join, using a hybrid coverage rule: `within` first, with a `sjoin_nearest` fallback (up to 1000 m) for buildings left unmatched. This avoids the large data loss of a pure `within` join.
- **Panel packing:** `floor(area x 0.85 / 2.0 m²)` panels per building.
- **Economics:** self-consumption valued at 0.25 EUR/kWh and surplus at 0.05 EUR/kWh, 0.5%/year degradation, 25-year horizon, 7% discount rate. Indicators: NPV, IRR (numerical bisection), simple payback.
- **Optimisation:** mixed-integer linear programme (PuLP/CBC) selecting which CP7 areas to fund under a budget constraint.

Further notes are in [`docs/metodologia.md`](docs/metodologia.md) (in Portuguese).

## Repository structure

```
aveiro-solar-potential/
├── notebooks/
│   ├── 01_roteamento_espacial_ptd.ipynb
│   ├── 02_analise_economica_ptd.ipynb
│   ├── 03_econometria_espacial_milp.ipynb
│   └── 04_producao_pv_consumo_cp7.ipynb
├── dashboard/                 # Streamlit app (see dashboard/README.md)
├── docs/metodologia.md
├── data/raw/                  # input data (not versioned, see below)
├── check_env.py               # checks dependencies and input files
├── environment.yml
└── requirements.txt
```

## Data sources

Input data are not versioned in this repository because of file size. Place them in `data/raw/`.

| Data | Source |
|------|--------|
| Hourly electricity consumption per CP7 | E-Redes open data portal (public data only) |
| Hourly solar irradiation | [PVGIS, European Commission JRC](https://re.jrc.ec.europa.eu/pvg_tools/) |
| Building footprints | OpenStreetMap and Municipality of Aveiro |
| PTD service areas | E-Redes data, network Voronoi built with OSMnx |

Larger files (`edificios_aveiro.gpkg`, `serie_consumo_cp7_2024_2025_v2.csv`) are available in the original group repository's [release](https://github.com/gabrielaneves21404-rgb/Seminario/releases/tag/Dados_semin%C3%A1rio).

Only publicly available E-Redes data were used. More granular data were not available to the group. Each data source keeps its own terms of use; please check them before reusing the data. OpenStreetMap data are subject to the ODbL licence.

## Reproducing the analysis

```bash
git clone https://github.com/rebeca-gomes-de-freitas/aveiro-solar-potential.git
cd aveiro-solar-potential

# with Conda (recommended)
conda env create -f environment.yml
conda activate aveiro-cer

# or with pip
python -m venv venv
source venv/bin/activate        # Windows: venv\Scripts\activate
pip install -r requirements.txt

python check_env.py             # verify dependencies and data files
jupyter lab                     # run notebooks 04, 01, 02, 03
```

## Configurable parameters

Set at the top of each notebook.

| Parameter | Default | Notebook |
|-----------|---------|----------|
| `ETA_PAINEL` | 0.20 | 04 |
| `PR` | 0.80 | 04 |
| `FATOR_OCUPACAO_RESIDENCIAL` | 0.40 | 04 |
| `FATOR_EMPACOTAMENTO` | 0.85 | 01 |
| `AREA_PAINEL_M2` | 2.0 m² | 01 |
| `POTENCIA_PAINEL_KWP` | 0.450 kWp | 01, 02 |
| `CUSTO_POR_PAINEL_EUR` | 1200 EUR/panel | 02 |
| `TAXA_DESCONTO` | 7% | 02 |
| `HORIZONTE_ANOS` | 25 years | 02 |
| `ORCAMENTO` | EUR 5,000,000 | 03 |

## Authors

Group project, Seminar 2025/2026, MSc in Data Science, University of Aveiro.

- Rebeca Freitas
- Carolina Raposo
- Gabriela Silva

All members contributed jointly to the analysis, code and dashboard.
Original group repository: https://github.com/gabrielaneves21404-rgb/Seminario

