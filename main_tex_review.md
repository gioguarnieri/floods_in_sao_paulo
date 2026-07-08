# Review notes for `main.tex`

Nothing in `main.tex` was modified. Each item below points to a line number in the current file,
quotes the text at issue, and says what needs to change and why. Items are grouped by severity.

**One structural decision first.** The manuscript currently describes the *legacy* pipeline
(CEM street base + GIS4Graph, 1,214 streets, Jan–Mar 2019, name-based crossing). The repo now also
contains the rebuilt OSM pipeline (8,714 streets, full-year 2019, coordinate-based matching in
meters, corrected statistics, elevation-controlled regression, resilience simulation). You have
three options, and several flags below depend on which you pick:

- **A. Keep the CEM analysis as-is** — then fix only the internal inconsistencies (§1) and errors (§3).
- **B. Switch to the OSM pipeline** — every number in §2 changes; the paper gets stronger methods
  (match-rate reporting, effect sizes, multiple-comparison correction, terrain control).
- **C. Present both (recommended)** — keep CEM as the primary analysis and add the OSM rebuild as a
  robustness/replication section; the mean-shortest-path finding replicates, which is a strong argument.

---

## 1. Internal contradictions (must fix regardless of option)

- **L132 (abstract) vs L207+L218 (methods) — the data source contradicts itself.** The abstract says
  the street network comes from *"the OpenStreetMap project (OSM)"* and repeats *"obtained via OSM"*,
  but Methods says it was *"procured from the Center for Studies of the Metropolis \cite{CEM}"* and
  built with GIS4Graph. The 1,214-node/2,971-edge graph is the CEM/G4G one, not OSM. Under option A,
  fix the abstract to say CEM; under B/C the abstract can legitimately say OSM but the numbers change (§2).

- **L133 — "resulting in 1016 floods" is not supported by the data in the repo.** `floods.csv` has
  1,165 events for all of 2019 and **726** for the stated Jan–Mar window. The old crossing yielded 185
  flooded streets. I could not reproduce 1,016 from any combination in the repo — recheck its origin
  or replace it (726 citywide Jan–Mar; or, new pipeline: 282 matched events full-year, 167 in Jan–Mar).

- **L134 (abstract) vs L469/L492 — which metrics "passed" disagrees with the table.** The abstract
  lists *"mean shortest path, betweenness, mean efficiency, and vulnerability"* as significant; that
  matches Table `tbl:mwu`, but only under the 0.005 threshold — Degree (p = 7.1e-3) is described as
  non-significant while the conclusion (L501) says "Most tests pass". Make abstract, table, and
  conclusion use one consistent threshold and phrasing.

- **L517 vs L529 — acknowledgements contradict.** A commented-out acknowledgement thanking CNPq
  (L517) coexists with *"Acknowledgements: Not applicable."* (L529) while the same grants appear
  under Funding (L526). Delete one; "Not applicable" is odd when funding is acknowledged.

- **L541–544 — references are commented out.** `\bibliographystyle`/`\bibliography` are disabled and
  no `.bbl` is included, so none of the `\cite` commands resolve. Must be restored before submission.

---

## 2. Numbers that change if you adopt the new OSM pipeline (options B/C)

All replacement values below come from the executed notebooks and `results/*.csv` in this repo.

- **L133 — graph size**: 1,214 streets / 2,971 intersections → OSM network: **8,714 streets**
  (5,637 primal nodes), central districts + 1 km buffer.
- **L133 — flood window/count**: "January to March 2019 … 1016 floods" → full-year 2019, **1,165
  citywide events, 286 inside the study area, 282 matched (98.6% match rate at 50 m)**.
- **L133–134 — metric list**: communicability / efficiency / vulnerability are not in the new
  pipeline; it computes degree, mean shortest path length, closeness, betweenness, eigenvector,
  subgraph centrality, PageRank (all on the line graph = street level).
