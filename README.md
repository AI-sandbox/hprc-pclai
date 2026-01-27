![PCLAI Banner](./figures/pclai_banner.png)


*PCLAI is a deep learning-based approach for inferring continuous population genetic structure along the genome. Instead of assigning each window to a discrete ancestry label, PCLAI predicts a continuous coordinate (e.g., in PCA space) for every genomic window, plus an uncertainty score.*

------------------
### What PCLAI does

PCLAI produces two core outputs per genomic window:

+ **Continuous coordinates per window**: a low-dimensional position (e.g., (PC1, PC2)) representing where that window lies in a reference genetic space.
+ **Uncertainty score prediction**: a per-window confidence/uncertainty value, useful for filtering or downstream QC.

PCLAI can be framed as a regression problem in any vector space. For HPRC samples we use PCA as the default surrogate for genetic distance.

### Method overview applied on HPRC samples

At a high level:

1. **First, we build a reference embedding**: we construct a PCA from a reference panel (e.g., 1000 Genomes).
2. **Next, we define genomic windows**: we split the genome into a discrete set of windows (1000 SNPs).
3. **Then, we regress continuous coordinates**: we train the PCLAI model on 1000 Genomes and then infer the genomic windows' continuous coordinate on HPRC samples in the reference space.
4. **Finally, we quantify uncertainty**: we produce an uncertainty score per genomic window alongside the coordinate.

The key point is that discrete annotation is optional: you can label regions after the fact, but the model is designed to represent the full continuous spectrum of ancestry variation, avoiding artificial hard binning.

### Output format (BED9)

We provide results in BED9, which makes the track easy to visualize in genome browsers (and supports color encoding via `itemRgb`).

| Field        | Description                                                                             |
| ------------ | --------------------------------------------------------------------------------------- |
| `chrom`      | Chromosome                                                                              |
| `chromStart` | Window start                                                                            |
| `chromEnd`   | Window end                                                                              |
| `name`       | Sample/haplotype/window identifier **+ predicted coordinate** (e.g., `(PC1,PC2)`)       |
| `score`      | Uncertainty score (e.g., MC uncertainty)                                                |
| `strand`     | `+`, `-`, or `.`                                                                        |
| `thickStart` | Usually equals `chromStart`                                                             |
| `thickEnd`   | Usually equals `chromEnd`                                                               |
| `itemRgb`    | RGB color derived from the predicted coordinate (depends on your PCA/space constructor) |

Example row (illustrative):
```
chr22  49203384  49262883  NA21144/h2/w0353 (0.435,-0.711)  104  .  49203384  49262883  255,155,255
```

**Visualization tip**: `itemRgb` lets you color each window by position in the embedding (e.g., mapping a 2D coordinate into a perceptual color space → RGB), so continuous shifts along the genome are visually apparent.

#### Optional: discretizing the output

If you need discrete ancestry calls for a downstream tool, you can discretize PCLAI outputs after inference.

One simple approach is to train a classifier on reference samples in the same space. We recommend:

+ **k-NN (k = 15) decision boundaries** over the reference embedding can convert continuous coordinates into discrete bins.

That said, a major motivation for PCLAI is that discrete LAI cannot become continuous, while PCLAI retains continuous structure by default, discretization is strictly optional and task-dependent.


## Cite

When using the PCLAI method or PCLAI outputs, please cite the following paper:

```{tex}
@article{geleta_pclai_2026,
    author = {Geleta, Margarita and Mas Montserrat, Daniel and Ioannidis, Nilah M. and Ioannidis, Alexander G.},
    title = {Point-cloud local ancestry inference: coordinate-based ancestry along the genome},
    year = {2026}
}
```