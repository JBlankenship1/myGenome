# myGenome

Repo to hold Applied Biometrics materials in relation to Pyricularia oryzae sequencing project.

| **SUBID**    | **BioProject** | **BioSample** | **Accession**    | **Organism**           | **Sample Name** |
|-------------|----------------|---------------|------------------|------------------------|-----------------|
| SUB16053105 | PRJNA926786    | SAMN55302557  | JBWXUS000000000  | Pyricularia oryzae     | Bc395           |

## Documentation Structure

This repository contains comprehensive documentation of the Pyricularia oryzae genome sequencing and analysis pipeline, organized by analysis step:

### 01 - Sequence QC & Trimming
**File:** `01-sequence-qc-trimming/sequence-qc-trimming.txt`

Quality assessment and trimming of raw sequencing reads using FastQC and Trimmomatic software.

**Key outputs:**
- 5,437,306 cleaned & single-end reads
- 1,635,036,372 total bases (R1 + R2)

### 02 - Genome Assembly
**File:** `02-genome-assembly/genome-assembly.txt`

De novo assembly of trimmed reads using Velvet and SPAdes assemblers with optimization for assembly quality metrics.

**Software:** Velvet, SPAdes, Bandage

**Final assembly statistics:**
- Genome size: 43,890,846 Bp
- Number of contigs: 5,527
- N50: 73,820
- Coverage: ~37x
- BUSCO score: 98.30%

### 03 - Contig Identification
**File:** `03-contig-identification/contig-identification.txt`

Identification of mitochondrial contigs for NCBI submission using BLAST against reference mitochondrial sequences.

**Software:** BLAST

**Results:**
- 1,661 matching contigs identified
- 1,330 non-matching contigs
- Mitochondrial contigs exported to CSV for NCBI submission

### 04 - Gene Prediction
**File:** `04-gene-prediction/gene-prediction.txt`

Gene prediction and annotation using three complementary algorithms: SNAP, AUGUSTUS, and MAKER with IGV visualization.

**Software:** SNAP, AUGUSTUS, MAKER, IGV

**Gene prediction results:**
- SNAP: 12,740 predicted genes
- AUGUSTUS: 17,691 predicted genes
- MAKER: 13,196 predicted genes (refined with ab initio predictions)

## FastQC Reports

### Raw Data Warnings & Errors
- [WARNING] Per base sequence content
- [WARNING] Per sequence GC content
- [WARNING] Overrepresented sequences
- [ERROR] Adapter Content

**Images:** Genome Trimming/`SummaryContentRaw.png`, Genome Trimming/`AdapterContentRaw.png`

### Trimmed Data Warnings
- [WARNING] Per tile sequence quality
- [WARNING] Per base sequence content
- [WARNING] Per sequence GC content
- [WARNING] Sequence Length Distribution

**Images:** Genome Trimming/`SummaryContentTrimmed.png`, Genome Trimming/`AdapterContentTrimmed.png`

## Additional Notes

All code executed using genome ID: **Bc395** (nicknamed "Bernard")

Execution environments:
- Local VM (Ubuntu)
- University of Kentucky Supercomputer (MCC) - for computationally intensive steps (marked with `sbatch` commands)

Some software utilities and shell scripts (.sh files) are not included in this documentation but are referenced with descriptions of their functionality.