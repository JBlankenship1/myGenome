# Contig Identification Steps

**Author:** Jacob Blankenship  
**Software:** BLAST  

> **Note:** All code ran is done with my genome ID (**Bc395**)[cite: 1]. It is sometimes completed with a VM (which runs Ubuntu) and other times is run on the University of Kentucky supercomputer **MCC** (any code that starts with `sbatch` is done on MCC as the execution is computationally intense)[cite: 1]. Some software utilized and some shell scripts that are ran (`.sh` files) are not provided in this documentation, but general descriptions of what they do will be provided[cite: 1]. Finally, some folder or file names could be renamed in between function runnings[cite: 1].

---

## Step 1 — Mitochondrial Contig Identification

NCBI submission requires the identification of mitochondrial contigs[cite: 1]. These are isolated by running BLAST on the assembly FASTA file and comparing it to the sequence of a similar organism, specifically `MoMitochondrion.fasta`[cite: 1].

### BLAST Execution
On the MCC, BLAST is executed using a Singularity container[cite: 1]. The command utilizes the `amd-conda1-centos8.sinf` environment image, a statistical filter of $10^{-50}$, and a maximum of 20,000 sequences[cite: 1].

```bash
singularity run --app blast2120 /share/singularity/images/ccs/conda/amd-conda1-centos8.sinf blastn \
-query MoMitochondrion.fasta -subject Bc395_final.fasta -evalue 1e-50 -max_target_seqs 20000 \
-outfmt '6 qseqid sseqid slen length qstart qend sstart send btop' -out MoMitochondrion.Bc395.BLAST
```

### Filtering Results
The BLAST file is simplified to isolate contigs where the mitochondrial genome represents **90% or more** of the total length[cite: 1]. This filtered data is written to a `.csv` file for NCBI submission[cite: 1].

```bash
awk '$4/$3 >= 0.9 {print $2 ",mitochondrion"}' MoMitochondrion.Bc395.BLAST > Bc395_mitochondrion.csv
```

---

## Step 2 — Comparative Genome Analysis (B71v2sh)

The **Bc395** assembly can be contrasted with the **B71v2sh** reference genome to determine alignment coverage[cite: 1].

### Cross-Genome BLAST
A BLAST file is generated to identify merged contigs between the two organisms[cite: 1]:

```bash
blastn -query B71.fasta -subject Bc395_final.fasta -evalue 1e-100 -outfmt 7 > B71.Bc395.BLAST
```

### Contig Quantification
The following commands quantify the unique and shared contigs between genomes[cite: 1]:

*   **Total Assembly Contigs**: **2,991**[cite: 1]
*   **Matching Contigs**: **1,661**[cite: 1]
*   **Difference (Unmatched)**: **1,330**[cite: 1]

```bash
# Identify matching contigs
awk '$2 ~ "contig" {sub(/~.*/, "", $2); print $2}' Blast_Bc395/B71.MyBc395.BLAST | sort -u > matchcontigs.txt

# Calculate total contigs in assembly
awk '/contig/ {gsub(/^>/, "", $1); sub(/~.*/, "", $1); print $1}' Bc395_Fasta_Files/Bc395_final.fasta | sort -u > totalcontigs.txt

# Find the difference and list unmatched contigs
echo $(( $(wc -l < totalcontigs.txt) - $(wc -l < matchcontigs.txt) ))
grep -Fxv -f matchcontigs.txt totalcontigs.txt
```

---

## Step 3 — GFF3 Conversion & Visualization

To facilitate visual analysis in the **Integrative Genomics Viewer (IGV)**, the BLAST output is converted into a **GFF3** format[cite: 1].

### Alignment Conversion
The following script maps BLAST columns to GFF3 attributes, including sequence ID, source, type (match), coordinates, score, and strand[cite: 1].

```bash
awk -v OFS='\t' 'BEGIN{print "##gff-version 3"; id=1} !/^#/{if($7<$8){s=$7;e=$8;d="+"}else{s=$8;e=$7;d="-"} \
print $1,"AWK","match",s,e,$12,d,".","ID=match"id; id++}' Blast_Bc395/B71.Bc395.BLAST > Bc395_B71_alignment.gff3
```

### IGV Visualization
The `B71.fasta` file is loaded as the reference genome in IGV, with the `Bc395_B71_alignment.gff3` file serving as the alignment track to visualize where the **Bc395** genome aligns[cite: 1].

> **Screenshot Reference**: `Contig Identification\Bc395_B71_GenomeAlignment.png`[cite: 1]