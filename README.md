# Spatiotemporal gene expression and cellular dynamics of the developing human heart (forcked)

This repository is an study over the Spatial Transcriptomics data of the embryonic heart developmental atlas (HDCA). 

DOI: https://doi.org/10.1038/s41588-025-02352-6

Abstract:

- we combined unbiased spatial and single-cell transcriptomics with imaging-based validation (Visium, 10× Genomics Chromium platform)
- across postconceptional weeks 5.5 to 14 to uncover the molecular landscape of human early cardiogenesis.
- high-resolution transcriptomic map of the developing human heart,
- revealing the spatial arrangements of 
  - clustering of the 76,991 high-quality cardiac cells defined 
    - 31 coarse-grained and 
    - 72 fine-grained cell states organized into distinct functional niches.
- key insights into 
  - the formation of the cardiac pacemaker-conduction system, 
  - heart valves and 
  - atrial septum, and 
  - uncover unexpected diversity among cardiac mesenchymal cells.
  - emergence of autonomic innervation and 
  - provide the first spatial account of chromaffin cells in the fetal heart.

Results (abstract): 
- 10× Genomics Visium spatial transcriptomics analysis 
  - on 16 hearts between PCWs 6 and 12, 
  - complemented by ISS of 150 selected transcripts in 4 additional hearts.

We compiled 
- a dataset of 69,114 tissue spots from 38 heart sections,
- covering all major structural components of the developing organ.

To investigate the molecular determinants of regionality, 
- we selected 17 sections encompassing at least three cardiac chambers
- and performed unsupervised clustering of the corresponding 25,208 tissue spots, 
- resulting in 23 temporally largely consistent spatial clusters with different transcriptomic profiles
-  non-negative matrix factorization on the entire Visium dataset delineated 20 spatial gene modules


## Source and Data

- Github: https://github.com/rmauron/HDCA_heart_dev
  - Code: https://github.com/rmauron/HDCA_heart_dev/tree/main/code
  - Environment: https://github.com/rmauron/HDCA_heart_dev/tree/main/environments
    - different yml
      - liana (biocondutorSS)
      - scFates
      - scVelo
      - stereoscope

- Zenodo (code): https://zenodo.org/records/15912657

- Mendeley (data):
  - part one (https://doi.org/10.17632/fhtb99mdzd.1) 
  - part two (https://doi.org/10.17632/w65jtfsvpr.1) 
  - under the following links:
    - https://data.mendeley.com/datasets/fhtb99mdzd/1 
    - https://data.mendeley.com/datasets/w65jtfsvpr/1

??? https://data.mendeley.com/datasets/bundle.js?342865b28b070babe69b

- The raw sequencing data are availbale at the European Genome-Phenome Archive (EGA) upon request at [EGAS50000001122](https://ega-archive.org/studies/EGAS50000001122) and [EGAS50000001029](https://ega-archive.org/studies/EGAS50000001029).


## Cell types

The predicted spatial localization, 
- expression of canonical cell type markers and 
- exploration of differentially expressed gene profiles 
- enabled the identification of major populations of 

1. cardiomyocytes (CMs), 
2. endothelial cells (ECs) 
3. non-mural MCs 
4. Fibroblasts (FBs), 
5. neuroblasts and neurons (NB-N), 
6. Schwann cell progenitors 
7. glial cells (SCP-GC), 

further explored by fine-grained clustering, in addition to other core cardiac cell types 

Temporal changes in coarse-grained cluster distribution corresponded to key early cardiogenesis events. 

We also identified and characterized mural cells, such as 
8. pericytes (PC) and 
9. two spatially distinct smooth muscle cell populations (OFT_SMC and CA_SMC), in addition to two clusters
(TMSB10high_C_1–2) enriched in thymosin transcripts, previously implicated in coronary vessel development

Furthermore, we detected
10. myeloid (MyC) and 
11. lymphoid cell (LyC) populations, 
12. along with two red blood cell-dominant clusters that were excluded from downstream analysis, and 
13. identified an epicardial cell (EpC) population


## Papers

1. An integrated cell atlas of the lung in health and disease - 2023.pdf
2. Best practices for single-cell analysis across modalities - 2023.pdf
3. Benchmarking atlas-level data integration in single-cell genomics - 2021.pdf
4. Current best practices in single‐cell RNA‐seq analysis: a tutorial - 2019.pdf
5. cardiogenesis in chicken - Spatiotemporal single-cell RNA sequencing of developing chicken hearts identifies interplay between cellular differentiation and morphogenesis
M Mantri, GJ Scuderi, R Abedini-Nassab, MFZ Wang… - Nature communications, 2021
6. the first published spatiotemporal atlas of human heart development - A spatiotemporal organ-wide gene expression and cell atlas of the developing human heart
M Asp, S Giacomello, L Larsson, C Wu, D Fürth, X Qian… - Cell, 2019
7. a recent report assessing cellular communities in the fetal human heart - Spatially organized cellular communities form the developing human heart
EN Farah, RK Hu, C Kern, Q Zhang, TY Lu, Q Ma… - Nature, 2024


#### See also:

https://scatlastb.readthedocs.io/en/latest/


## Resources
The analyses presented in this repository were predominantly conducted on a MacBook Pro M2 Max chip (2023), 32 GB of memory, running Ventura 13.0; however, some computations were also executed on a private server for enhanced performance and scalability.

Although an extensive effort was attributed to reproducibility, some system dependencies might slightly affect the results. Support on it is out of the scope of that study.
Find how to set up the docker container or the different environments in the [environments folder](./environments).

## Useful links
- [Nature Genetics](missing-link)
- [Interactive viewer](https://hdcaheart.serve.scilifelab.se/web/index.html)
- [biorXive](https://www.biorxiv.org/content/10.1101/2024.03.12.584577v3) (preprint)
- [Zenodo](https://zenodo.org/records/15912657)
- [Mendeley repo 1](https://data.mendeley.com/datasets/fhtb99mdzd/1) (Cellranger, Spaceranger, metadata)
- [Mendeley repo 2](https://data.mendeley.com/datasets/w65jtfsvpr/1) (R-objects)
- [EGA Visium](https://ega-archive.org/studies/EGAS50000001122) (raw sequencing data, available upon formal request)
- [EGA Single-cell](https://ega-archive.org/studies/EGAS50000001029) (raw sequencing data, available upon formal request)

## Citation
Lázár E., Mauron R., Andrusivová Ž., Foyer J., He M., Larsson L., Shakari N., Salas S. M., Avenel C., Sariyar S., Hansen J. N., Vicari M., Czarnewski P., Braun E., Li X., Bergmann O., Sylvén C., Lundberg E., Linnarsson S., Nilsson M., Sundström E., Adameyko I., Lundeberg J.. Spatiotemporal gene expression and cellular dynamics of the developing human heart. Nature Genetics (2025). [https://doi.org/10.1038/s41588-025-02352-6](https://www.nature.com/articles/s41588-025-02352-6)
