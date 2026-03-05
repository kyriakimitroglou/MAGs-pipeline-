# Metagenomics Pipeline for MAGs Recovery

A step-by-step workflow for analyzing metagenomic data, 
from raw reads to annotated metagenome-assembled genomes (MAGs).

## Pipeline Overview

1. Data Acquisition
2. Quality Control & Trimming
3. Metagenomic Assembly
4. Coverage Estimation
5. Binning & MAG Quality
6. Phylogenetic Visualization
7. Functional Annotation

## 1. Data Acquisition
Download public datasets using the SRA Toolkit.
Use accession number PRJNA448333 as practice data.
```
fastq-dump --split-files  <accession_number>
```

## 2. Quality Control & Trimming

Run FastQC to assess read quality:
```
mkdir fastqc_results
for file in *.fastq.gz; do
    fastqc ${file} -o fastqc_results
done
```

Trim low-quality reads with  Trimmomatic:
```
# Example command to run Trimmomatic
trimmomatic PE -threads 4 -phred33 \
    input_R1.fastq.gz input_R2.fastq.gz \
    trimmed_R1_paired.fastq.gz trimmed_R1_unpaired.fastq.gz \
    trimmed_R2_paired.fastq.gz trimmed_R2_unpaired.fastq.gz \
    ILLUMINACLIP:TruSeq3-PE.fa:2:30:10 LEADING:3 TRAILING:3  SLIDINGWINDOW:4:15 MINLEN:36
```

Quality Control after trimming
Run FastQC again after trimming to confirm improvement:
```
mkdir fastqc_results_after_trimming
for file in trimmed_*_paired.fastq.gz; do
    fastqc ${file} -o fastqc_results_after_trimming
done
```

## 3. Metagenomic Assembly
Assemble trimmed reads into contigs using metaSPAdes.
Use only the paired output files from Trimmomatic:
```
metaspades.py -1 trimmed_R1_paired.fastq.gz \
              -2 trimmed_R2_paired.fastq.gz \
              -o assembly_output
```

## 4. Coverage Estimation
Map reads back to contigs to calculate coverage:
```
# Index the contigs
bwa index assembly_output/contigs.fasta

# Map trimmed reads to contigs
bwa mem assembly_output contigs.fasta trimmed_R1_paired.fastq.gz trimmed_R2_paired.fastq.gz > mapped.sam

# Convert SAM to BAM and sort
samtools view -Sbu mapped.sam | samtools sort -o mapped_sorted.bam

# Index the sorted BAM file
samtools index mapped_sorted.bam
```

## 5. Binning & MAG Quality Assessment
Bin contigs into MAGs using MetaBAT2:
```
metabat2 -i assembly_output/contigs.fasta \
         -o bins/bin \
         -m 1500 --unbinned
```

Assess MAG completeness and quality with CheckM2:
```
checkm2 predict --threads 4 \
                --input bins/ \
                --output-directory checkm2_output
```

## 6. Phylogenetic Visualization
Visualize the phylogenetic tree using iTOL:
- Convert tree to Newick format with FigTree
- Upload to https://itol.embl.de

## 7. Functional Annotation
Annotate MAGs using Prokka:
```
prokka --outdir annotation_output \
       --prefix my_sample \
       assembly_output/contigs.fasta
```


## Useful Resources
- [EMBL Metagenomics Course](https://www.ebi.ac.uk/training/online/courses/metagenomics-bioinformatics/)
- [MGnify Assembly Tutorial](https://mgnify-ebi-2020.readthedocs.io/en/latest/assembly.html)
- [MicrobiomeHD Database](https://github.com/cduvallet/microbiomeHD)

