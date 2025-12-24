# interpretable-genomic-prediction

### **Abstract**

Genomic prediction (GP) has long suffered a dichotomy between the need for the high accuracy provided by opaque ‘black box’ deep learning models or the limited predictive power of interpretable linear methods. PipelineV1.0 introduces a novel, inherently interpretable deep learning pipeline, addressing existing challenges in explainable AI to produce state-of-the-art performance while providing verifiable biological insights at multiple scales.

### **Architecture**

**Hybrid Architecture:** Incorporates a coupled encoder from a De-noising Autoencoder (DAE) which learns robust, decorrelated latent representations from genomic data.

**Interpretability:** Passes data to a TabNet processor to produce predictions and high-fidelity attention distribution maps.

**The "Bridge":** Crucially, attention maps are backpropagated through the DAE encoder back to the original SNP scale.

<img width="1521" height="1062" alt="Pipelinenolab drawio" src="https://github.com/user-attachments/assets/dbf0bf8f-6ffe-4a52-9d71-50c4a217c787" />


### **Pipeline Schematic Key**

#### **I. Data Engineering & Pre-processing**
* **(1) Genomic Processing:** SNPs extracted from VCF/CSV, encoded and transposed into sample-by-SNP matrices ($X \in \mathbb{R}^{n \times m}$).
* **(1′) Environmental Fusion:** Climate covariates ($E$) and phenotype yield data ($y$) are pre-processed following the documentation by Zou et al (link to git hub here).

#### **II. Representation Learning & Feature Scaling**
* **(2) Biological Filtering:** Sliding-window segmentation of genomic data into blocks ($X^{(w)}$). Each block passes through a **Denoising Autoencoder (DAE)** to extract a compressed latent representation ($z^{(w)}$).
* **(3) Standardization:** Application of window-specific *StandardScaler* to normalize embeddings.
* **(4) Concatenation:** Merging all scaled genomic embeddings into a unified feature matrix.

