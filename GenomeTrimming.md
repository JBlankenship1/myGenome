# Gene Trimming Implementation Steps

**Author:** Jacob Blankenship  
**Software:** Trimmomatic, FastQC  
**Genome ID:** Bc395

---

## Step 1 — Download Workshop Materials

Download a compressed archive containing the files needed for class. The workshop materials contain resources needed to run our code, such as an assembly genome of a similar organism to get a BUSCO score. We access it with a tarball:

```bash
wget https://www.cs.uky.edu/~acta225/CS485/workshop-materials.tar.xz
```

Downloading the actual software to run our files on is also needed, and is retrieved with a setup shell script:

```bash
wget https://www.cs.uky.edu/~acta225/CS485/vm_soft_setup.sh
```

---

## Step 2 — Initial Quality Assessment with FastQC

### Run FastQC on Raw Reads

Run FastQC on both forward and reverse reads to assess initial quality:

```bash
fastqc Bc395_1.fq.gz Bc395_2.fq.gz -o ~/sequences
```

This will return `.html` files that can be downloaded and opened in a web browser to view. This can be done with `scp`:

```bash
scp jrbl245@jrbl245.cs.uky.edu:Bc395_1.fastqc.html .
scp jrbl245@jrbl245.cs.uky.edu:Bc395_2.fastqc.html .
```

### FastQC Results: Raw Data

The viewed HTML files include data on provided screenshots such as summary statistics and adapter content.

**Quality Issues Identified in Raw Data:**

| Issue Type | Severity | Description |
|------------|----------|-------------|
| Per base sequence content |  WARNING | Uneven base composition across read positions |
| Per sequence GC content |  WARNING | GC distribution deviates from normal |
| Overrepresented sequences |  WARNING | Some sequences appear with high frequency |
| **Adapter Content** |  **FAIL** | **Significant adapter contamination detected** |

**Screenshots Available:**
![Summary of raw quality metrics](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/Genome%20Trimming/SummaryContentRaw.png) - Summary of all quality metrics

![Adapter content in raw data](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/Genome%20Trimming/AdapterContentRaw.png) - Adapter contamination levels

The **FAIL** status on adapter content confirms the need for Trimmomatic processing.

---

## Step 3 — Trimming with Trimmomatic

We next need to run Trimmomatic to trim off adapter contamination and low-quality bases.

**Trimmomatic Documentation:** http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf

### Trimmomatic Command

```bash
java -jar trimmomatic-0.38.jar PE \
  -threads 2 \
  -phred33 \
  -trimlog Bc395_errorlog.txt \
  Bc395_1.fq.gz \
  Bc395_2.fq.gz \
  Bc395_1_paired.fastq \
  Bc395_1_unpaired.fastq \
  Bc395_2_paired.fastq \
  Bc395_2_unpaired.fastq \
  ILLUMINACLIP:adaptors.fa:2:30:10 \
  SLIDINGWINDOW:20:20 \
  MINLEN:125
```

### Parameter Explanation

| Parameter | Value | Description |
|-----------|-------|-------------|
| `PE` | - | Paired-end mode |
| `-threads` | 2 | Number of CPU threads to use |
| `-phred33` | - | Phred quality score encoding |
| `-trimlog` | `Bc395_errorlog.txt` | Log file for trimming operations |
| `ILLUMINACLIP` | `adaptors.fa:2:30:10` | Remove Illumina adapters (seed mismatches:2, palindrome clip threshold:30, simple clip threshold:10) |
| `SLIDINGWINDOW` | `20:20` | Cut once average quality drops below 20 in 20bp window |
| `MINLEN` | 125 | Drop reads shorter than 125bp after trimming |

### Output Files

Trimmomatic generates four FASTQ files:

| File | Type | Description |
|------|------|-------------|
| `Bc395_1_paired.fastq` | Paired | Forward reads where both R1 and R2 survived trimming |
| `Bc395_2_paired.fastq` | Paired | Reverse reads where both R1 and R2 survived trimming |
| `Bc395_1_unpaired.fastq` | Unpaired | Forward reads where only R1 survived |
| `Bc395_2_unpaired.fastq` | Unpaired | Reverse reads where only R2 survived |

**The paired reads are used for downstream assembly as they maintain read-pair relationships.**

---

## Step 4 — Post-Trimming Quality Assessment

### Run FastQC on Trimmed Data

```bash
fastqc Bc395_1_paired.fastq Bc395_2_paired.fastq -o ~/sequences/trimmed
```

### FastQC Results: Trimmed Data

After Trimmomatic processing, the quality profile improves significantly:

| Issue Type | Severity | Description |
|------------|----------|-------------|
| Per tile sequence quality |  WARNING | Minor variation in quality across flow cell tiles |
| Per base sequence content |  WARNING | Slight bias in base composition (expected after trimming) |
| Per sequence GC content |  WARNING | GC distribution slightly skewed |
| Sequence Length Distribution |  WARNING | Variable lengths due to trimming (expected) |
| **Adapter Content** |  **PASS** | **Adapter contamination successfully removed** |

**Screenshots Available:**
![Summary of trimmed quality metrics](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/Genome%20Trimming/SummaryContentTrimmed.png) - Summary showing improvement

![Adapter content successfully removed after trimming](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/Genome%20Trimming/AdapterContentTrimmed.png) - Confirms adapter removal

**Key Improvement:** The critical adapter content error has been resolved! All remaining warnings are expected artifacts of the trimming process or minor quality variations that won't significantly impact assembly.

---

## Step 5 — Trimming Results Summary

### Final Output Statistics

| Metric | Value |
|--------|-------|
| **Total Cleaned Reads** | 5,437,306 reads (paired + single-end) |
| **Total Bases After Trimming** | 1,635,036,372 bp (R1 + R2 combined) |
| **Read Pairing Status** | Maintained for downstream assembly |
| **Quality Threshold** | Minimum Q20 (99% base call accuracy) |
| **Minimum Read Length** | 125 bp |

### Quality Improvements

 **Successfully removed Illumina adapter sequences**  
 **Trimmed low-quality bases (Q < 20)**  
 **Removed reads shorter than 125bp**  
 **Maintained paired-read relationships for assembly**  
 **Generated clean, high-quality data ready for genome assembly**

### Files Ready for Next Step

The cleaned paired FASTQ files are now ready for genome assembly:
- `Bc395_1_paired.fastq` (forward reads)
- `Bc395_2_paired.fastq` (reverse reads)
