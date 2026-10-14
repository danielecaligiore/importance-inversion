# Importance Inversion Transfer (IIT) and X-CDTL Framework

This repository contains the Python implementation of the **Explainable Cross-Domain Transfer Learning (X-CDTL)** framework and the **Importance Inversion Transfer (IIT)** mechanism used in the manuscript:

> **Importance inversion identifies candidate structural anchors for cross-domain transfer across social, linguistic and biological networks**  
> Daniele Caligiore, National Research Council of Italy (CNR), 2026

The repository supports reproducibility of the experiments, analyses, and figures reported in the manuscript.

---

## 1. Overview

X-CDTL is a computational framework for identifying **candidate structural anchors**: topological descriptors that show relatively stable cross-domain relationships and can therefore support exploratory transfer between heterogeneous network domains.

The framework operates on four graph domains:

- **Social** networks
- **Molecular** networks
- **Protein** networks
- **Linguistic** networks

A common set of **twelve multi-scale topological descriptors** is extracted from the graph ensembles. The IIT mechanism combines domain-discriminative feature importance with cross-domain structural compatibility to identify a compact set of candidate structural anchors.

The experimental design distinguishes two validation paradigms:

1. **Functional validation**, in which labels are defined from domain-specific functional properties or controlled perturbations and are kept separate from structural anchor discovery.
2. **Structural verification**, which retains the original benchmark based on multivariate topological extremeness.

The functional-validation paradigm is the principal paradigm of the revised analysis. The structural-verification paradigm is retained for comparison and diagnostic purposes.

The framework should be interpreted as a **preliminary, interpretable feature-selection and cross-domain transfer heuristic**. The experiments do not establish universal organizational laws, substrate-independent invariants, causal effects, or a general predictive law of transferability.

---

## 2. Repository structure

The complete computational analysis is organized as a **single comprehensive Jupyter Notebook**:

```text
IIT-X-CDTL.ipynb
```

The notebook is divided into five sequential modules:

```text
Module 1  ─  Topological Characterization & IIT Logic
     │
     ├──────────────► Module 2  ─  Topological Profiling & Ensemble Variance
     │
     └──────────────► Module 3  ─  Cross-Domain Transfer & Stress Robustness
                                      │
                                      ├──► Module 4  ─  Predictive Validation & Transfer Gain
                                      │
                                      └──► Module 5  ─  Statistical Synthesis & Stress Analysis
```

Module 1 provides the graph pools, topological descriptors, structural rankings, IIT scores, and candidate anchors used by the downstream modules.

---

## 3. Important: selecting the validation paradigm in Module 1

Module 1 contains a global configuration variable named:

```python
VALIDATION_PARADIGM
```

The two supported settings are:

```python
VALIDATION_PARADIGM = "functional_validation"
```

or

```python
VALIDATION_PARADIGM = "structural_verification"
```

### Functional validation

Use:

```python
VALIDATION_PARADIGM = "functional_validation"
```

This setting generates and uses the **domain-specific functional labels or controlled perturbation labels** for downstream validation.

In this paradigm, the functional labels are kept separate from the topological descriptors used for structural anchor discovery. The purpose is to evaluate whether the structural anchors identified by IIT retain utility when the validation task is defined by an external functional criterion.

**Use this setting to reproduce the principal revised functional-validation analysis.**

### Structural verification

Use:

```python
VALIDATION_PARADIGM = "structural_verification"
```

This setting retains the original structural benchmark based on **multivariate topological extremeness**, defined through the 90th-percentile z-score sum.

**Use this setting to reproduce the structural-verification benchmark and related diagnostic analyses.**

### Important execution rule

Set `VALIDATION_PARADIGM` **explicitly before running Module 1** and keep the same setting throughout the downstream analysis associated with that run.

To reproduce both paradigms, execute the complete workflow separately for:

```python
"functional_validation"
```

and

```python
"structural_verification"
```

The diagnostic analyses comparing the two label paradigms are intended to document their differences. They do not establish statistical independence or prove the absence of all possible topological associations.

---

## 4. Module 1 — Topological Characterization & IIT Logic

### Purpose

Module 1 characterizes graph ensembles from the four heterogeneous domains using a common set of twelve multi-scale topological descriptors.

The module:

