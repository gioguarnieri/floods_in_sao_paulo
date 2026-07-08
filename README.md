# Floods in São Paulo

Street-level analysis of the 2019 flood occurrences in São Paulo, relating flooding to the
topology of the street network. The core question: **do flooded streets occupy distinctive
positions in the network, and does that signal survive controlling for terrain?**

There are two generations of the pipeline in this repo:

1. **CEM cadastral network (legacy)** — `Flood_analysis_Graphics_and_statistics.ipynb` crosses the
   flood records with a street graph derived from the CEM cadastral base by street name, and runs
   the original distribution comparisons (histograms, violin plots, Mann-Whitney U).
2. **OpenStreetMap network (current)** — `Flood_analysis_OSMNX.ipynb` +
   `Flood_analysis_statistics_OSM.ipynb`, described below.

## Reproducing the OSM analysis

```
pip install -r requirements.txt
jupyter nbconvert --to notebook --execute --inplace Flood_analysis_OSMNX.ipynb          # 1. build dataset
jupyter nbconvert --to notebook --execute --inplace Flood_analysis_statistics_OSM.ipynb # 2. analysis
```

### 1. `Flood_analysis_OSMNX.ipynb` — data pipeline

- Selects the 8 central districts of São Paulo (Bela Vista, Bom Retiro, Cambuci, Consolação,
  Liberdade, República, Santa Cecília, Sé) from the CEM district shapefile, buffered by 1 km.
- Downloads the OSM drive network for that polygon (cached in `cache/`), converts it to an
  undirected graph, and adds node elevations (OpenTopoData, ASTER 30 m) and edge grades.
- Matches each 2019 flood occurrence to its nearest street edge **in meters** (UTM 23S,
  EPSG:31983), 50 m tolerance, reporting the match rate, a tolerance sensitivity table, and a
  street-name agreement diagnostic.
- Computes edge centralities on the line graph (degree, mean shortest path length, closeness,
  betweenness, eigenvector, subgraph centrality, PageRank).
- Exports `edges_centrals.csv`, `nodes_centrals.csv`, `matched_events.csv`, and
  `central_graph.graphml`.

### 2. `Flood_analysis_statistics_OSM.ipynb` — analysis

- **Group comparison**: flooded vs non-flooded streets — violin plots, Mann-Whitney U with
  Holm/Benjamini-Hochberg correction and rank-biserial effect sizes, Spearman correlation with
  per-street event counts, Moran's I spatial-autocorrelation check.
- **Elevation-controlled regression**: logistic models of flooding on terrain (elevation, grade)
  plus street attributes and centralities; negative binomial model on event counts with street
  length as exposure offset.
- **Predictive model**: random forest / gradient boosting with district-blocked spatial
  cross-validation, permutation importances, and a flood-probability map.
- **Resilience simulation**: network-efficiency degradation when flooded streets are removed,
  versus random and betweenness-targeted baselines.
- **Temporal analysis**: seasonality, rainy vs dry season comparison, duration-weighted
  correlations.

Figures are written to `figures/`, result tables to `results/`.

## Data files

| File | Description |
| --- | --- |
| `floods.csv` | One row per flood occurrence in São Paulo in 2019 (CGE records): longitude/latitude, street name, reference, date, month, start/end time, duration. Carefully geo-referenced by our colleague Lívia — thanks! |
| `Shapes/DI*_RMSP_CEM.*` | CEM district shapefiles for the São Paulo metropolitan region (1980/1991/2000/2010); `DI2010` defines the study area. See `Shapes/DIC_DI_1980a2010_RMSP_CEM.pdf` for the data dictionary. |
| `edges_centrals.csv` | **Pipeline output.** One row per street edge of the central OSM network: geometry, OSM tags, elevation, grade, flood count/any/duration, and all centralities. |
| `nodes_centrals.csv` | **Pipeline output.** Network nodes with coordinates and elevations. |
| `matched_events.csv` | **Pipeline output.** The flood occurrences matched to a street edge (event attributes + edge id + match distance). |
| `central_graph.graphml` | **Pipeline output.** The full graph with flood and centrality attributes, used by the resilience simulation. |
| `g4g_nodes.csv` | Legacy: CEM street graph nodes; `nome_caps`/`gid` are used to cross with the flood records by name. |
| `recorte_TTI_data.dat` | Legacy: network metrics computed on the CEM graph. |
| `DI_1980a2010_RMSP_CEM.zip` | Original CEM shapefile package (same content as `Shapes/`). |

## Data sources & credits

- Flood occurrences: CGE — Centro de Gerenciamento de Emergências Climáticas, São Paulo (2019).
- District boundaries: CEM — Centro de Estudos da Metrópole.
- Street network: © OpenStreetMap contributors, via [OSMnx](https://github.com/gboeing/osmnx).
- Elevation: [OpenTopoData](https://www.opentopodata.org/) (ASTER GDEM 30 m).
