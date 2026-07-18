# Dual-stream architecture with asymmetric clinical safety evaluation for colorectal histopathology

Reproducibility artifact for **Nasser-eddine et al.**, *"Dual-stream architecture with asymmetric clinical safety evaluation for colorectal histopathology"* (submitted to *Medical Image Analysis*).

A two-stage colorectal histopathology classifier (**NOR** normal / **ADE** adenoma / **CAR** carcinoma) with three contributions over the two-stage SupCon+focal baseline:

- a **dual-stream** encoder — each patch is seen at native detail *and* as a `rebinned` 3×3 context view, fused through one weight-shared BiT-M R50x1;
- the **Asymmetric Clinical Safety Score (ACSS)** — a metric that weights each confusion by its clinical cost (a carcinoma called *normal* is the worst case), with penalties elicited from the HUC Pathology Department;
- a **Gated Attention MIL** aggregator that turns frozen patch embeddings into a slide-level diagnosis.

## Everything is in one self-contained notebook

[`reproduce.ipynb`](reproduce.ipynb) defines every model, the ACSS, and the cross-validation **inline** and reproduces all headline numbers from the deposited embeddings. The only dependencies are `torch`, `numpy`, `scikit-learn` — no `timm`, no training framework, no local package to install.

| Level | F4 (selected) | S4 | F6 |
|---|---|---|---|
| **Patch** accuracy / ACSS / CAR→NOR | 92.37 % / 95.92 / 0.2 % | 93.39 % / 95.88 / 1.4 % | 92.67 % / 95.56 / 1.3 % |
| **Slide** (91-slide cohort) accuracy / CAR→NOR | 92.8 ± 1.1 % / 0 of 205 | 95.0 ± 0.5 % / 1 of 205 | 92.3 ± 1.2 % / 0 of 205 |

F4 is selected not because it is the most accurate (S4 is) but because the ACSS ranks it first: it sends the fewest carcinomas to the normal class. That trade-off — accuracy would pick S4, clinical safety picks F4 — is the point of the metric.

## Quickstart

```bash
pip install -r requirements.txt jupyterlab   # torch, numpy, scikit-learn (+ jupyter to run the notebook)

# get the artifacts: either set ZENODO_RECORD in the notebook to auto-download,
# or download them from Zenodo (DOI 10.5281/zenodo.21327829) into ./zenodo/
jupyter lab reproduce.ipynb              # run top to bottom
```

The patch level (§3/§4) needs the shared test embeddings (~88 MB) + each per-model head (~8 MB); the slide level (§5) trains the Gated Attention MIL live over the 91-slide cache (~8 GB, a few minutes).

## Repository layout

```
reproduce.ipynb     self-contained notebook: inline models + ACSS + patch (F4/S4/F6) + slide level
README.md  README_zenodo.md  requirements.txt  LICENSE (MIT)  .zenodo.json
zenodo/             deposited artifacts land here (git-ignored; see Data availability)
```

There is intentionally **no `src/` package**: the embeddings are the frozen encoder's output, so reproduction needs only the small classifier head and the attention aggregator, both defined inline in the notebook. The from-images training pipeline (which does need the encoder and the raw data) is available on request.

## Data availability

- **Code**: this repository, tag `v1.0-media-submission`, archived on Zenodo (DOI `10.5281/zenodo.YYYYYYY`).
- **Embeddings, heads and slide caches** (one shared `test_embeddings.npz` — F4/S4/F6 share the frozen encoder — plus per-model `<M>_head.pth` and `external_cache_<M>.pt`, M ∈ {F4, S4, F6}): Zenodo DOI `10.5281/zenodo.21327829`.
- **Whole-slide images and patch-level image arrays**: available on **reasonable request** to the corresponding author, subject to HUC approval and a data-processing agreement. They contain patient material and are **not** publicly redistributable — which is why this repo reproduces the results from de-identified embeddings instead.

## Citation

```bibtex
@article{nassereddine2026dualstream,
  title   = {Dual-stream architecture with asymmetric clinical safety evaluation for colorectal histopathology},
  author  = {Nasser-eddine L{\'o}pez, Fernando H. and Asensio Ramos, Andr{\'e}s and Salido, Eduardo and Westendorp Plaza, Carlos},
  journal = {Medical Image Analysis (submitted)},
  year    = {2026}
}
```

## License

Code released under the [MIT License](LICENSE). The deposited artifacts are de-identified embeddings and model weights; the underlying patient images are not included.
