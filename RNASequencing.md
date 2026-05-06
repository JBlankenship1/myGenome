# RNA Sequencing Implementation Steps

**Author:** Jacob Blankenship  
**Software:** HISAT2, IGV  

> **Note:** All code ran is done with my genome ID (Bc395, nicknamed "Bernard"). It is sometimes completed with a VM (which runs ubuntu) and other times is run on the University of Kentucky supercomputer MCC (any code that starts with `sbatch` is done on MCC as the execution is computationally intense). Some softwares utalized and some shell scripts that are ran (`.sh` files) are not provided in this documentation, but general descriptions of what they do will be provided. Finally, some folder or file names could be renamed in between function runnings (such as a file being ran initialy as `xyz.txt` and later renamed and run again with `x.txt`).

---

## Step 1 — Index Generation & Read Alignment

First, we build an indexed version of the Bc395 assembly for ultra-fast alignment. We then align reads from RNA extracted from *P. oryzae* strain FR13 grown in liquid culture (inCulture) as well as from RNA extracts from lesions of *P. oryzae* strain SSID16 growing in rice leaves (inPlanta).

### FR13 (inCulture) Alignment
Command to build a HISAT2 genome index comparing Bc395 to a comparison genome (FR13):

```bash
sbatch hisat2.sh Bc395_final.fasta FR13_inCulture.fastq.gz
```

### SSID16 (inPlanta) Alignment
Command to build a HISAT2 genome index comparing Bc395 to a comparison genome (SSID16):

```bash
sbatch hisat2.sh Bc395_final.fasta SSID116_inPlanta.fastq.gz
```

### Output Files
Running the commands above will return our alignment and index files when compared to Bc395. We can view these files in our IGV window:

* `SSID116_Bc395_hits.bam`
* `FR13_Bc395_hits.bam`
* `SSID116_Bc395_hits.bam.bai`
* `FR13_Bc395_hits.bam.bai`

---

## Step 2 — Visualization & Gene Prediction

We can then finally visualize the RNAseq read alignments in the IGV browser, along with the gene prediction tracks. 

### Genes with Predicted Introns

**Do the RNAseq data support the placement of the predicted introns?**
Overall, yes. Introns are rarely out of prediction from the RNAseq predictions. Many of the genes' predicted introns are matching.
>  **Image Reference:** `"RNA Sequencing"/IntronsFullySplicedOut.png`

**Are the introns spliced out 100% of the time?**
No, there is evidence of an intron not fully spliced out of the expression.
>  **Image Reference:** `"RNA Sequencing"/IntrongsNOTFullySplicedOut.png`

### Expression Profiles

Below are the findings based on evidence of expression across the different environments:

| Query | Image Reference |
|-------|-----------------|
| Genes that are only expressed in culture | `"RNA Sequencing"/InPlantaONLYExpression.png` |
| Genes that are only expressed in planta | `"RNA Sequencing"/InCultureONLYExpression.png` |
| Predicted genes with no evidence of expression | `"RNA Sequencing"/GeneWithNOEvidenceOfExpression.png` |
| Expressed genes that were not predicted | `"RNA Sequencing"/EvidenceOfExpressionWithNOGene.png` |