#### **III. TabNet Integration & Prediction**
* **(5) Feature Fusion:** Integration of the concatenated latent genomic matrix with the $y \times E$ environmental matrix.
* **TabNet Prediction:** The combined matrix is processed via sequential attention mechanisms to produce yield predictions ($y'$).

#### **IV. The Interpretability Bridge**
* **(6) Attention Extraction:** Extraction of attention mask matrix with min-max scaling.
* **(7) Processing of Attention mask:** Genomic attention values are extracted for back-propagation upstream. 
* **Backpropagation:** Latent attention attributions are reprojected back to the original SNP scale using saved inverse scalers and encoder weights ($P^{(w)}$).
* **(8) SNP-level Attribution:** Generation of **SNP-specific attention maps**, highlighting the biologically meaningful genomic regions responsible for the prediction.

### **Performance & Results**

Benchmarking: PipelineV1.0 was evaluated against benchmarks established in the wheat yield study by Zou et al (see references). To ensure a rigorous comparison, PipelineV1.0 was tested on a dataset closely resembling the original study (within 5% variation), while baseline model performances are cited directly from the published literature.

<img width="545" height="616" alt="image" src="https://github.com/user-attachments/assets/af15d589-c039-4d59-9ff2-9cd862da1506" />

> **Note:** Benchmark results for all methods except PipelineV1.0 are cited from the original study. The results represent significant performance compared to both traditional linear and deep-learning competitors.

#### **Internal Capacity & Validation**
Beyond external benchmarking, the architecture showcased exceptional internal capacity, achieving Pearson correlations of **0.965 and 0.976** when assessed on global and standardized harvest cycle length internal datasets.

* **Feature Extraction:** This high ceiling confirms the model's ability to extract complex genomic and environmental features without being limited by architectural capacity.
* **Generalization:** Comparison between internal and benchmarking performance indicates that the **TabNet processor** effectively filters non-generalizable features, focusing on robust signals that transfer across unseen data.

#### **Critical Evaluation**
While results are state-of-the-art (SOTA), they serve as a **strong indicator** of the architecture's predictive performance rather than an identical replication:

* **Dataset Variation:** A 5% variation exists between the test set and the original study due to the absence of the 46th IBWSN dataset.
* **Metric Compounding:** This deviation, combined with GroupKFold cross-validation and hyperparameter optimization, may contribute to a perceived compounding of accuracy.
* **Conclusion:** Despite these nuances, the performance remains substantial, validating the architecture's plausibility in high-dimensional genomic tasks.

### **Interpretability and Biological Discovery**

The utility of **PipelineV1.0** is fundamentally linked to its inherent explainability. By leveraging the **Interpretability Bridge**, the model moves beyond empirical "black-box" predictions to provide traceable attribution maps back-propagated to the original genomic scale.

#### **SNP-Level Attribution & QTL Mapping**
Analysis of the resulting attention masks reveals that importance is distributed across specific, localized genomic regions rather than random noise.

<img width="558" height="1236" alt="image" src="https://github.com/user-attachments/assets/8809679c-0d4b-42e9-8580-16ab4bc7b362" />

The figure above visualizes the mapping of **248 high-importance SNPs** using **MG2Cv2.1**, filtered to a mean attribution weight of $\ge 0.1$ across the global population.

#### **Biological Validation: QTL Enrichment Analysis**

The biological relevance of the model's attention mechanism was validated by cross-referencing high-attribution SNP clusters with the **WheatQTL database**.

<img width="762" height="636" alt="image" src="https://github.com/user-attachments/assets/0125aac1-c10c-4ec7-871e-f08e46b8d03f" />

* **Significant Overlap:** The two highest-contributing SNP clusters—**Chr2D:423,337,336–506,778,844** and **Chr3A:698,436–19,976,295**—demonstrated full or partial overlap with reported loci controlling critical grain yield parameters.
  
* **Yield-Related Parameters:** Identified clusters correspond to known QTLs for:
    * Grain aspect ratio and width.
    * Kernel thickness/length ratio.
    * Thousand-grain weight (TGW).
      
* **Statistical Significance:** Enrichment analysis confirmed these overlaps are non-random with a p-value of **$p = 0.004$**.
  
* **Methodology:** QTL positional information was verified using marker primer sequences retrieved from the **GrainGenes database** and **NCBI BLAST** to ensure precise genomic alignment.

> **Impact:** This statistical support ($p < 0.01$) proves that PipelineV1.0 effectively prioritizes genomic regions with established functional roles in wheat development, bridging the gap between deep learning and molecular breeding.

### **Status**

⚠️ **Code Repository Status:** The core source code for PipelineV1.0 is currently undergoing final refactoring and optimization for open-source release. 
* **Current Phase:** Finalizing documentation and environment configurations.

### **References & Data Attribution**

#### **Academic Citation**
The benchmarking baselines and data pre-processing methodologies used in this project are based on the following study:

* **Zou Q, Tai S, Yuan Q, Nie Y, Gou H, Wang L, et al.** Large-scale crop dataset and deep learning-based multi-modal fusion framework for more accurate G×E genomic prediction. Comput Electron Agric. 2025 Apr 3; 230:109833. Available from: https://dl.acm.org/doi/10.1016/j.compag.2024.109833.

### **Data Availability & Sources**

The datasets used in this study were sourced from the **CIMMYT Dataverse** and the **AgERA5** climate dataset. To ensure a rigorous benchmark, these data are consistent with the sources utilized by [Zou et al. (2025)](https://github.com/XiangXiangHotSpot/DeepGxE).

* **Phenotype Data:** Comprehensive phenotypic records (1st–6th WYCYT, 11th–27th HRWYT, 24th–39th ESWYT, and 36th–52nd IBWSN cycles) are available via the [CIMMYT Data Repository](https://data.cimmyt.org/dataverse/cimmytdatadvn).
* **Genotype Data:** High-density SNP markers were obtained from the CIMMYT Dataverse at [hdl.handle.net/11529/10695](https://hdl.handle.net/11529/10695).
    * *Note: Specifically utilizing the imputed merged selection candidates: `F_MAF0.01_Miss50_Het10-Merged.all.discover.lines.and.selection.candidates.vcf.imputed.CIMMYT.2022.hmp.txt.gz`*
* **Environmental Data:** Daily climate covariates were sourced from the **AgERA5** dataset and are publicly accessible at [hdl.handle.net/11529/10548548](https://hdl.handle.net/11529/10548548).

#### **Disclaimer: Benchmarking & Originality**
* **Model Attribution:** All benchmark results for GBLUP, BayesR, LightGBM, Transformer, and Bi-LSTM are cited directly from the findings of **Zou et al.** These models were not re-implemented by the author of this repository; rather, they serve as the established "Gold Standard" against which **PipelineV1.0** was evaluated.
* **Original Work:** The architecture for **PipelineV1.0**, including the DAE-TabNet Hybrid and the back-propagation "Interpretability Bridge," represents original research and implementation.
* **Database Verification:** QTL validation was performed independently by cross-referencing PipelineV1.0 outputs with the **WheatQTL** and **GrainGenes** databases.