1. Constructs the graph ensembles and extracts the topological descriptors.
2. Performs inductive domain classification using training observations.
3. Computes descriptor importance and the consensus **Borda ranking**.
4. Evaluates pairwise structural compatibility in a common standardized representation.
5. Computes pairwise **IIT scores**.
6. Selects **eight candidate structural anchors** for downstream transfer.
7. Aggregates pairwise information to derive the global consensus candidate anchor set.

### Inductive separation

Anchor discovery is performed using **training observations only**.

Held-out observations are excluded from:

- feature-importance estimation;
- pairwise IIT calculation;
- structural anchor selection.

This separation is part of the experimental design and should be preserved when modifying or rerunning the code.

### IIT logic

The IIT procedure combines:

- domain-discriminative feature importance;
- rank-order consistency;
- pairwise structural compatibility.

The resulting pair-specific IIT scores quantify the observed compatibility of structural descriptors between source and target domains and support the selection of candidate transferable features.

### Module 1 diagnostics

The module also provides diagnostics comparing:

- `y_functional`: labels generated from domain-specific functional properties or controlled perturbations;
- `y_structural`: labels associated with the original structural-verification benchmark.

These diagnostics are descriptive. They document label distributions and task-specific structural differences but do not establish statistical independence, non-circularity, or causal effects.

---

## 5. Module 2 — Topological Profiling & Ensemble Variance

### Purpose

Module 2 provides descriptive profiling of the twelve topological descriptors across the four network domains using reproducible resampling from the graph pools generated by Module 1.

### Main analyses

The module:

1. Samples **500 graphs per domain and seed** from the Module 1 graph pools.
2. Aggregates observations in long-format representation.
3. Produces individual boxplots and a consolidated multipanel visualization for the twelve descriptors.
4. Exports publication-format graphics in **PNG** and vector **PDF** formats.
5. Computes domain-level descriptive statistics, including mean and standard deviation.
6. Provides a diagnostic comparison of selected structural metrics across functional labels.

The final diagnostic is descriptive and is not intended to establish statistical independence, non-circularity, or causal effects.

---

## 6. Module 3 — Cross-Domain Transfer & Stress Robustness

### Purpose

Module 3 evaluates cross-domain anomaly-detection transfer using the candidate structural anchors identified in Module 1.

The evaluation follows an **inductive protocol**:

- anchor selection uses training observations;
- preprocessing uses training observations;
- stress generation is applied to the target training data;
- dimensionality reduction is fitted using training observations;
- the clean held-out target test set remains unchanged until final evaluation.

### Experimental conditions

Performance is compared across three conditions:

1. **No Transfer (Aligned)** — target-only learning using the available stressed target training data.
2. **No Transfer (Augmented)** — target-only learning with target observations replicated to match the sample count of the transfer condition.
3. **X-CDTL Transfer (Aligned)** — source and stressed target training representations are combined after alignment using the selected structural anchors.

The sample-count-matched target-only condition is included to distinguish transfer effects from effects attributable solely to a larger training set.

### Stress regimes

The module evaluates four predefined stress regimes that vary:

- target-data availability;
- feature noise.

A controlled missingness component is also applied according to the experimental protocol.

### Core processing

For transfer, source and target training observations are jointly transformed into a common representation. Target-only preprocessing is performed separately for the corresponding control.

The downstream detector is an **Isolation Forest**, with optional IIT-derived feature weighting according to the implementation.

### Evaluation metrics

Performance is evaluated on the clean held-out target test set using:

- **ROC-AUC**
- **Average Precision (AP)**
- **F1 score**

Results are aggregated across:

- source-target domain pairs;
- random seeds;
- stress regimes;
- experimental scenarios.

The functional-validation paradigm uses the functional or controlled-perturbation labels generated in Module 1. The structural-verification paradigm uses the original structural labels.

---

## 7. Module 4 — Predictive Validation & Transfer Gain

### Purpose

Module 4 evaluates the relationship between pairwise IIT compatibility and empirical transfer gain.

Transfer gain is computed as:

```text
Transfer Gain = Transfer performance − No Transfer performance
```

using the metric selected for the active validation paradigm.

### Statistical analysis

The principal analysis is performed at the **domain-pair level**:

