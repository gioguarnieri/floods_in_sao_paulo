# Floods in São Paulo

Street-level analysis of the 2019 flood occurrences in São Paulo, relating flooding to the
topology of the street network. The core questions: **do flooded streets occupy distinctive
positions in the network, does that signal survive controlling for terrain, and does it
generalize across the city?**

## Reproducing the analysis

```
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace Flood_analysis_OSMNX.ipynb          # 1. build dataset
python compute_vulnerability.py                                                         # 2. per-street vulnerability (hours; checkpointed)
jupyter nbconvert --to notebook --execute --inplace Flood_analysis_statistics_OSM.ipynb # 3. pooled analysis
jupyter nbconvert --to notebook --execute --inplace Flood_analysis_districts.ipynb      # 4. city-wide districts (hours; checkpointed)
```

### 1. `Flood_analysis_OSMNX.ipynb` — data pipeline

- Selects the 8 central districts of São Paulo (Bela Vista, Bom Retiro, Cambuci, Consolação,
  Liberdade, República, Santa Cecília, Sé) from the CEM district shapefile, buffered by 1 km.
- Downloads the OSM drive network for that polygon (cached in `cache/`), converts it to an
  undirected graph, and adds node elevations (OpenTopoData, ASTER 30 m) and edge grades.
- Matches each 2019 flood occurrence to its nearest street edge **in meters** (UTM 23S,
  EPSG:31983), 50 m tolerance, reporting the match rate, a tolerance sensitivity table, and a
  street-name agreement diagnostic.
- Computes edge centralities on the line graph: degree, mean shortest path length, closeness,
  betweenness, subgraph centrality, PageRank. (Eigenvector centrality is computed but excluded
  from the analysis: it localizes on the densest cluster, its ranking is unstable to the network
  boundary, and it carries no signal after elevation adjustment.)
- Exports `edges_centrals.csv`, `nodes_centrals.csv`, `matched_events.csv`, and
  `central_graph.graphml`.

### 2. `compute_vulnerability.py` — per-street vulnerability

Efficiency-based removal impact for every street: V(s) = (E − E₋ₛ)/E, where E is the
Latora–Marchiori global efficiency of the (unweighted) line graph. Computed with igraph,
multiprocessed, checkpointed to `results/vulnerability_partial.csv` (resumes automatically);
final table in `results/vulnerability.csv`.

### 3. `Flood_analysis_statistics_OSM.ipynb` — pooled analysis

- **Group comparison**: flooded vs non-flooded streets — violin plots, Mann-Whitney U with
  Holm/Benjamini-Hochberg correction and rank-biserial effect sizes, Spearman correlation with
  per-street event counts, Moran's I spatial-autocorrelation check.
- **Recurrence**: chronic (≥2 events) vs one-time flooded streets.
- **Elevation-controlled regression**: logistic models of flooding on terrain (elevation, grade)
  plus street attributes and centralities; Poisson/negative-binomial models on event counts with
  street length as exposure offset.
- **Predictive model**: random forest / gradient boosting with district-blocked spatial
  cross-validation, permutation importances, and a flood-probability map.
- **Resilience simulation**: network-efficiency degradation when flooded streets are removed,
  versus random and betweenness-targeted baselines.
- **Temporal analysis**: seasonality, rainy vs dry season comparison, duration-weighted
  correlations.

### 4. `Flood_analysis_districts.ipynb` — does the signature generalize city-wide?

- **Design 1**: the central-area network stratified by district (within-district tests +
  fixed-effects heterogeneity test).
- **Design 2**: an independent street network for every São Paulo district with ≥5 flood events
  (52 districts), each with its own OSM graph, metrics, and flood matching; buffer size chosen
  empirically (buffer-sensitivity analysis at 0/250/500/1000 m); per-district tests pooled by
  DerSimonian–Laird random-effects meta-analysis (forest plots, effect choropleth,
  meta-regression).
- **Design comparison**: do the two modeling choices agree? (They do: ρ = 0.56, 92% sign
  agreement.)

Figures are written to `figures/`, result tables to `results/` (per-district edge tables in
`results/districts/`).

## Data files

| File | Description |
| --- | --- |
| `floods.csv` | One row per flood occurrence in São Paulo in 2019 (CGE records): longitude/latitude, street name, reference, date, month, start/end time, duration. Carefully geo-referenced by our colleague Lívia — thanks! |
| `Shapes/DI2010_RMSP_CEM.*` | CEM 2010 district shapefile for the São Paulo metropolitan region; defines the study area and the district polygons. See `Shapes/DIC_DI_1980a2010_RMSP_CEM.pdf` for the data dictionary. |
| `edges_centrals.csv` | **Pipeline output.** One row per street edge of the central OSM network: geometry, OSM tags, elevation, grade, flood count/any/duration, and all centralities. |
| `nodes_centrals.csv` | **Pipeline output.** Network nodes with coordinates and elevations. |
| `matched_events.csv` | **Pipeline output.** The flood occurrences matched to a street edge (event attributes + edge id + match distance). |
| `central_graph.graphml` | **Pipeline output.** The full graph with flood and centrality attributes, used by the resilience simulation. |
| `results/vulnerability.csv` | **Pipeline output.** Per-street vulnerability (efficiency drop when the street is removed). |

## Manuscript

`main.tex` is the paper (Springer Nature format); `main_tex_review.md` is the running revision
plan that maps each manuscript section to the numbers and figures produced by this pipeline.

## Data sources & credits

- Flood occurrences: CGE — Centro de Gerenciamento de Emergências Climáticas, São Paulo (2019).
- District boundaries: CEM — Centro de Estudos da Metrópole.
- Street network: © OpenStreetMap contributors, via [OSMnx](https://github.com/gboeing/osmnx).
- Elevation: [OpenTopoData](https://www.opentopodata.org/) (ASTER GDEM 30 m).
