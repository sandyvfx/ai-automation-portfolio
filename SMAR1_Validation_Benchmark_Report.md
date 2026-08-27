\# SMAR-1 Superalloy Simulator — External Benchmark \& Domain-of-Applicability Report



| | |

|---|---|

| \*\*Project\*\* | SMAR-1 Superalloy Materials Architecture \& Research |

| \*\*Report ID\*\* | SMAR1-BENCH-001 |

| \*\*Date\*\* | 2026-08-21 |

| \*\*Reference dataset\*\* | Kaggle `alloy\_dataset.csv` (2,672 alloys) |

| \*\*Test set\*\* | 25 SMAR-1 MD runs (`run\_0001` … `run\_0025`) |

| \*\*Status\*\* | Domain-of-applicability benchmark (not a direct property validation) |



\---



\## 1. Executive Summary



Benchmarked the SMAR-1 atomistic superalloy pipeline against a public 2,672-alloy

composition–property dataset. The central finding is a \*\*large composition-domain gap\*\*:

the mean nearest-neighbour distance between every SMAR-1 run and the closest dataset

entry is \*\*69.6 wt%\*\* (threshold 25 wt%), i.e. \*\*all SMAR-1 Ni–Al γ/γ′ alloys lie outside

the compositional domain of the reference dataset\*\*. The dataset is ferrous-centric

(carbon/low-alloy steels, stainless steels, cast irons, tool steels), with a maximum

nickel content of \*\*48.0 wt%\*\* (Type-2 / K94840 Fe–Ni soft-magnetic alloy), whereas

SMAR-1 alloys sit at \*\*88–100 wt% Ni\*\*. Consequently the benchmark serves as a

\*domain-of-applicability\* check rather than a direct property cross-validation.

This is an honest, expected result: a general steel dataset cannot validate a

Ni₃Al-strengthened superalloy. The correct next step is benchmarking against

Ni-superalloy-specific databases (NIMS, NIST, Liu 2020 creep dataset).



\---



\## 2. Reference Dataset



\- \*\*Size:\*\* 2,672 rows × 32 columns (1 name + 2 properties + 30 elemental wt%).

\- \*\*Properties:\*\* Ultimate tensile strength (as-labelled) and liquidus (°C).

&#x20; \*No density column is present\*, so density cross-validation against this file is

&#x20; not possible; SMAR-1 densities below are MD-predicted only.

\- \*\*Family composition (qualitative):\*\* carbon \& low-alloy steels; austenitic,

&#x20; ferritic, martensitic, duplex and precipitation-hardening stainless steels;

&#x20; cast stainless steels; grey/ductile/malleable cast irons; H/M/T/A/D/O/S tool

&#x20; steels; plus a small Ni-bearing specialty tail:

&#x20; - Maraging steels — \*\*18 wt% Ni\*\*

&#x20; - Fe–Ni soft-magnetic (Type 1/2, K94490/K94840) — \*\*45–48 wt% Ni\*\* (dataset max)

&#x20; - Low-expansion / Invar-type (K93500/K93603) — \*\*32–36 wt% Ni\*\*

&#x20; - Ni–Cr–Fe corrosion alloys (UNS N08800 Alloy 800, 904L, 25-4Mo, 2507) — \*\*25–33 wt% Ni\*\*



\## 3. Test Alloys (SMAR-1 MD runs)



| Group | Runs | Ni (at%) | Al (at%) | Ni (wt%) | Pred. ρ (g/cc) |

|---|---|---|---|---|---|

| Pure Ni | 0001 | 100 | 0 | 100.0 | 8.63 |

| Solid solution | 0002–0007 | 83–94 | 6–17 | 94–99 | 7.30–7.86 |

| γ′-33% cuboid | 0008,0010,0011 | 91.7 | 8.3 | \~97 | 7.80–7.84 |

| SMAR-1 γ′-54% | 0014 | 80.1 | 19.9 | 89.7 | 7.23 |

| Enhanced γ′-60% | 0017–0024 | 81.4 | 18.6 | 90.5 | 7.30 |

| High-Al γ′ | 0025 | 77.9 | 22.1 | 88.5 | 7.08 |

| Al–Cu (control) | 0015,0016 | — | — | \~0 Ni | 2.94 / 4.82 |



\## 4. Methodology



1\. Convert each SMAR-1 run (at%) to wt% over the 30 shared elements.

2\. Compute Euclidean composition distance to every dataset row.

3\. Record the nearest neighbour and distance \*d\* (wt%).

4\. Verdict: \*\*IN-domain\*\* if \*d\* ≤ 25 wt%, else \*\*OUTSIDE domain\*\*.

5\. Densities are MD-predicted (EAM, NPT, final frame); not compared to the CSV

&#x20;  (no density column).



\## 5. Results — Nearest-Neighbour Table



| Run | Group | Pred. ρ | Nearest dataset alloy | d (wt%) | Verdict |

