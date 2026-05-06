# Genome Assembly Implementation Steps

**Author:** Jacob Blankenship  
**Software:** Velvet, Spades, Bandage

> **Note:** All code ran is done with my genome ID (Bc395). It is sometimes completed with a VM (which runs Ubuntu) and other times is run on the University of Kentucky supercomputer MCC (any code that starts with `sbatch` is done on MCC as the execution is computationally intense). Some software utilized and some shell scripts that are ran (`.sh` files) are not provided in this documentation, but general descriptions of what they do will be provided. Finally, some folder or file names could be renamed in between function runnings (such as a file being ran initially as `xyz.txt` and later renamed and run again with `x.txt`).

---

## Step 1 — Velvet Assembly

Next we need to run Velvet + SPAdes on our genome and find out which parameters with the assembler software create the optimal assembled genome, using statistics such as genome size, number of contigs, and N50.

 [Velvet on GitHub](https://github.com/dzerbino/velvet/tree/master)

### Finding an Optimal K-mer Value

A good k-mer length can be found with Velvet Advisor, using the summary stats from our `.html` files as input. The k-mer length is the average length we believe our contigs should be — the higher the k-mer value, the more "complete" and "clean" the assembly is, and the fewer contigs remain.

 [Velvet Advisor](http://dna.med.monash.edu.au/~torsten/velvet_advisor/)

### Initial Optimiser Run

With an optimal k-mer value identified, we can submit a batch job to run Velvet Optimiser (called via shell script). This takes in k-mer values and step sizes — the lowest in this case being `77 - 40 = 37`, the highest being `77 + 40 = 107`, and a step size of `10`, which checks all increments of 10 between 37 and 107:

```bash
sbatch velvetoptimiser.sh MyGenomeID 37 107 10
```

**Results — K = 97:**

| Metric | Value |
|--------|-------|
| Assembled genome size | 44,092,031 bp |
| Number of contigs | 6,622 |
| N50 | 22,575 |

> **What is N50?** Take all contigs and order them by length. Then, moving from longest to shortest, find the contig at the 50% mark — meaning 50% of the genome is made up of contigs as large or larger than this one. That contig's length is the N50 value.

### Refined Optimiser Run

To refine our result, we rerun the optimiser over a shortened range around our best k-mer value (97), with a step size of 2:

```bash
sbatch velvetoptimiser.sh MyGenomeID 89 105 2
```

**Results — K = 95:**

| Metric | Value |
|--------|-------|
| Assembled genome size | 44,001,401 bp |
| Number of contigs | 5,958 |
| N50 | 24,881 |

---

## Step 2 — SPAdes Assembly

With Velvet complete, we can also run SPAdes (another assembler). This is also run via shell script:

```bash
sbatch spades.sh . Bc395
```

The shell script was run with two different input configurations — one using both paired and unpaired reads, and one using only paired reads.

**Results Comparison:**

| Configuration | Genome Size (bp) | Contigs | N50 |
|---------------|-----------------|---------|-----|
| Paired + Unpaired | 45,755,384 | 15,452 | 58,752 |
| Paired only | 43,890,846 | 5,527 | 73,820 |

### Choosing the Best Assembly

When evaluating assemblies, the ideal output has:
- **More base pairs** — more genome to work with
- **Fewer contigs** — less fragmentation, more of the genome has "found a home"
- **Higher N50** — more of the genome is represented by larger contigs

**The SPAdes paired-only assembly was selected**, as it had the best contig count and by far the best N50, with a similar base pair total.

### Fold Coverage

To find the fold coverage, we divide the total base pairs across both the forward and reverse reads by the typical genome size of the organism (*Pyricularia oryzae*). This yields approximately **~37x coverage**, meaning the reads cover the organism's genome 37 times over. This provides strong redundancy, helping to flag sequencing and tracking errors.

> It should also be noted that the SPAdes output includes a `scaffolds.gfa` graph file, which is the source of the genome visualizations. This file is visualized using **Bandage**.

---

## Step 3 — NCBI Submission Cleanup

To prepare data for submission to the **National Center for Biotechnology Information (NCBI)**, some final cleaning must be done.

### Rename FASTA Headers

Using a Perl script, we can iterate through the FASTA file and rename headers with incremental contig names. We use the `scaffolds.fasta` file from the SPAdes output directory:

```bash
perl SimpleFastaHeaders.pl scaffolds.fasta Bc395
```

### Post-Processing

Using the output `Bc395_newheader.fasta`, we run the genome post-processing shell script. This will identify adaptor contamination, remove contaminating sequences, and cull sequences less than 200 nt in length:

```bash
sbatch GenomePostProcess.sh Bc395_newheader.fasta
```

This produces the final submission-ready file: **`Bc395_final.fasta`**.

---

## Step 4 — BUSCO Scoring

As a final validation step, we run BUSCO on our final FASTA file. This compares our assembled genome against a fungal phylum reference (`odb10_ascomycota`) and returns a score representing the percentage of ortholog matches between the two organisms:

```bash
sbatch BuscoSingularity.sh Bc395_final.fasta
```

**Results:**

| Metric | Score |
|--------|-------|
| BUSCO score | 98.30% |
| Completed + fragmented score | 98.50% |

A high BUSCO score is expected for closely related organisms, so a result of **98.3%** confirms our assembly is strong and we can proceed.