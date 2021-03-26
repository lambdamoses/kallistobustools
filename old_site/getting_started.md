---
layout: page
title: "Getting Started"
---

{% include JB/setup %}

<p align="center">
  <a href="secret.html">
    <img src="assets/secret_tsne.jpg" width="70%">
  </a>
</p>

This page provides instructions for how to pre-process the [mouse retinal cells SRR8599150](https://www.ncbi.nlm.nih.gov/sra/?term=SRR8599150) dataset from [Koren et al., 2019](https://doi.org/10.1016/j.immuni.2019.02.007) using the __kallisto &#124; bustools workflow__. A video for the tutorial can be viewed [here](https://youtu.be/hWxnL86sak8).

__Note:__ command line arguments are preceeded by`$`. For example, if you see `$ cd my_folder` then type `cd my_folder`.

#### 0. Download and install the software
Obtain ```kallisto``` from the [__kallisto__ installation page](https://pachterlab.github.io/kallisto/download), and ```bustools``` from the [bustools installation page](https://bustools.github.io/download). A video tutorial for how to install the software can be viewed [here](https://youtu.be/thvtp7Ik6ts).

__Note:__ this dataset is v2 chemistry. If you would like to process v3 chemistry then you would use the [10xv3 whitelist](https://github.com/BUStools/getting_started/releases).

#### 1. Download the materials
Prepare a folder:
```
$ mkdir kallisto_bustools_getting_started/; cd kallisto_bustools_getting_started/
```
Download the following files:

- Mouse transcriptome `Mus_musculus.GRCm38.cdna.all.fa.gz`
- 10x Chromium v2 chemistry barcode whitelist `10xv2_whitelist.txt`
- Transcripts to Genes map
- Read 1 fastq file `SRR8599150_S1_L001_R1_001.fastq.gz`
- Read 2 fastq file `SRR8599150_S1_L001_R2_001.fastq.gz`

```
$ wget ftp://ftp.ensembl.org/pub/release-96/fasta/mus_musculus/cdna/Mus_musculus.GRCm38.cdna.all.fa.gz
$ wget https://github.com/bustools/getting_started/releases/download/getting_started/10xv2_whitelist.txt
$ wget https://github.com/bustools/getting_started/releases/download/getting_started/transcripts_to_genes.txt
$ wget https://github.com/bustools/getting_started/releases/download/getting_started/SRR8599150_S1_L001_R1_001.fastq.gz
$ wget https://github.com/bustools/getting_started/releases/download/getting_started/SRR8599150_S1_L001_R2_001.fastq.gz
```
#### 2. Build an index
Build the species index (alternatively download a pre-built index from the [kallisto transcriptome indices](https://github.com/pachterlab/kallisto-transcriptome-indices) page):
```
$ gunzip Mus_musculus.GRCm38.cdna.all.fa.gz
$ kallisto index -i Mus_musculus.GRCm38.cdna.all.idx -k 31 Mus_musculus.GRCm38.cdna.all.fa
```

#### 3. Run kallisto
Pseudoalign the reads:
```
$ kallisto bus -i Mus_musculus.GRCm38.cdna.all.idx -o bus_output/ -x 10xv2 -t 4 SRR8599150_S1_L001_R1_001.fastq.gz SRR8599150_S1_L001_R2_001.fastq.gz
```
#### 4. Run bustools
Correct, sort, and count the bus file. This creates the gene count matrix:
```
$ cd bus_output/
$ mkdir genecount/ tmp/
$ bustools correct -w ../10xv2_whitelist.txt -p output.bus | bustools sort -T tmp/ -t 4 -p - | bustools count -o genecount/genes -g ../transcripts_to_genes.txt -e matrix.ec -t transcripts.txt --genecounts -
```

#### 5. Load the count matrices into a notebook
See [this python notebook](https://github.com/BUStools/getting_started/blob/master/getting_started.ipynb) for how to load the count matrices into [ScanPy](https://scanpy.readthedocs.io/en/latest/index.html) for analysis.

**Note:** Details for each of the steps are described in the [explanation page](getting_started_explained.html).
