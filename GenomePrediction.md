# Genome Prediction Implementation Steps

**Author:** Jacob Blankenship  
**Software:** SNAP, AUGUSTUS, MAKER, IGV
**Genome ID:** Bc395

---

## Step 1 — Training the SNAP HMM Model

Gene analysis begins by accessing reference genomes from organisms similar to *Pyricularia oryzae* (Bernard), specifically the **B71Ref2** datasets.

### Preparing the Training Data
The gene annotations and genome sequences are combined into a single file to create a comprehensive reference:

```bash
echo '##FASTA' | cat B71Ref2_a0.3.gff3 - B71Ref2.fasta > B71Ref2.gff3
```

### Exporting Sequences and Model Training
The `fathom` utility extracts genome, transcript, and protein sequences while maintaining 1,000 base pairs of context and standardizing sequence orientation. Subsequently, `forge` is employed to train the dataset.

```bash
# Export formatted dataset
fathom uni.ann uni.dna -export 1000 -plus

# Train the model
forge export.ann export.dna
```

### HMM Assembly
The final phase of training involves assembling the generated parameters into a singular `.hmm` file required for prediction:

```bash
hmm-assembler.pl Moryzae . > Moryzae.hmm
```

---

## Step 2 — Gene Prediction (SNAP & AUGUSTUS)

### SNAP Prediction
The `Moryzae.hmm` file is executed against the cleaned FASTA file to generate a `.zff` text document listing the coordinates and features inferred by the Markov model.

```bash
# Run SNAP prediction
snap-hmm Moryzae.hmm Bc395_final.fasta > Bc395-snap.zff

# Generate statistics
fathom Bc395-snap.zff Bc395.fasta -gene-stats

# Create GFF2 for visualization
snap-hmm Moryzae.hmm Bc395_final.fasta -gff > Bc395-snap.gff2
```

### AUGUSTUS Prediction
AUGUSTUS provides *ab initio* gene matches using pre-existing species profiles. For this implementation, the species is set to *Magnaporthe grisea* (synonymous with *Pyricularia oryzae*):

```bash
augustus --species=magnaporthe_grisea --gff3=on --singlestrand=true --progress=true Bc395_final.fasta > Bc395-augustus.gff3
```

---

## Step 3 — MAKER Pipeline

The MAKER pipeline integrates outputs from SNAP and AUGUSTUS with external protein evidence to create a consensus gene set.

### Configuration
Control (`.ctl`) files define the locations of external programs, alignment thresholds, and input data descriptions:

```bash
# Generate configuration files
singularity exec /share/singularity/images/ccs/MAKER/amd-maker-debian10.sinf maker -CTL
```

**Key Parameters adjusted in `maker_opts.ctl`:**
* `genome=Bc395_final.fasta`
* `snaphmm=Moryzae.hmm`
* `augustus_species=magnaporthe_grisea`
* `protein=ncbi-protein-Magnaporthe_organism.fasta`
* `keep_preds=1` (Retains *ab initio* predictions even without protein evidence)

### Execution and Merging
The batch job processes individual genes across various directories, which are then merged into a master GFF3 file for usability:

```bash
# Submit batch job
sbatch maker.sh Bc395_final.fasta

# Merge outputs into a single file
singularity exec /share/singularity/images/ccs/MAKER/amd-maker-debian10.sinf gff3_merge -d Bc395_final.maker.output/Bc395_final_master_datastore_index.log -o Bc395-maker.gff3
```

---

## Step 4 — Results Summary & Visualization

### Quantitative Summary
The unique gene counts for each prediction method were determined using specialized command-line queries:

| Software | Query Command | Genes Identified |
| :--- | :--- | :--- |
| **SNAP** | `awk '{print $9}' Bc395-snap.gff3 \| sort -u \| wc -l` | **12,740** |
| **AUGUSTUS** | `grep "^# start gene" Bc395-augustus.gff3 \| wc -l` | **17,691** |
| **MAKER** | `awk '$3 ~ "gene"' Bc395-maker.gff3 \| wc -l` | **13,196** |

### IGV Visualization & Spot Check
The **Integrative Genomics Viewer (IGV)** facilitates the interactive exploration of genomic datasets. A spot check conducted on **Contig 19** (Coordinates: 25,081–26,664 BP) validated the agreement between prediction models.

**Image Reference:** 
![SNAP & AUGUSTUs Agreed Gene](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/Genome%20Prediction/SNAP+AUGUSTUSAgreedGene.png) - SNAP & AUGUSTUs Agreed Gene

![SNAP & AUGUSTUS Disagreed Gene](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/Genome%20Prediction/SNAP+AUGUSTUSDisagreedGene.png) - SNAP & AUGUSTUS Disagreed Gene

![SNAP Only Gene](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/Genome%20Prediction/SNAPOnlyGene.png) - SNAP Only Gene

![AUGUSTUS Only Gene](https://raw.githubusercontent.com/JBlankenship1/myGenome/main/Genome%20Prediction/AUGUSTUSOnlyGene.png) - AUGUSTUS Only Gene

**Validation Evidence:**
The following query confirms that the consensus MAKER model (`ID=snap-Bc395_contig19-processed-gene-0.51`) is supported by underlying SNAP and AUGUSTUS matches:

```bash
grep "25081" MAKER_Bc395/Bc395-maker.gff3 | grep "contig19" | grep -E "maker|snap|augustus"
```