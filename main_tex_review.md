# Revision plan for `main.tex` — OSM pipeline, district-comparison framing

**Decision (2026-07-09): the manuscript is rewritten around the OSM pipeline; the CEM/GIS4Graph
analysis falls entirely.** This supersedes the earlier three-option review. Line numbers refer to
the current `main.tex`. Nothing in `main.tex` has been modified yet.

**Scope update (2026-07-10): the district comparison is now the paper's main framework.** The
headline claim is that the *same behaviour repeats in every district*: the main streets — the
high-centrality ones — are the ones that flood. The city-wide 52-district analysis leads; the
central-8-districts deep dive becomes the mechanism/robustness study that explains *why* (terrain
adjustment, vulnerability, resilience), not the primary result. Concretely:

- **Lead result**: betweenness rank-biserial positive in **42/42 testable districts, I² = 0%**
  (pooled r = +0.30); closeness pooled +0.34. All district results come from Design 2 —
  one independent street graph per district; **Design 1 (stratifying the central network) is
  dropped from the manuscript** (user decision 2026-07-10).
- **Street-level form of the same claim**: pooling 498 flooded streets across districts, the
  median flooded street sits at the **75th percentile of closeness within its own district**
  (betweenness 70th, degree 60th; Wilcoxon vs uniform 50%, p ≈ 10⁻²⁹). This is the sentence a
  reader feels — use it alongside the pooled-r values.
- **Universality, not just consistency**: the meta-regression (`results/meta_regression_extended.csv`)
  shows *no* district characteristic moderates the effect — elevation mean/std, street-orientation
  entropy, circuity, distance to river, network size all have p > 0.67; the intercept (+0.343,
  p ≈ 6×10⁻⁹) is the whole story. Districts as different as Sé and Parelheiros show the same
  signature. This is the paper's strongest sentence and should appear in the abstract.
- The central study area is reframed as: "to understand the mechanism behind this universal
  pattern, we analyze the eight central districts in depth" — full metric set, terrain controls,
  keystone/resilience quantification.

---

## 1. What falls with the CEM analysis

Text, figures, and tables that describe the legacy pipeline and have no direct replacement — delete
or fully rewrite:

- **L207–221** — CEM data source, Sé Square study-area framing, GIS4Graph description and citation,
  the 1,214-street/2,971-intersection graph, the name-based crossing via pandas. Replaced by the
  OSMnx pipeline description (§3.2 below). The `\cite{CEM}` and `\cite{g4g}` references fall unless
  CEM is kept as context.
- **L306–334 (Figures `all_metrics_histo2`, `histo_non_flooded`, `histo_flooded` + discussion)** —
  histograms of the old graph. The new pipeline's distribution figure is the 9-panel violin
  (`figures/violins_all_metrics.*`), which subsumes them; the three-histogram progression can go.
- **L336–358 (Table `tbl:stats`)** — all means/medians/stds are from the old data (185 vs 1,029
  streets). Replace with the new group-comparison table (§4.1).
- **L362–441 (the entire Maps subsection)** — all six metric maps (`all_degree.jpg`, `msp.jpg`,
  `betweenness.jpg`, `mean_comm.jpg`, `mean_effi.jpg`, `vuln_efi.jpg`) show the old graph with old
  Jenks class breaks; the quoted class intervals (L389 "0.0–2.0 and 2.1–4.4") are meaningless on the
  new MSP scale (~17–45). Replacement maps from the new pipeline: the flood-probability map
  (`figures/flood_probability_map.*`) and the district choropleth (`figures/district_effect_map.*`).
  The L420 inverted-efficiency description dies with this subsection (mean efficiency is not in the
  final metric set — closeness covers it).
- **L450–465 (violin figure + discussion)** — quoted ranges ("0 and 2", "0 and 5", "0.005 and 0.01")
  are old-data. Replace figure with `figures/violins_all_metrics.*` and rewrite the walkthrough
  around the new panels (includes Vulnerability, Elevation, Absolute grade).
- **L471–490 (Table `tbl:mwu`)** — replaced by §4.1.
- **L365** — "4.89 mean degree … diameter 20 … 185 flooded against 1029 non-flooded (15.25%)" — all
  old-graph numbers.

## 2. Headline numbers reference card (old → new)