- **L206 — "records from January to March 2019"**: the new analysis covers all of 2019 and the
  seasonal split is a separate result (`results/seasonal_tests.csv`) — the contrast holds in both
  rainy *and* dry seasons.
- **L218 — GIS4Graph description**: under option B this whole paragraph is replaced by the
  OSMnx/coordinate-matching description (UTM 23S projection, 50 m tolerance, sensitivity at
  25/50/100 m, street-name agreement diagnostic).
- **L336–358 (Table `tbl:stats`)**: all means/medians/stds are from the old data (185 vs 1,029
  streets). New medians are in `results/group_comparison_tests.csv` (137 flooded vs 8,577
  non-flooded edges).
- **L365 — "4.89 mean degree … diameter 20 … 185 flooded against 1029 non-flooded (15.25%)"**:
  new values: mean street (line-graph) degree **4.55**, **137 flooded of 8,714 (1.6%)**. Note the
  prevalence drops dramatically because the OSM network is much denser — worth a sentence.
- **L378–441 (metric maps)**: all six map figures (`all_degree.jpg`, `msp.jpg`, `betweenness.jpg`,
  `mean_comm.jpg`, `mean_effi.jpg`, `vuln_efi.jpg`) show the old graph and old class breaks; the
  communicability/efficiency/vulnerability maps have no counterpart in the new pipeline.
- **L389 — mean-shortest-path class values "0.0–2.0 and 2.1–4.4"**: the new MSP scale runs ~17–45
  (different graph), so any quoted class intervals must be regenerated.
- **L450–465 (violin figure + discussion)**: quoted value ranges ("0 and 2", "0 and 5",
  "0.005 and 0.01") are from the old data. New violins: `figures/violins_all_metrics.{png,pdf}`
  (9 panels, includes elevation and grade).
- **L471–490 (Table `tbl:mwu`)**: every p-value changes. New table
  (`results/group_comparison_tests.csv`) with the new metric set — and note Degree is now clearly
  significant (p ≈ 6e-4) while PageRank is not. Recommend adding the **rank-biserial effect size**
  and **Holm/BH-corrected p-values** columns (reviewers increasingly expect both; see §3).
- **L492 — "We reject the null hypothesis to four of six metrics"**: recount for the new results
  (at BH-corrected 0.05: all except PageRank; at 0.005: degree, MSP, closeness, betweenness,
  eigenvector).

---

## 3. Technical/statistical issues a reviewer will likely raise (any option)

- **L420 — the efficiency description is inverted.** *"if a node has a low value of mean efficiency,
  it takes fewer steps on average to reach it"* — efficiency is the inverse of distance (L241 says so),
  so **low** efficiency means **more** steps / harder to reach. As written it contradicts Table 1.
- **L469 — "if it is lower than 0.5% (or 0.005), we consider that it passes"**: "passes" is
  ambiguous (a test "passing" usually suggests failing to reject). Rephrase as "we reject the null
  hypothesis at α = 0.005". Also justify the unusual α, or use 0.05 with a multiple-comparison
  correction — six (now nine) simultaneous tests are reported with no Holm/Bonferroni/FDR
  adjustment anywhere. The new `results/group_comparison_tests.csv` already has corrected columns.
- **No effect sizes.** The MWU section reports only p-values; with n ≈ 1,200 (or 8,700) even tiny
  differences reach significance. Rank-biserial correlations are computed in the new results and
  should be quoted alongside p-values.
- **Spatial autocorrelation is unaddressed.** Flooded streets cluster (Moran's I = 0.067,
  p = 0.001 on the new data), so MWU p-values are optimistic. At minimum add a caveat sentence;
  ideally cite the Moran's I check.
- **The confounder question is asked but answerable.** L509 poses *"Does the network's topology
  correlate with this lack of drainage?"* as future work — the new elevation-controlled logistic
  regression answers a large part of it: closeness (OR 1.61/SD), MSP (OR 0.59/SD), and degree
  (OR 1.44/SD) remain significant **after** controlling for elevation, slope, road class, length,
  and lanes, while **betweenness and eigenvector do not survive the controls**
  (`results/logit_one_centrality_at_a_time.csv`). This directly strengthens (and slightly
  nuances) the conclusions at L497–505 — the betweenness finding at L482/L497 is confounded by
  road class/terrain.
