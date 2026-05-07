# myGenome

Repo to hold Applied Biometrics materials in relation to Pyricularia oryzae sequencing project.

| **SUBID**    | **BioProject** | **BioSample** | **Accession**    | **Organism**           | **Sample Name** |
|-------------|----------------|---------------|------------------|------------------------|-----------------|
| SUB16053105 | PRJNA926786    | SAMN55302557  | JBWXUS000000000  | Pyricularia oryzae     | Bc395           |

**Software:** TRIMMOMATIC, SNAP, AUGUSTUS, MAKER, IGV, BLAST, VELVET, SPADES, BANDAGE, HISAT2

## Documentation Structure

This repository contains comprehensive documentation of the Pyricularia oryzae genome sequencing and analysis pipeline, organized by implementation step:

### 01 - Genome Trimming

**File:** `GenomeTrimming.md`

Quality assessment and trimming of raw sequencing reads using FastQC and Trimmomatic software.

**Software:** Trimmomatic, FastQC

### 02 - Genome Assembly

**File:** `GenomeAssembly.md`

De novo assembly of trimmed reads using Velvet and SPAdes assemblers with optimization for assembly quality metrics. Visualized with bandage.

**Software:** Velvet, SPAdes, Bandage

### 03 - Contig Identification
**File:** `ContigIdentification.md`

Identification of mitochondrial contigs for NCBI submission using BLAST against reference mitochondrial sequences.

**Software:** BLAST

### 04 - Genome Prediction
**File:** `GenomePrediction.md`

Gene prediction and annotation using three complementary algorithms: SNAP, AUGUSTUS, and MAKER with IGV visualization.

**Software:** SNAP, AUGUSTUS, MAKER, IGV

### 05 - RNA Sequencing

**File:** `RNASequencing.md`

RNA strand comparison of foreign genome using HISAT2 for the genome index for comparison and IGV for visualization.

**Software:** HISAT2, IGV

## Additional Notes

> All code ran is done with my genome ID (Bc395, nicknamed "Bernard")
>
> **Execution environments:**
> - Local VM (Ubuntu)
> - University of Kentucky Supercomputer (MCC) - for computationally intensive steps (marked with `sbatch` commands)
>
> Some software utilities and shell scripts (.sh files) are not included in this documentation but are referenced with descriptions of their functionality.
>
> Some folder or file names could be renamed in between function runnings (such as a file being ran initially as `xyz.txt` and later renamed and run again with `x.txt`)