| Quantity | Old (CEM) | New (OSM) |
|---|---|---|
| Streets / intersections | 1,214 / 2,971 | **8,714 / 5,637** (street level via line graph) |
| Flood window | Jan–Mar 2019 | **full-year 2019** |
| Flood events | "1016" (unreproducible) | **1,165 citywide; 286 in study area; 282 matched (98.6% at 50 m)** |
| Flooded streets | 185 (15.25%) | **137 (1.6%)** |
| Mean street degree | 4.89 | **4.55** |
| Metric set | degree, MSP, betweenness, communicability, efficiency, vulnerability | **degree, MSP, closeness, betweenness, subgraph centrality, PageRank, vulnerability** (+ elevation, grade as covariates) |
| Spatial scope | 8 central districts only | **52 districts city-wide (42 testable, ≥5 flooded streets)** + 8-district deep dive |
| District result | — (promised at L514, never delivered) | **betweenness positive 42/42 (I² = 0%), pooled r = +0.30; closeness pooled +0.34; no district-level moderator (all p > 0.67)** |

The prevalence drop (15.25% → 1.6%) needs one explanatory sentence: the OSM network is far denser
and matching is coordinate-based at 50 m rather than name-based.

## 3. Section-by-section rewrite plan

### 3.1 Abstract (L132–134)

Full rewrite. The current abstract contradicts itself (says OSM, describes the CEM graph) and cites
the unreproducible "1016 floods". Under the district-first framing the abstract leads with the
city-wide result, not the central study area. New skeleton, all numbers verified against
`results/*.csv`:

> Across São Paulo, 1,165 flood events (2019) matched to the OSM drive network. In every one of the
> 42 testable districts, the streets that flood are the network's main streets: betweenness
> rank-biserial positive in 42/42 (pooled r = +0.30, I² = 0%), closeness pooled +0.34; the median
> flooded street is more central than 75% of the streets in its own district. No district
> characteristic — elevation, street-grid geometry, river distance, network size — moderates the
> effect. A deep dive on the
> eight central districts (8,714 streets, 137 flooded) shows the pattern survives controls for
> elevation, slope, and street attributes (closeness, MSP, degree), and quantifies the cost: the
> flooded streets are the network's keystones — removing them cuts global efficiency 4.6% versus
> 1.8% for random removal.

The title's "network keystones" framing still works, but the running claim shifts from "flooded
streets are keystones (in central SP)" to "every district concentrates floods on its main streets" —
consider whether the title should signal the city-wide universality.

### 3.2 Materials and methods (L202–298)

- **Data** (replaces L205–221): OSMnx download (cite Boeing 2017) of the drive network for the 8
  central districts + 1 km buffer; node elevations from OpenTopoData ASTER 30 m, edge grades;
  flood records from \cite{Tomas2022} for all of 2019; coordinate-based matching in UTM 23S
  (EPSG:31983) with 50 m tolerance, sensitivity checked at 25/50/100 m; match report (282/286,
  98.6%). Street-level analysis via the line graph (streets = nodes), same dual representation the
  old text already explains at L223.
- **Buffer justification** (new paragraph, preempts the boundary-effects objection): buffers of
  0/250/500/1000 m compared on three districts (`results/buffer_sensitivity.csv`); at 500 m the
  closeness rank correlation vs the 1 km reference is still 0.947 and the effect size shifts up to
  0.145; at 0 m the effect can flip sign (Vila Matilde −0.22 vs +0.66). 1 km adopted.
- **Metric table (`tab:metrics`, L226–251)**: drop Shortest Path (row is redundant with MSP), drop
  Communicability and Efficiency rows; add Closeness, Subgraph centrality (the Estrada2008 cite
  moves here — it *is* self-communicability), PageRank (add cite), keep Vulnerability with its
  Latora2004/Goldshtein2004 cites and the explicit formula V(s) = (E − E₋ₛ)/E; add one line on
  eigenvector's exclusion (computed, then excluded: localization — zero medians in both groups,
  boundary-unstable ranking, no adjusted signal).
- **District analysis** (new subsection — this is now primary methodology, not a footnote): an
  independent OSMnx street graph is built for *every* São Paulo district (1 km buffer each), so
  metrics see only the district's own network — 52 districts, 42 testable at ≥5 flooded streets.
  (This is Design 2 of the districts notebook; Design 1, stratifying the central-area network by
  district, is **dropped from the manuscript** — only 9 testable districts and its metrics
  conflate district and central-network context. Its CSVs stay in `results/` but nothing in the
  paper cites them.) Per-district MWU + rank-biserial, BH-corrected; effects pooled with
  DerSimonian–Laird random-effects; meta-regression on district characteristics (elevation
  mean/std, orientation entropy, circuity, distance to river, network size) tests whether *any*
  kind of district escapes the pattern.
