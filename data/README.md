# Data

No images or masks are stored in this repository. The splits used in every notebook are in `splits/`.

## Japanese data

| Dataset | Content | Source | License |
|---|---|---|---|
| BeachLitter v2022 | 3,500 beach photos from Yamagata Prefecture (2011–2019) with 8-class pixel masks | Sugiyama et al. (2022), [doi:10.17882/85472](https://doi.org/10.17882/85472) | CC BY-NC-SA |
| BePLi v2 | 3,722 beach photos with instance masks of plastic litter (COCO format) | Hidaka et al. (2023), [SEANOE 96963](https://www.seanoe.org/data/00858/96963/) | CC BY-NC-SA |

- The v2022 masks are mapped to the three classes used here (background, artificial, natural).
- BePLi only annotates plastic. Its natural class is pseudo-labelled with Mask2Former (notebook 02); artificial comes from the original plastic annotations.
- 1,570 BePLi images also appear in v2022 and are removed (notebook 01), leaving 2,152.

Papers to cite:

- Sugiyama, D., Hidaka, M., Matsuoka, D., Murakami, K., Kako, S. (2022). The BeachLitter dataset for image segmentation of beach litter. *Data in Brief*, 42, 108072. https://doi.org/10.1016/j.dib.2022.108072
- Hidaka, M., Matsuoka, D., Sugiyama, D., Murakami, K., Kako, S. (2022). Pixel-level image classification for detecting beach litter using a deep learning approach. *Marine Pollution Bulletin*, 175.

## Local data

150 photos from two beaches in Indonesia: Cirebon (89) and Bali (61). The Cirebon photos show staged litter arrangements. Masks were generated with Google Gemini using the prompt in [`../docs/gemini_prompt.txt`](../docs/gemini_prompt.txt). Mask colours are mapped to class indices (red → artificial, green → natural, black → background). The local data is not published in this repository.

## Split files

| File | Columns | Content |
|---|---|---|
| `v2022_split.csv` | `image`, `mask`, `artificial_pct`, `natural_pct`, `split` | 2,450 train / 350 val / 700 test, stratified by natural-litter area |
| `bepli_split.csv` | `image`, `split`, `natural_pct` | 1,506 train / 215 val / 431 test, stratified by natural-litter area (seed 42) |
| `local_configs.json` | `{config: {"train": [...], "test": [...]}}` | 6 local configurations (3 splits × 2 directions), 74 or 76 images per half |

The Japanese test splits are used only to evaluate the Mask2Former pseudo-labeller in notebook 02.