- **L504 — "streets with the highest number of floods also have the greatest potential impact"**:
  currently asserted from vulnerability values only. The new resilience simulation supports it
  quantitatively: removing the 137 flooded streets cuts network efficiency by **4.6%**, vs
  1.8% ± 0.65 for random removal (z = 4.35) and 2.0% for top-betweenness removal
  (`results/resilience_summary.csv`, `figures/resilience_curves.pdf`).
- **L514 — future work already partially done**: "extending … to a year-round analysis" is exactly
  what the new pipeline does; if you keep option A, consider rewording so the paper doesn't promise
  something the same repo already contains.
- **L512 — the "no standard primary key" limitation** is solved by coordinate-based matching in the
  new pipeline (98.6% match rate); under B/C this limitation paragraph should be rewritten.
- **L293 — normal approximation of U**: fine for these sample sizes, but the text doesn't mention
  tie correction (scipy applies it). One clause fixes it.
- **L280 — "we select the smaller value between U1 and U2"**: harmless in theory, but note scipy's
  `mannwhitneyu` reports U for the *first* sample, not min(U1,U2) — make the description match
  whatever was actually computed.

---

## 4. Typos and language (line → fix)

- **L202**: "Mateirals and methods" → "Materials and methods" (section title!).
- **L288**: "compute the the z-value" → "compute the z-value".
- **L124**: "Departament of Computing" → "Department of Computing".
- **L126**: "Escola de matematica aplicada" → "Escola de Matemática Aplicada" (capitalization + accent).
- **L166**: "\cite {Porta2006" — stray space after `\cite` (can break some styles).
- **L369**: figure file "roads_separeted.png" — misspelled ("separated"); rename file+reference or leave consistent, but flag for the figure package.
- **L212, L425, L437**: `\caption` placed *above* `\includegraphics` in these three figures but *below* in all others — SN style wants captions consistently below the graphic.
- **L98**: title duplicates the short-title in brackets verbatim — allowed, but the bracket version is meant to be a *short* running head; consider shortening.
- **L158**: "playing a critical role on transport" → "in transport".
- **L253**: "We adapt some metrics, such as the shortest path and efficiency, by taking means to fit this approach." — ambiguous "taking means"; rephrase ("by averaging over all node pairs").
- **L256**: "It is used to visualize…" — dangling subject after a sentence about plots ("Violin plots are used…").
- **L302**: "compare flooded with non-flooded" → "compare flooded with non-flooded streets".
- **L334**: "necessitating more tools" — vague; "motivating the statistical tests below".
- **L455–456**: duplicate labels `\label{fig:enter-label}` and `\label{fig:todos_violin}` on the same figure — delete the leftover `fig:enter-label`.

---

## 5. Housekeeping

- **L213/L311/etc. — `Figures/` directory is not in this repo.** All `\includegraphics` point to
  `Figures/*.png|jpg|pdf`, which exist only on the Overleaf side. The new pipeline wrote
  publication-ready replacements to `figures/` (violins, probability map, permutation importance,
  resilience curves, monthly seasonality) — note the case difference (`Figures/` vs `figures/`).
- **L258 — "available at a GitHub repository"**: give the URL (it already appears at L524) or cite
  it; also consider citing OSMnx (Boeing 2017), scipy, and geopandas under B/C since they become
  load-bearing methods.
- **L522–525 — data availability**: correct URL; if you adopt B/C, mention that the repo includes
  the executed notebooks (`Flood_analysis_OSMNX.ipynb`, `Flood_analysis_statistics_OSM.ipynb`),
  `requirements.txt`, and the derived datasets (`edges_centrals.csv`, `matched_events.csv`).
- **L528 — author contributions**: "Tomás made the plots" — if the new figures are used, update.
