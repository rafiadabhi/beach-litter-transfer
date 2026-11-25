# beach-litter-transfer

Semantic segmentation of beach litter, trained on Japanese data and evaluated on photos from two Indonesian beaches (Cirebon and Bali). The code measures how well a Japan-trained model works on the local photos without any local training (zero-shot), how much it improves when local images are added to training (adaptation gain), and which of two architectures, DeepLabV3+ ResNet-101 and SegFormer MiT-B3, is the better choice.

Classes: `0` background (sand, sea, sky), `1` artificial litter, `2` natural litter (wood, branches, seaweed and other plant material), `255` ignored.

## Results

Local test images (150; Bali 61, Cirebon 89). **mIoU litter** is the mean IoU of the artificial and natural classes. Values are averaged over 3 seeds and 6 local split configurations; gains have 95% paired-bootstrap confidence intervals.

| Model | Zero-shot (A) | Adapted (B) | Gain B − A | Gain, Bali | Gain, Cirebon | Total area MAE (B) | Inference |
|---|---|---|---|---|---|---|---|
| DeepLabV3+ R101 | 0.473 | 0.584 | +0.110 [0.090, 0.132] | +0.083 | +0.162 | 1.84 pp | 27 ms |
| SegFormer MiT-B3 | 0.537 | 0.625 | +0.088 [0.073, 0.103] | +0.089 | +0.146 | 1.86 pp | 61 ms |

- SegFormer is better than DeepLab in mIoU litter both zero-shot (+0.063 [0.047, 0.081]) and adapted (+0.041 [0.035, 0.047]), and wins 18 of 18 adapted runs.
- Total litter area error does not differ significantly between the two (adapted: +0.02 pp [−0.27, 0.32]).
- A weighted ensemble of both models adds only +0.001 mIoU over SegFormer alone, below the 0.01 threshold set for a practical difference.
- **Recommended model:** SegFormer MiT-B3 trained on the Japanese data plus all 150 local images (model F).

Inference times are median milliseconds per image on a Kaggle T4 GPU at short side 512.

## Pipeline

All notebooks were run on Kaggle (GPU T4 or P100, Internet on). Each one lists its inputs at the top; set the paths in its configuration cell.

| Notebook | Purpose | Main inputs | Results folder |
|---|---|---|---|
| `01_dedup_bepli_v2022` | Remove BePLi v2 images that also appear in BeachLitter v2022 (perceptual hash + pixel check) | BeachLitter v2022, BePLi v2 | `results/01_dedup` |
| `02_pseudolabel_mask2former_bepli` | Train Mask2Former on v2022 and pseudo-label the natural class in BePLi; artificial comes from BePLi's own plastic annotations | BeachLitter v2022, BePLi v2 | `results/02_pseudolabel` |
| `03_tuning_deeplabv3plus_r101` | Optuna search of learning rate and weight decay on Japanese data | datasets, output of 02 | `results/03_04_tuning` |
| `04_tuning_segformer_b3` | Same search for SegFormer | datasets, output of 02 | `results/03_04_tuning` |
| `05_adaptation_deeplabv3plus_r101` | Zero-shot (A) vs adapted (B) runs, adaptation gain, final model F | datasets, local data, output of 02 | `results/05_06_adaptation` |
| `06_adaptation_segformer_b3` | Same for SegFormer | datasets, local data, output of 02 | `results/05_06_adaptation` |
| `07_comparison_ensemble` | DeepLab vs SegFormer tests, ensemble test, final model choice | outputs of 05 and 06, local data | `results/07_comparison` |

Design of notebooks 05 and 06:

- **A (zero-shot):** trained on the Japanese training set only (BeachLitter v2022 + BePLi v2, 3,956 images).
- **B (adapted):** the same data plus the local training half. Each batch slot holds a local image with probability 0.1; steps per epoch are the same as for A.
- **Local splits:** three random 50/50 splits stratified by site, each used in both directions (6 configurations). Every local image is a test image once per split.
- **Seeds:** 42, 43, 44. The epoch is always chosen on Japanese validation data; local test data is never used for any choice.

## Repository layout

```
notebooks/   seven notebooks, in the order they are run
data/splits/ train/val/test splits for v2022 and BePLi, and the local split configurations
results/     outputs of each notebook (CSV, JSON, figures); per-image confusion matrices in */eval
docs/        prompt used to generate the local masks
```

Images, masks, model weights and intermediate probability files are not included. See [`data/README.md`](data/README.md) for the data sources.

## License

Code: MIT (see `LICENSE`). BeachLitter v2022 and BePLi v2 are distributed by their authors under CC BY-NC-SA; models trained on them should be shared under the same terms.
