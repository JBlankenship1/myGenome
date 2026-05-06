# Gene Trimming Implementation Steps

**Author:** Jacob Blankenship  
**Software:** Trimmomatic

> **Note:** All code ran is done with my genome ID (Bc395, nicknamed "Bernard"). It is sometimes completed with a VM (which runs Ubuntu) and other times is run on the University of Kentucky supercomputer MCC (any code that starts with `sbatch` is done on MCC as the execution is computationally intense). Some software utilized and some shell scripts that are ran (`.sh` files) are not provided in this documentation, but general descriptions of what they do will be provided. Finally, some folder or file names could be renamed in between function runnings (such as a file being ran initially as `xyz.txt` and later renamed and run again with `x.txt`).

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

## Step 2 — Sequencing

With all software and files ready, we can start sequencing.

### Run FastQC

Run FastQC on both forward and reverse reads:

```bash
fastqc Bc395_1.fq.gz Bc395_2.fq.gz -o ~/sequences
```

This will return `.html` files that can be downloaded and opened in a web browser to view. This can be done with `scp`:

```bash
scp jrbl245@jrbl245.cs.uky.edu:Bc395_1.fastqc.html .
scp jrbl245@jrbl245.cs.uky.edu:Bc395_2.fastqc.html .
```

The viewed HTML files include data on provided screenshots such as summary statistics and adaptor content.

### Run Trimmomatic

We next need to run Trimmomatic to trim off adaptor contamination.  
 [Trimmomatic Documentation](http://www.usadellab.org/cms/uploads/supplementary/Trimmomatic/TrimmomaticManual_V0.32.pdf)

```bash
java -jar trimmomatic-0.38.jar PE \
  -threads 2 \
  -phred33 \
  -trimlog Br80_errorlog.txt \
  Bc395_1.fq.gz \
  Bc395_2.fq.gz \
  Bc395_1_paired.fastq \
  Bc395_2_paired.fastq \
  ILLUMINACLIP:adaptors.fa:2:30:10 \
  SLIDINGWINDOW:20:20 \
  MINLEN:125
```

This will return four FASTQ files:

| File | Description |
|------|-------------|
| `Bc395_1.fq.gz` | Forward reads (original) |
| `Bc395_2.fq.gz` | Reverse reads (original) |
| `Bc395_1_paired.fastq` | Forward paired reads |
| `Bc395_2_paired.fastq` | Reverse paired reads |

The paired reads include both the forward and reverse reads.