# Week 22 Deliverables — Omics (BIOC3605)

> **Due**: End of Week 22 weekend (Sunday midnight)  
> **Submission**: Push to `04_Phase4_Advanced_Research/W22_Omics_BIOC3605/deliverables/`

---

## Deliverable 1: Bioinformatics Analysis Pipeline Report

### Type
**Technical Report** (2000-2500 words + Python code)

### Scenario

You are a bioinformatics analyst at a biotech company. Your team has performed RNA-seq on patient-derived organoids treated with a new cancer drug. You need to analyze the data and identify:

1. Differentially expressed genes (DEGs)
2. Enriched pathways
3. Potential drug targets

### Required Sections

**1. Introduction** (300 words)
- Cancer drug target rationale
- Organoid model advantages
- RNA-seq experimental design

**2. Methods** (500 words)
- Sequencing parameters: Illumina NovaSeq, 150 bp paired-end, 30M reads/sample
- Analysis pipeline: FastQC, Trimmomatic, STAR, featureCounts, DESeq2
- Normalization: TPM for expression, DESeq2 for DE
- Enrichment: GO, KEGG pathways

**3. Results** (800 words)

(a) QC summary (reads per sample, mapping rates)
(b) DE genes: top 10 upregulated, top 10 downregulated
(c) Volcano plot visualization
(d) GO enrichment: top 5 biological processes
(e) KEGG pathway enrichment

**4. Discussion** (300 words)
- Drug mechanism interpretation
- Potential resistance mechanisms
- Limitations

### Calculations Required (Python)

1. Calculate TPM from simulated count data
2. Perform DE analysis with fold change and p-value
3. Apply FDR correction
4. Generate volcano plot and heatmap
5. Perform GO enrichment

### Rubric

| Criterion | Points |
|-----------|--------|
| Pipeline completeness | 25 |
| DE analysis accuracy | 25 |
| Visualization quality | 20 |
| Biological interpretation | 15 |
| Code quality | 15 |
| **Total** | **100** |

---

## Deliverable 2: GWAS Analysis Report

### Type
**Technical Report** (1500 words + Python code)

### Task

Analyze a simulated GWAS dataset for type 2 diabetes. Identify:
1. Genome-wide significant SNPs
2. LD blocks and candidate causal variants
3. Functional annotation of top hits

### Calculations Required

1. χ² test per SNP
2. Bonferroni correction
3. Manhattan plot
4. QQ plot

---

## Deliverable 3: Weekly Reflection

### Type
**Research Journal** (400-600 words)

### Prompts

1. Which omics approach (genomics, transcriptomics, proteomics) is most relevant to your capstone project?

2. What computational resources would you need for large-scale omics analysis?

3. Rate your confidence on the 5 mental models.