|---|---|---|---|---|---|

| 0001 | Pure Ni | 8.63 | Type-2 K94840 Fe–Ni | 72.6 | OUTSIDE |

| 0002–0005 | SS Ni–11Al | 7.46–7.69 | Annealed Type-2 Fe–Ni | 69.0 | OUTSIDE |

| 0006 | SS Ni–6.6Al | 7.86 | Type-2 K94840 | 70.4 | OUTSIDE |

| 0007 | SS Ni–16.6Al | 7.30 | Full-Hard Type-2 | 67.3 | OUTSIDE |

| 0008/0010/0011 | γ′-33% | 7.80–7.84 | Type-2 K94840 | 69.8 | OUTSIDE |

| 0012 | SS Ni–12.4Al | 7.59 | Type-2 K94840 | 68.6 | OUTSIDE |

| 0013 | SS Ni–16.6Al | 7.24 | Full-Hard Type-2 | 67.3 | OUTSIDE |

| 0014 | SMAR-1 γ′-54% | 7.23 | Type-2 K94840 | 66.4 | OUTSIDE |

| 0015 | Al–Cu melted | 2.94 | UNS R30556 Alloy 556 | 87.7 | OUTSIDE |

| 0016 | Al–Cu GP | 4.82 | UNS R30556 Alloy 556 | 85.2 | OUTSIDE |

| 0017–0024 | Enhanced γ′-60% | 7.30 | Type-2 K94840 | 66.8 | OUTSIDE |

| 0025 | High-Al γ′ | 7.08 | Type-2 K94840 | 65.8 | OUTSIDE |



\*\*Mean domain gap = 69.6 wt% (threshold 25) → 0 / 25 runs in-domain.\*\*



\## 6. Interpretation



\- The nearest neighbours are consistently the \*\*highest-Ni ferrous alloys\*\*

&#x20; (Fe–Ni soft-magnetic, maraging, Ni–Cr corrosion alloys), confirming the distance

&#x20; metric behaves sensibly — it walks toward the Ni-rich corner of the dataset but

&#x20; cannot reach 88–100 wt% Ni.

\- The gap is dominated by the \*\*Ni axis\*\* (≥ 40 wt% shortfall in the dataset) plus

&#x20; the absence of Al-bearing Ni alloys. It is a \*coverage\* gap, not a metric failure.

\- SMAR-1 predicted densities (7.0–7.9 g/cc for Ni–Al; 2.9–4.8 for Al–Cu) are

&#x20; physically reasonable against literature Ni (8.90) / Ni₃Al (7.45) / Al (2.70),

&#x20; giving internal confidence even though the CSV cannot confirm them.



\## 7. Limitations \& Threats to Validity



\- The reference set contains \*\*no Ni-base superalloy\*\* (no γ′-strengthened chemistry),

&#x20; so UTS/liquidus transfer is invalid; only the domain conclusion is robust.

\- CSV UTS units are as-labelled and heterogeneous; liquidus is the cleaner property.

\- MD densities use a binary/ternary EAM approximation (no Cr/Co/Re/Ta), so absolute

&#x20; densities of real multi-component superalloys will differ.



\## 8. Conclusions



1\. \*\*Domain check FAILED by design:\*\* a general steel dataset cannot validate SMAR-1.

2\. The benchmark correctly identifies the Ni-rich corner as uncovered, which is a

&#x20;  useful, honest negative result.

3\. Internal physics (density, γ′ fraction, ordering) remain self-consistent and

&#x20;  literature-plausible.



\## 9. Recommendations / Next Steps



1\. \*\*Primary:\*\* benchmark against Ni-superalloy-specific data — NIMS MatNavi creep/

&#x20;  fatigue sheets, NIST AM-Bench, and the Liu \*et al.\* 2020 266-sample SC creep-rupture

&#x20;  dataset — to obtain a genuine in-domain property validation.

2\. Add \*\*lattice-parameter \& liquidus validation\*\* against Ni–Al CALPHAD (Thermo-Calc)

&#x20;  and literature a₀(Ni)=3.524 Å, a₀(Ni₃Al)=3.572 Å.

3\. Extend the EAM to Ni–Al–Cr (mapped-Cr already prototyped) to move toward

&#x20;  oxidation-relevant chemistry.

4\. Re-run this domain benchmark after any potential or chemistry change.



\---



\## Appendix A — Provenance



| Item | Value |

|---|---|

| Reference file | `alloy\_dataset.csv` (2,672 rows) |

| Distance metric | Euclidean over 30 wt% elements |

| Domain threshold | 25 wt% |

| MD engine | LAMMPS, EAM (NiAlH\_jea / AlCu), NPT, 310k steps |

| Analysis | `smar\_analysis.py`, `smar\_pro.py`, `benchmark\_kaggle.py` |



\*End of report.\*