- **Statistics** (extends L260–298): keep the MWU exposition but (a) rephrase L469's "passes" as
  "we reject H₀ at α = …", (b) state that scipy's tie-corrected U for the first sample is used
  (not min(U₁,U₂) as L280 currently says), (c) add rank-biserial effect size and Holm/BH
  correction, (d) add one sentence each for the deep-dive machinery: Moran's I check,
  elevation-adjusted logistic models (one centrality at a time), Poisson/NB count models with
  length offset, spatially blocked (district-grouped) cross-validation, efficiency-based
  resilience simulation.

### 3.3 Results (L299–492) — proposed structure (district-first)

The district comparison opens and closes the results; the central-area deep dive sits in the
middle as the mechanism study.

1. **The pattern, city-wide** (lead result; replaces the L514 promise): 52 districts, 42 testable;
   betweenness positive in **42/42, I² = 0%** (pooled r = +0.30); closeness largest pooled effect
   (+0.34, I² = 56%). Forest plots, effect histogram, choropleth (`district_effect_map`).
2. **The same claim at street level** — within-district percentile pooling (`local_percentiles`):
   for each of the 498 flooded streets city-wide, its percentile in its *own district's* ranking;
   under the null these are uniform on 0–100%. Observed medians: closeness **75th**, betweenness
   70th, degree 60th, PageRank 58th (Wilcoxon vs 0.5, p ≈ 10⁻²⁹ … 10⁻⁷). Robust by construction —
   each district contributes only its own ranking, so no district's size or density dominates.
   The small-multiples grid (`district_small_multiples`) makes it visible: district after district,
   each drawn with its own network colored by its own local closeness, floods sitting on the
   locally central streets. (Check the grid's district selection against the final effect
   estimates — the original pick included "failure" districts, and with betweenness at 42/42 the
   honest counterexamples may need re-choosing from the weakest closeness districts.)
3. **No district escapes it** — meta-regression: none of elevation mean/std, orientation entropy,
   circuity, distance to river, or network size moderates the effect (all p > 0.67); the pooled
   intercept (+0.343, p ≈ 6×10⁻⁹) carries everything. The behaviour is a property of street
   networks per se, not of any particular kind of district.
4. **Local vs global centrality** (`local_vs_global_rank`) — each street's closeness percentile in
   its own district vs in the whole central network: Spearman ρ = 0.53. Related but far from
   identical, so the within-district signal is not just the city-wide arterial hierarchy showing
   through. Planning implication worth one sentence: floods track *locally* central streets, which
   points at local drainage priority, not only arterial-level intervention.
5. **Deep dive: the eight central districts** (was the old paper's whole scope; now the mechanism
   section). Group comparison (§4.1 table + violins) — vulnerability leads (r = 0.343), then
   closeness/MSP (0.286), betweenness (0.265), degree (0.161); PageRank null. Moran's I caveat
   (I = 0.067, p = 0.001) pointing to the adjusted analyses.
6. **Terrain-adjusted models** — closeness OR 1.61/SD, MSP 0.59, degree 1.44, subgraph 1.24 all
   significant after controls; betweenness, PageRank, vulnerability not (vulnerability's null is
   highway-class/lanes overlap + skew, **not** terrain: ρ vs elevation = 0.006). Elevation itself
   is the strongest single predictor (OR ≈ 0.4/SD). Point: the district-level pattern is not just
   "main streets sit in valleys".
7. **Vulnerability & resilience** — flooded streets are the network's costly ones: strongest
   univariate discriminator; removing the 137 flooded streets cuts efficiency 4.6% vs 1.8% ± 0.65
   random (z = 4.35) and 2.0% top-betweenness. Quantifies what the city-wide pattern *costs*.
8. **Recurrence null** — chronic (n = 43) vs one-time (n = 94) flooded streets: topologically
   indistinguishable; defuses the "important streets get reported more" objection (limited-power
   caveat).
9. **Prediction & temporal robustness** — RF ROC-AUC 0.736 / PR-AUC 0.036 vs 0.016 chance under
   spatial CV (screening value, not street-level forecasting); contrast holds in rainy and dry
   seasons and duration-weighted.

### 3.4 Conclusions (L494–514)

Rewrite around the district-first message: **in every district tested, floods concentrate on the
main streets** — betweenness positive 42/42, the median flooded street more central than 75% of
its own district's streets, no district characteristic moderating it — and the
central-area deep dive shows why that matters (elevation-independent core of closeness/MSP/degree;
flooded streets are keystones whose loss costs 4.6% of network efficiency) plus the recurrence
null. Terrain matters (strongest single predictor) but does not explain the topological signature.
L509's drainage question is now partially answered (say so). L512's
"no standard primary key" limitation is solved by coordinate matching — replace with the real
remaining limitations (point-based flood representation, single year, no drainage-network data).
Future work: polygon-based floods, hydrology/drainage covariates, multi-year data.

