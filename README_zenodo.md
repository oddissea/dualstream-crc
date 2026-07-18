# Zenodo data record — contents and how to reproduce

De-identified data artifacts for **Nasser-eddine et al.**, *"Dual-stream architecture with asymmetric clinical safety evaluation for colorectal histopathology"* (submitted to *Medical Image Analysis*).

This is the **data record** (DOI `10.5281/zenodo.21327829`). The code lives in a **companion code record** (DOI `10.5281/zenodo.21429150`), minted from the GitHub release `v1.0-media-submission` of [`github.com/oddissea/dualstream-crc`](https://github.com/oddissea/dualstream-crc). Together they reproduce every headline number **without the raw whole-slide images**, which contain patient material and are available on reasonable request only.

## Files in this data record

**Per configuration** — F4 (selected), S4, F6 (the three best by ACSS):

| File | Size | What it is | Reproduces |
|---|---|---|---|
| `<M>_head.pth` | ~8 MB | stage-2 classifier head (4096→512→3) | patch level, with the embeddings below |
| `test_embeddings.npz` | ~88 MB | 4096-d embeddings of the 6894 held-out test patches + labels (shared by F4/S4/F6 — same frozen encoder) | Table 4 (patch): acc / ACSS / CAR→NOR |
| `external_cache_<M>.pt` | ~8 GB | frozen 512-d & 4096-d patch embeddings of the additional 91-slide cohort | Table 6 (slide): AttnMIL acc / CAR→NOR |

`M ∈ {F4, S4, F6}`. Class/label order in the `.npz`: `0=ADE, 1=NOR, 2=CAR` (2412 carcinoma test patches). In the slide cache the labels are `NOR/ADE/CAR`.

## Reproduce

```bash
pip install torch numpy scikit-learn jupyterlab   # compute deps + jupyter to run the notebook

# put the files above under ./zenodo/  (or set ZENODO_RECORD in the notebook to auto-download)
jupyter lab reproduce.ipynb                    # run top to bottom
```

`reproduce.ipynb` is **self-contained**: it defines the classifier head, the Gated Attention MIL, the ACSS, and the cross-validation inline, then

- **patch level** — loads each `<M>_head.pth` over the shared `test_embeddings.npz`: F4 92.37 % / 95.92 / 5-of-2412 ; S4 93.39 % / 95.88 / 33 ; F6 92.67 % / 95.56 / 31 ;
- **slide level** — trains the AttnMIL over `external_cache_<M>.pt` (5-fold × 5 seeds): F4 92.8 ± 1.1 % / 0-of-205 ; S4 95.0 % / 1 ; F6 92.3 % / 0.

## Not included (by design)

Whole-slide `.tif` images and the patch-level image arrays are **not** in this record: they are patient data, available on reasonable request to the corresponding author, subject to HUC approval and a data-processing agreement. Everything here is a de-identified embedding or a model weight. The full from-images training pipeline (which needs the raw images and the BiT-M encoder) is available on request.

## License and citation

Code: MIT. Please cite the paper (see the repository `README.md` for the BibTeX entry) and this record's DOI.
