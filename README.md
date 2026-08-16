# CSCN

[Paper](https://doi.org/10.1093/bioinformatics/btag480) - [Citation](#citation)

This repository contains the official Python implementation of **Cell-Specific Causal Network (CSCN)**.

> Menghan Wang, Junya Yang, Luyao Lyu, Jiaxing Chen, *CSCN: Inference of Cell-Specific Causal Networks Using Single-Cell RNA-Seq Data*, Bioinformatics, 2026.

## Installation

Run CSCN from the repository root in a virtual environment:

```bash
python3 -m venv .venv
. .venv/bin/activate
python3 -m pip install -r requirements.txt
python3 -m cscn --help
```

## Input data

- scRNA-seq preprocessing accepts a CSV with cells as rows, genes as columns,
  and cell barcodes in the first column.
- Paired multiome preprocessing accepts a 10x HDF5 feature matrix containing
  both `Gene Expression` and `Peaks` features.
- A multiome skeleton prior is read from a per-module
  `*_allowed_pairs.csv` file with `gene_left` and `gene_right` columns and an
  optional `prior_strength` column.
- An ATAC TF prior is read from a per-module `*_tf_target_prior.csv` file with
  `tf_module_column`, `gene_module_column`, and `tf_target_score` columns.
- Joint RNA-ATAC CI uses `gene_peak_links.csv` and the source HDF5 recorded by
  paired multiome preprocessing.

## RNA-only quick start

Build gene modules, infer one graph pair per cell, and calculate CKM:

```bash
python3 -m cscn preprocess \
  --input-modality scrna \
  --module-backend wgcna \
  --analysis-mode regulatory \
  --input-csv data/expression.csv \
  --outdir work/modules

python3 -m cscn run \
  --module-dir work/modules \
  --result-dir work/results \
  --analysis-mode regulatory \
  --nmf-mode off

python3 -m cscn ckm \
  --module-dir work/modules \
  --result-dir work/results \
  --out-csv work/ckm.csv
```

`--analysis-mode regulatory --nmf-mode off` keeps graph nodes at the gene
level. For a direct run without preprocessing, use `--module-csv` instead of
`--module-dir`.

## Inference modes

These controls are independent and can be combined. External priors are off by
default, so the default configuration remains RNA-only.

| Dimension | CLI values | Default | Effect |
|---|---|---|---|
| CI alternative | `--ci-alternative two-sided` or `greater` | `two-sided` | Selects the two-sided or positive upper-tail CI test. |
| Collider conflicts | `--collider-conflict-policy prioritize_existing` or `overwrite` | `prioritize_existing` | Selects how conflicting collider proposals are resolved. |
| Multiome skeleton prior | `--use-multiome-skeleton-prior` | off | Reads per-module `*_allowed_pairs.csv` evidence. When enabled, defaults are `hard`, `weighted`, alpha `0.20`, and minimum strength `0.0`. |
| ATAC TF prior | `--tf-prior-mode none` or `atac_prior_cscn` | `none` | Uses per-module TF-target evidence; `--tf-top-k` defaults to `5`. |
| Joint RNA-ATAC CI | `--atac-ci-mode none` or `joint_rna_atac_conditioned_cscn` | `none` | Uses target-linked peak accessibility; profile mode defaults to `max` and open threshold to `0.0`. |

## Citation

If you use CSCN in your research, please cite:

```bibtex
@article{wang2026cscn,
    author  = {Wang, Menghan and Yang, Junya and Lyu, Luyao and Chen, Jiaxing},
    title   = {{{CSCN}: Inference of Cell-Specific Causal Networks Using Single-Cell RNA-Seq Data}},
    journal = {Bioinformatics},
    pages   = {btag480},
    year    = {2026},
    month   = {06},
    issn    = {1367-4811},
    doi     = {10.1093/bioinformatics/btag480},
    url     = {https://doi.org/10.1093/bioinformatics/btag480}
}
```

## License

This project is released under the MIT License.