## 4. Ready replacement content

### 4.1 Replacement for Table `tbl:mwu` (from `results/group_comparison_tests.csv`)

| Metric | Median flooded | Median non-fl. | rank-biserial | p (BH) |
|---|---|---|---|---|
| Vulnerability | 1.0e-4 | 2.9e-5 | **+0.343** | <10⁻⁴ |
| Mean shortest path length | 26.11 | 28.79 | −0.286 | <10⁻⁴ |
| Closeness | 0.0383 | 0.0347 | +0.286 | <10⁻⁴ |
| Betweenness | 0.0042 | 0.0017 | +0.265 | <10⁻⁴ |
| Degree | 5 | 4 | +0.161 | 0.0009 |
| Subgraph centrality | 10.71 | 8.06 | +0.115 | 0.0246 |
| PageRank | 1e-4 | 1e-4 | +0.069 | 0.164 |
| Elevation (m) | 732.0 | 752.5 | −0.430 | <10⁻⁴ |
| Absolute grade | 0.037 | 0.047 | −0.114 | 0.0246 |

(Elevation/grade rows can sit in the same table or a companion one; keeping them adjacent makes the
"terrain is the strongest single factor" point honest and visible.)

### 4.2 Figure inventory (all regenerated eigenvector-free, 2026-07-08/09)

`violins_all_metrics`, `flood_probability_map`, `district_effect_map`, `forest_betweenness`,
`forest_closeness`, `forest_mean`, `effect_distribution`, `local_percentiles`,
`district_small_multiples`, `local_vs_global_rank`, `permutation_importance`,
`resilience_curves`, `recurrence_distribution`, `monthly_events` (PNG+PDF each, in `figures/` —
note the manuscript's `\includegraphics` paths use `Figures/` with a capital F; align case).
`design_comparison` also exists but falls with Design 1 — do not include it.
A study-area figure must be regenerated for the new network extent (old `StudyArea.png` shows the
CEM extent).

Under the district-first framing the headline figures are the district ones — `district_effect_map`,
`forest_betweenness` (the 42/42 lead claim), `forest_closeness`/`forest_mean`,
`effect_distribution`, `local_percentiles` (the 75th-percentile street-level claim), and
`district_small_multiples` (the claim made visible) — and they belong in the main results, not an
appendix. `local_vs_global_rank` supports the local-vs-global discussion (main text or appendix).
All of them are generated from Design 2 data only (verified in `Flood_analysis_districts.ipynb`),
so nothing needs regenerating for the Design 1 drop — but re-check `district_small_multiples`'
district selection against the final effect estimates (see §3.3 item 2). The study-area figure
should show the whole city with the 52 districts and highlight the central 8 as the deep-dive
subset.

## 5. Fixes that carry over regardless (from the original review)

- **L517 vs L529** — acknowledgements contradict (commented CNPq thanks vs "Not applicable" while
  Funding names the same grants). Delete one.
- **L541–544** — `\bibliographystyle`/`\bibliography` commented out; no `\cite` resolves. Restore
  before submission. New must-cite additions under the OSM pipeline: OSMnx (Boeing 2017), scipy,
  geopandas/statsmodels, OpenTopoData/ASTER.
- **L202** "Mateirals and methods" → "Materials and methods" (section title!).
- **L288** "compute the the z-value" → "compute the z-value".
- **L124** "Departament of Computing" → "Department of Computing".
- **L126** "Escola de matematica aplicada" → "Escola de Matemática Aplicada".
- **L166** stray space in "\cite {Porta2006".
- **L98** bracketed short title duplicates the full title — shorten the running head.
- **L158** "critical role on transport" → "in transport".
- **L212** caption above the graphic (SN style: below) — check all figures for consistency.
- **L455–456** duplicate `\label{fig:enter-label}` — dies with the old violin figure, but don't
  recreate the pattern.
- **L522–525 data availability** — keep URL; add that the repo contains the executed notebooks
  (`Flood_analysis_OSMNX.ipynb`, `Flood_analysis_statistics_OSM.ipynb`,
  `Flood_analysis_districts.ipynb`), `compute_vulnerability.py`, `requirements.txt`, and derived
  data (`edges_centrals.csv`, `matched_events.csv`, `results/vulnerability.csv`).
- **L528 author contributions** — "Tomás made the plots" needs updating for the regenerated figures.