1. Seed-level transfer gains are extracted from Module 3.
2. Transfer gains are merged with the pairwise IIT scores generated by Module 1.
3. Results are averaged across seeds for each directed domain pair.
4. The association between IIT compatibility and transfer gain is assessed using **Spearman rank correlation**.
5. Statistical significance is evaluated using a **two-sided permutation test**.

The same approach is used for the exploratory intra-target analysis, where only three possible source domains are available for each target.

The module distinguishes between seed-level variability and pair-level mean effects. The resulting associations are descriptive of the evaluated experiments and do not establish a general predictive law.

---

## 8. Module 5 — Statistical Synthesis & Stress Analysis

### Purpose

Module 5 provides the final statistical synthesis of the transfer experiments generated by Module 3.

The module:

1. Synchronizes and validates Module 3 results.
2. Aggregates performance across seeds, domain pairs, scenarios, and stress regimes.
3. Performs non-parametric comparisons of matched experimental conditions.
4. Visualizes performance trajectories under increasing stress.
5. Generates seed-aware summary statistics for the Combined Stress regime.

This module is intended to characterize transfer behaviour under controlled stress conditions. It does not by itself establish causal effects, universal transferability, or a general predictive law.

---

## 9. Recommended execution order

Run the notebook sequentially.

### Main revised analysis

For the principal analysis reported in the revised manuscript:

```python
VALIDATION_PARADIGM = "functional_validation"
```

Then execute:

```text
Module 1
   ↓
Module 2
   ↓
Module 3
   ↓
Module 4
   ↓
Module 5
```

### Structural benchmark

To reproduce the structural-verification benchmark:

```python
VALIDATION_PARADIGM = "structural_verification"
```

Then rerun the workflow with the same module order.

Do not mix outputs generated under the two paradigms within a single analysis unless the corresponding comparison is explicitly intended.

---

## 10. Installation

A Python environment with the dependencies listed in `requirements.txt` is recommended.

Create and activate a virtual environment, for example:

```bash
python -m venv ml-env
source ml-env/bin/activate
```

Install the required packages:

```bash
pip install -r requirements.txt
```

The implementation uses scientific Python and graph-learning libraries required by the individual modules, including graph analysis, machine learning, and dataset-loading components.

---

## 11. Data availability

The analysis uses public datasets:

| Domain | Dataset |
|---|---|
| Social | Facebook Ego Networks (SNAP) |
| Molecular | QM9 |
| Protein | PROTEINS |
| Linguistic | Brown Corpus |

The code obtains or processes these public datasets according to the procedures implemented in Module 1.

No private human-subject dataset is required for the experiments described in this repository.

---

## 12. Reproducibility

The experimental workflow uses predefined random seeds and propagates the corresponding stochastic realization consistently through the analyses.

The revised transfer analysis evaluates:

- **10 predefined random seeds**;
- **12 directed source-target domain pairs**;
- **4 predefined stress regimes**.

For reproducibility, the same seed configuration and processing order should be retained when rerunning the analysis.

The code keeps the train/test distinction explicit: transformations and model-related quantities that depend on the data are computed from the appropriate training observations, while the clean held-out target test set is reserved for final evaluation.

---

## 13. Outputs

Depending on the executed modules, the notebook generates:

- topological descriptor summaries;
- domain-classification diagnostics;
- Borda and IIT rankings;
- candidate structural anchors;
- pairwise IIT scores;
- topological profiling figures;
- transfer-performance statistics;
- stress-trajectory visualizations;
- IIT–transfer association analyses;
- statistical synthesis tables and figures;
- publication-format graphics in PNG and PDF formats.

The exact filenames and output directories are defined in the notebook.

---

## 14. Interpretation of the results

The repository implements the computational analyses used in the manuscript. The resulting evidence should be interpreted within the scope of the tested benchmark domains, graph representations, stress conditions, and validation paradigms.

In particular:

- IIT identifies **candidate structural anchors**, not proven universal invariants.
- Transfer gains may vary across directed domain pairs and stress regimes.
- Structural verification and functional validation are distinct analytical paradigms.
- The IIT–transfer association is evaluated statistically but does not establish a universal predictive relationship.
- The experiments do not establish causal mechanisms or general transferability to arbitrary graph-classification or real-world applications.

---

## 15. Contact

**Daniele Caligiore**  
National Research Council of Italy (CNR) — Institute of Cognitive Sciences and Technologies (ISTC)

Email: daniele.caligiore@cnr.it
