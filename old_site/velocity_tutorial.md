---
layout: page
title: "RNA velocity tutorial"
---

{% include JB/setup %}

<p align="center">
  <a href="">
    <img src="assets/website_velocity.jpg" width="60%">
  </a>
</p>

This tutorial provides instructions for how to pre-process a single-cell RNA-seq dataset with __kallisto &#124; bustools__ to perform an RNA velocity analysis. The tutorial explains the steps using as an example a single-cell RNA-seq experiment of human week 10 fetal forebrain from the [La Manno et al. 2018 paper](https://doi.org/10.1038/s41586-018-0414-6) (accessions SRR6470906 & SRR6470907).

__Note:__ for the instructions, command line arguments are preceeded by`$`. For example, if you see `$ cd my_folder` then type `cd my_folder`. 

#### 0. Download and install software
Obtain ```kallisto``` from the [__kallisto__ installation page](https://pachterlab.github.io/kallisto/download), and ```bustools``` from the [__bustools__ installation page](https://github.com/BUStools/bustools). Download and install ```bamtofastq``` from [here](https://support.10xgenomics.com/docs/bamtofastq) to generate the original FASTQ files from the BAM files provided by the authors. For a brief tutorial on how to install ```bamtofastq``` please see [this page](install_bamtofastq.html).

#### 1. Download materials
Prepare a folder:
```
$ mkdir kallisto_bustools_getting_started/; cd kallisto_bustools_getting_started/
```
Download the following files:

- Human cDNA Transcripts `cDNA.correct_header.fa.gz`
- Human introns Transcripts `introns.correct_header.fa.gz`
- cDNA Transcripts to Capture `cDNA_transcripts.to_capture.txt.gz`
- Introns Transcripts to Capture `introns_transcripts.to_capture.txt.gz`
- cDNA/introns Transcripts to Genes map `cDNA_introns.t2g.txt.gz`
- 10x Chromium v2 chemistry barcode whitelist `10xv2_whitelist.txt`
- [SRR6470906 & SRR6470907 BAM Files](https://www.ebi.ac.uk/ena/data/view/PRJNA429950) 

```
$ wget https://github.com/BUStools/getting_started/releases/download/velocity_tutorial/cDNA.correct_header.fa.gz
$ wget https://github.com/BUStools/getting_started/releases/download/velocity_tutorial/introns.correct_header.fa.gz
$ wget https://github.com/BUStools/getting_started/releases/download/velocity_tutorial/cDNA_transcripts.to_capture.txt.gz
$ wget https://github.com/BUStools/getting_started/releases/download/velocity_tutorial/introns_transcripts.to_capture.txt.gz
$ wget https://github.com/BUStools/getting_started/releases/download/velocity_tutorial/cDNA_introns.t2g.txt.gz
$ wget https://github.com/BUStools/getting_started/releases/download/velocity_tutorial/10xv2_whitelist.txt
$ wget ftp://ftp.sra.ebi.ac.uk/vol1/SRA646/SRA646572/bam/10X_17_029.bam
$ wget ftp://ftp.sra.ebi.ac.uk/vol1/SRA646/SRA646572/bam/10X_17_028.bam
```
Uncompress all the files:
```
$ gunzip *.gz
```
#### 2. Make the FASTQ files
Since La Manno et al. did not release the FASTQ files, we have to generate them from the BAM files using 10x Genomics `bamtofastq`.
```
$ bamtofastq --reads-per-fastq=500000000 10X_17_029.bam ./06
$ bamtofastq --reads-per-fastq=500000000 10X_17_028.bam ./07
```
Your folder structure will look like the following. Note the `MissingLibrary` does not affect this analysis.
```
├── 06
│   ├── 10X_17_029_MissingLibrary_1_HL73JBCXY
│   │   ├── bamtofastq_S1_L002_I1_001.fastq.gz
│   │   ├── bamtofastq_S1_L002_R1_001.fastq.gz
│   │   └── bamtofastq_S1_L002_R2_001.fastq.gz
│   └── 10X_17_029_MissingLibrary_1_HLFGJBCXY
│       ├── bamtofastq_S1_L002_I1_001.fastq.gz
│       ├── bamtofastq_S1_L002_R1_001.fastq.gz
│       └── bamtofastq_S1_L002_R2_001.fastq.gz
├── 07
│   ├── 10X_17_028_MissingLibrary_1_HL73JBCXY
│   │   ├── bamtofastq_S1_L001_I1_001.fastq.gz
│   │   ├── bamtofastq_S1_L001_R1_001.fastq.gz
│   │   └── bamtofastq_S1_L001_R2_001.fastq.gz
│   └── 10X_17_028_MissingLibrary_1_HLFGJBCXY
│       ├── bamtofastq_S1_L001_I1_001.fastq.gz
│       ├── bamtofastq_S1_L001_R1_001.fastq.gz
│       └── bamtofastq_S1_L001_R2_001.fastq.gz
├── 10X_17_028.bam
├── 10X_17_029.bam
├── 10xv2_whitelist.txt
├── cDNA.correct_header.fa
├── cDNA_introns.t2g.txt
├── cDNA_transcripts.to_capture.txt
├── introns.correct_header.fa
└── introns_transcripts.to_capture.txt
```

#### 3. Build Index
Build the species velocity index:
```
$ cat cDNA.correct_header.fa introns.correct_header.fa > cDNA_introns.fa
$ kallisto index -i cDNA_introns.idx -k 31 cDNA_introns.fa
```
For instructions on how to create cDNA and intron references for index construction see the [Building a cDNA and intron index tutorial](velocity_index_tutorial.html). 

#### 4. Run kallisto
Pseudoalign the reads for 06:
```
$ kallisto bus -i cDNA_introns.idx -o bus_output_06/ -x 10xv2 -t 4 \
06/10X_17_029_MissingLibrary_1_HL73JBCXY/bamtofastq_S1_L002_R1_001.fastq.gz \
06/10X_17_029_MissingLibrary_1_HL73JBCXY/bamtofastq_S1_L002_R2_001.fastq.gz \
06/10X_17_029_MissingLibrary_1_HLFGJBCXY/bamtofastq_S1_L002_R1_001.fastq.gz \
06/10X_17_029_MissingLibrary_1_HLFGJBCXY/bamtofastq_S1_L002_R2_001.fastq.gz
```
And for 07:
```
$ kallisto bus -i cDNA_introns.idx -o bus_output_07/ -x 10xv2 -t 4 \
07/10X_17_028_MissingLibrary_1_HL73JBCXY/bamtofastq_S1_L001_R1_001.fastq.gz \
07/10X_17_028_MissingLibrary_1_HL73JBCXY/bamtofastq_S1_L001_R2_001.fastq.gz \
07/10X_17_028_MissingLibrary_1_HLFGJBCXY/bamtofastq_S1_L001_R1_001.fastq.gz \
07/10X_17_028_MissingLibrary_1_HLFGJBCXY/bamtofastq_S1_L001_R2_001.fastq.gz
```

#### 5. Run bustools
Correct, sort, capture, and count the spliced and unspliced matrices for 06:
```
$ cd bus_output_06/
$ mkdir cDNA_capture/ introns_capture/ spliced/ unspliced/ tmp/
$ bustools correct -w ../10xv2_whitelist.txt -p output.bus | bustools sort -o output.correct.sort.bus -t 4 -
$ bustools capture -o cDNA_capture/ -c ../cDNA_transcripts.to_capture.txt -e matrix.ec -t transcripts.txt output.correct.sort.bus
$ bustools capture -o introns_capture/ -c ../introns_transcripts.to_capture.txt -e matrix.ec -t transcripts.txt output.correct.sort.bus
$ bustools count -o unspliced/u -g ../cDNA_introns.t2g.txt -e cDNA_capture/split.ec -t transcripts.txt --genecounts cDNA_capture/split.bus
$ bustools count -o spliced/s -g ../cDNA_introns.t2g.txt -e introns_capture/split.ec -t transcripts.txt --genecounts introns_capture/split.bus
```
And for 07:
```
cd ../bus_output_07
$ bustools correct -w ../10xv2_whitelist.txt -p output.bus | bustools sort -o output.correct.sort.bus -t 4 -
$ bustools capture -o cDNA_capture/ -c ../cDNA_transcripts.to_capture.txt -e matrix.ec -t transcripts.txt output.correct.sort.bus
$ bustools capture -o introns_capture/ -c ../introns_transcripts.to_capture.txt -e matrix.ec -t transcripts.txt output.correct.sort.bus
$ bustools count -o unspliced/u -g ../cDNA_introns.t2g.txt -e cDNA_capture/split.ec -t transcripts.txt --genecounts cDNA_capture/split.bus
$ bustools count -o spliced/s -g ../cDNA_introns.t2g.txt -e introns_capture/split.ec -t transcripts.txt --genecounts introns_capture/split.bus
```

#### 6. Load count matrices into notebook
See [this notebook](https://github.com/BUStools/getting_started/blob/master/velocity_tutorial.ipynb) for how to process the spliced and unspliced count matrices to generate a velocity plot. 
