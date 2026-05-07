# RNA Sequencing Implementation Steps

**Author:** Jacob Blankenship  
**Software:** HISAT2, IGV
**Genome ID:** Bc395

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
**Image Reference:** ![IntronsFullySplicedOut](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/RNA%20Sequencing/IntronsFullySplicedOut.png) - Introns Fully Spliced Out

**Are the introns spliced out 100% of the time?**
No, there is evidence of an intron not fully spliced out of the expression.
**Image Reference:** ![IntronsNOTFullySplicedOut](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/RNA%20Sequencing/IntronsNOTFullySplicedOut.png) - Introns Not Fully Spliced Out

---

## Step 3 — Expression Profiles Wtih Image Reference

Below are the findings based on evidence of expression across the different environments:

![InCultureONLYExpression](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/RNA%20Sequencing/InCultureONLYExpression.png) - Genes that are only expressed in culture

![InPlantaONLYExpression](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/RNA%20Sequencing/InPlantaONLYExpression.png) - Genes that are only expressed in planta

![GeneWithNOEvidenceOfExpression.png](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/RNA%20Sequencing/GeneWithNOEvidenceOfExpression.png) - Predicted genes with no evidence of expression

![EvidenceOfExpressionWithNOGene](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/RNA%20Sequencing/EvidenceOfExpressionWithNOGene.png) - Expressed genes that were not predicted
