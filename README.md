# MS-GAM — Design Control Files

Discussion materials for the MS-GAM (Markov-Switching Generalized Additive Model) paper on climate-tourism response.

This repo mirrors the `design/` control layer from the working vault. The files in `design/` are the canonical chapter-level control documents for the paper. The canonical estimator specification is `design/ms-gam-core.md`; the chapter map is `design/framework-ms-gam.md`; the paper outline is `design/outline.md`.

## Reading order

If you are new to the project, read in this order:

1. `design/outline.md` — paper outline (six-section structure, nine substantive chapters)
2. `design/framework-ms-gam.md` — chapter map and top-level chain
3. `design/ms-gam-core.md` — canonical baseline specification (read this first for the math)
4. The remaining files in any order, by interest

## File roles

| File | Role |
|---|---|
| `ms-gam-core.md` | Canonical baseline estimator specification. **Source of truth** for the state process, observation equation, and input block. Contains the §0 notation glossary. |
| `outline.md` | Paper outline (six sections, nine chapters) |
| `framework-ms-gam.md` | Chapter map and top-level chain (MS-GAM inference → Chapter 5 / Chapter 6) |
| `data.md` | Chapter 3 — data and system design (estimation unit, temporal sampling, climate observation space) |
| `method.md` | Chapter 2 — conceptual framework (conceptual layer, regime process, climate memory) |
| `function-generate.md` | Chapter 4 — inference of regime-specific functions and filtered probabilities |
| `functional-comparision.md` | Chapter 5 — regime-specific functional geometry (with regime alignment rule) |
| `prediction.md` | Chapter 6 — forecasting and structural validation (M_1–M_4 + B_0–B_2) |
| `robustness.md` | Chapter 7 — structural robustness |
| `theory.md` | Unified ontology (figure-level summary) |

## Status

These files are control documents, not the paper text. They specify the empirical contract and the inference / geometry / forecasting pipeline. The paper text, tables, and figures are written against this contract.

A separate group of `idea-origin.md` / `function-prediction-formula.md` / `intro.md` exists in the local working vault and is intentionally **not** mirrored here. Those files document the pre-MS-GAM brainstorming and are kept for history only.

## Local source

These files are maintained in an Obsidian vault at `Realization/segment_glm/design/`. This GitHub repo is a public discussion mirror, not the source of truth.
