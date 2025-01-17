# rMATS-long

[![Latest Release](https://img.shields.io/github/release/Xinglab/rMATS-long.svg?label=Latest%20Release)](https://github.com/Xinglab/rMATS-long/releases/latest)
[![Total GitHub Downloads](https://img.shields.io/github/downloads/Xinglab/rMATS-long/total.svg?label=Total%20GitHub%20Downloads)](https://github.com/Xinglab/rMATS-long/releases)
[![Total Bioconda Installs](https://img.shields.io/conda/dn/bioconda/rmats-long.svg?label=Total%20Bioconda%20Installs)](https://anaconda.org/bioconda/rmats-long)

## About

rMATS-long is an integrated computational workflow for long-read RNA-seq data. Building on our ESPRESSO tool for robust transcript discovery and quantification using long-read RNA-seq data, rMATS-long enables differential isoform analysis between sample groups, as well as classification and visualization of isoform structure and abundance.

## Table of Contents

* [Dependencies](#dependencies)
* [Usage](#usage)
  + [Detect Differential Isoforms](#detect-differential-isoforms)
  + [Visualize Isoforms and Abundance](#visualize-isoforms-and-abundance)
  + [Classify Isoform Differences](#classify-isoform-differences)
  + [Example](#example)

## Dependencies

Dependencies can be installed to a conda environment by running [./install](./install). Then the scripts can be run when the conda environment is activated: `conda activate ./conda_env`

Another option is to install the rmats-long bioconda package:
```
conda install -c conda-forge -c bioconda rmats-long
```
The rMATS-long scripts are included in the bioconda package and will be available in that conda environment at `/path/to/conda_env/rMATS-long/`

* [Python](https://www.python.org/) (v3.10.9)
  + [NetworkX](https://networkx.org/) (v2.8.8)
  + [NumPy](https://numpy.org/) (v1.24.2)
  + [pandas](https://pandas.pydata.org/) (v1.5.3)
  + [matplotlib](https://matplotlib.org/) (v3.7.1)
* [R](https://www.r-project.org/) (v4.2.0)
  + [BiocParallel](https://bioconductor.org/packages/release/bioc/html/BiocParallel.html) (v1.32.5)
  + [DRIMSeq](https://bioconductor.org/packages/release/bioc/html/DRIMSeq.html) (v1.26.0)
  + [ggplot2](https://ggplot2.tidyverse.org/) (v3.4.1)
  + [cowplot](https://wilkelab.org/cowplot/index.html) (v1.1.1)

Those versions were used during testing. Other versions may also be compatible

## Usage

First run [ESPRESSO](https://github.com/Xinglab/espresso) to detect and quantify isoforms using long-read data. The ESPRESSO output can be used with [scripts/rmats_long.py](scripts/rmats_long.py) which runs the following steps:
* [Detect differential isoform usage](#detect-differential-isoforms): [scripts/detect_differential_isoforms.py](scripts/detect_differential_isoforms.py)
* [Visualize isoforms and abundance](#visualize-isoforms-and-abundance): [scripts/visualize_isoforms.py](scripts/visualize_isoforms.py)
* [Classify isoform differences](#classify-isoform-differences): [scripts/classify_isoform_differences.py](scripts/classify_isoform_differences.py)

The individual scripts used by [scripts/rmats_long.py](scripts/rmats_long.py) can also be run directly if desired

```
python rmats_long.py -h

usage: rmats_long.py [-h] --abundance ABUNDANCE --updated-gtf UPDATED_GTF
                     [--gencode-gtf GENCODE_GTF] --group-1 GROUP_1 --group-2
                     GROUP_2 [--group-1-name GROUP_1_NAME]
                     [--group-2-name GROUP_2_NAME] --out-dir OUT_DIR
                     [--num-threads NUM_THREADS]
                     [--process-top-n PROCESS_TOP_N]
                     [--plot-file-type {.pdf,.png}]
                     [--diff-transcripts DIFF_TRANSCRIPTS]
                     [--adj-pvalue ADJ_PVALUE]
                     [--delta-proportion DELTA_PROPORTION]
                     [--compare-all-within-gene]

Analyze ESPRESSO results and produce plots for significant isoforms

options:
  -h, --help            show this help message and exit
  --abundance ABUNDANCE
                        The path to the abundance.esp file from ESPRESSO
  --updated-gtf UPDATED_GTF
                        The path to the updated.gtf file from ESPRESSO
  --gencode-gtf GENCODE_GTF
                        The path to a gencode annotation.gtf file. Will be
                        used to identify the Ensembl canonical isoform and the
                        gene name
  --group-1 GROUP_1     The path to a file listing the sample names for group
                        1. The file should have a single line with the sample
                        names as a comma separated list. The sample names
                        should match with the ESPRESSO abundance column names.
  --group-2 GROUP_2     The path to a file listing the sample names for group
                        2
  --group-1-name GROUP_1_NAME
                        A name for group 1 (default group 1)
  --group-2-name GROUP_2_NAME
                        A name for group 2 (default group 2)
  --out-dir OUT_DIR     The path to use as the output directory
  --num-threads NUM_THREADS
                        The number of threads to use (default 1)
  --process-top-n PROCESS_TOP_N
                        Generate plots and classify isoform differences for
                        the top "n" significant genes. By default all
                        significant genes are processed
  --plot-file-type {.pdf,.png}
                        The file type for output plots (default .pdf))
  --diff-transcripts DIFF_TRANSCRIPTS
                        The path to the differential transcript results. If
                        given then skip the differential isoform calculation.
  --adj-pvalue ADJ_PVALUE
                        The cutoff for adjusted p-value (default 0.05)
  --delta-proportion DELTA_PROPORTION
                        The cutoff for delta isoform proportion (default 0.1)
  --compare-all-within-gene
                        Compare the most significant isoform to all other
                        isoforms in the gene. By default, the most significant
                        isoform is only compared to the most significant
                        isoform with a delta proportion in the opposite
                        direction.
```

### Detect Differential Isoforms

[scripts/detect_differential_isoforms.py](scripts/detect_differential_isoforms.py) detects differential isoform expression using DRIMSeq. The samples in the ESPRESSO abundance file need to be separated into two groups. The two groups are written to the `--group-1` and `--group-2` input files as comma separated lists. The isoform counts from the abundance file are then used in the DRIMSeq pipeline

The main output file written to `--out-dir` is `differential_transcripts.tsv` which has these columns:
* `gene_id`
* `feature_id`: isoform ID
* `lr`: likelihood ratio statistic from DRIMSeq
* `df`: degrees of freedom
* `pvalue`
* `adj_pvalue`: Benjamini & Hochberg adjusted p-values
* `{sample_name}_proportion`: proportion of this isoform among all isoforms in this gene (1 column per sample)
* `group_1_average_proportion`
* `group_2_average_proportion`
* `delta_isoform_proportion`: `group_1_average_proportion - group_2_average_proportion`

The proportion columns were appended to the original output from DRIMSeq. `differential_transcripts_filtered.tsv` contains only the rows meeting the significance cutoffs.

The `--out-dir` also contains these files output by DRIMSeq:
* `differential_genes.tsv`
* `gene_pvalues.png`
* `precision_by_gene_expression.png`
* `transcript_pvalues.png`
* `transcripts_per_gene.png`

A summary of the number of isoforms and genes passing the default filters will be printed to stdout. The counts using different filters can be printed using [scripts/count_significant_isoforms.py](scripts/count_significant_isoforms.py)

```
python detect_differential_isoforms.py -h

usage: detect_differential_isoforms.py [-h] --abundance ABUNDANCE --out-dir
                                       OUT_DIR --group-1 GROUP_1 --group-2
                                       GROUP_2 [--num-threads NUM_THREADS]
                                       [--adj-pvalue ADJ_PVALUE]
                                       [--delta-proportion DELTA_PROPORTION]

Detect differential isoform expression using DRIMSeq

options:
  -h, --help            show this help message and exit
  --abundance ABUNDANCE
                        The path to the abundance.esp file from ESPRESSO
  --out-dir OUT_DIR     The path to use as the output directory
  --group-1 GROUP_1     The path to a file listing the sample names for group
                        1. The file should have a single line with the sample
                        names as a comma separated list. The sample names
                        should match with the ESPRESSO abundance column names.
  --group-2 GROUP_2     The path to a file listing the sample names for group
                        2.
  --num-threads NUM_THREADS
                        The number of threads to use (default: 1)
  --adj-pvalue ADJ_PVALUE
                        The cutoff for adjusted p-value (default: 0.05)
  --delta-proportion DELTA_PROPORTION
                        The cutoff for delta isoform proportion (default: 0.1)
```

```
python count_significant_isoforms.py -h

usage: count_significant_isoforms.py [-h] --diff-transcripts DIFF_TRANSCRIPTS
                                     --out-tsv OUT_TSV
                                     [--adj-pvalue ADJ_PVALUE]
                                     [--delta-proportion DELTA_PROPORTION]

Count isoforms that meet the cutoff values

options:
  -h, --help            show this help message and exit
  --diff-transcripts DIFF_TRANSCRIPTS
                        The path to the differential transcript results
  --out-tsv OUT_TSV     The path to write transcripts that meet the cutoff
                        values
  --adj-pvalue ADJ_PVALUE
                        The cutoff for adjusted p-value (default: 0.05)
  --delta-proportion DELTA_PROPORTION
                        The cutoff for delta isoform proportion (default: 0.1)
```

### Visualize Isoforms and Abundance

[scripts/visualize_isoforms.py](scripts/visualize_isoforms.py) creates plots showing the isoform abundance and structure. The `--gene-id` can be selected from the [differential isoform test](#detect-differential-isoforms). The `--abundance` and `--updated-gtf` files are from the ESPRESSO output. By default, the most abundant isoforms for the gene will be plotted. Specific isoforms can be plotted with `--main-transcript-ids` or isoforms can be determined automatically if `--diff-transcripts` or `--gencode-gtf` are given. The most significant isoform and another significant isoform with opposite `delta_isoform_proportion` will be chosen from `--diff-transcripts` and the Ensembl canonical transcript will be selected based on a tag in the `--gencode-gtf`

```
python visualize_isoforms.py -h

usage: visualize_isoforms.py [-h] --gene-id GENE_ID [--gene-name GENE_NAME]
                             --abundance ABUNDANCE --updated-gtf UPDATED_GTF
                             [--gencode-gtf GENCODE_GTF]
                             [--diff-transcripts DIFF_TRANSCRIPTS] --out-dir
                             OUT_DIR [--plot-file-type {.pdf,.png}]
                             [--main-transcript-ids MAIN_TRANSCRIPT_IDS]
                             [--max-transcripts MAX_TRANSCRIPTS]
                             [--intron-scaling INTRON_SCALING]
                             [--group-1 GROUP_1] [--group-2 GROUP_2]
                             [--group-1-name GROUP_1_NAME]
                             [--group-2-name GROUP_2_NAME]

Visualize the structure and abundance of isoforms

options:
  -h, --help            show this help message and exit
  --gene-id GENE_ID     The gene_id to visualize
  --gene-name GENE_NAME
                        The name for the gene (used as plot title). If not
                        given then the gene_name from --gencode-gtf will be
                        used. If no other name is found then --gene-id is used
                        as a default
  --abundance ABUNDANCE
                        The path to the abundance.esp file from ESPRESSO
  --updated-gtf UPDATED_GTF
                        The path to the updated.gtf file from ESPRESSO
  --gencode-gtf GENCODE_GTF
                        The path to a gencode annotation.gtf file. Can be used
                        to identify the gene_name and Ensembl canonical
                        isoform
  --diff-transcripts DIFF_TRANSCRIPTS
                        The path to the differential transcript results. Can
                        be used to determine --main-transcript-ids
  --out-dir OUT_DIR     The path to use as the output directory
  --plot-file-type {.pdf,.png}
                        The file type for output plots (default .pdf))
  --main-transcript-ids MAIN_TRANSCRIPT_IDS
                        A comma separated list of transcript IDs to plot as
                        the main transcripts. If not given then the most
                        significant isoform from --diff-transcripts, a second
                        significant isoform with a delta proportion in the
                        opposite direction, and the Ensembl canonical isoform
                        from --gencode-gtf will be used if possible
  --max-transcripts MAX_TRANSCRIPTS
                        How many transcripts to plot individually. The
                        remaining transcripts in the gene will be grouped
                        together (max 5, default 5)
  --intron-scaling INTRON_SCALING
                        The factor to use to reduce intron length in the plot.
                        A value of 2 would reduce introns to 1/2 of the
                        original plot length (default 1)
  --group-1 GROUP_1     The path to a file listing the sample names for group
                        1. The file should have a single line with the sample
                        names as a comma separated list. The sample names
                        should match with the ESPRESSO abundance column names.
  --group-2 GROUP_2     The path to a file listing the sample names for group
                        2.
  --group-1-name GROUP_1_NAME
                        A name for group 1 (default group 1)
  --group-2-name GROUP_2_NAME
                        A name for group 2 (default group 2)
```

### Classify Isoform Differences

[scripts/classify_isoform_differences.py](scripts/classify_isoform_differences.py) compares the structures of isoforms within a gene by calling [scripts/FindAltTSEvents.py](scripts/FindAltTSEvents.py) with a "main" isoform and a second isoform (or all other isoforms in the gene by default)

[scripts/FindAltTSEvents.py](scripts/FindAltTSEvents.py) compares the structures of any two transcript isoforms. Local differences in transcript structure are classified into 7 basic alternative splicing categories:

![basic alternative splicing patterns](docs/basic_alt_splicing_patterns.jpg)

* Exon skipping (SE)
* Alternative 5'-splice site (A5SS)
* Alternative 3'-splice site (A3SS)
* Mutually exclusive exons (MXE)
* Intron retention (RI)
* Alternative first exon (AFE)
* Alternative last exon (ALE)

Any local differences in transcript structure that could not be classified as one of the 7 basic alternative splicing categories are classified as "complex" (COMPLEX). **Note:** It is possible to have combinations of alternative splicing events for any given pair of transcript isoforms.

The output is a tab-delimited file consisting of four fields:
* **Field 1**: ID for transcript isoform 1
* **Field 2**: ID for transcript isoform 2
* **Field 3**: Discovered alternative splicing events
* **Field 4**: Genomic coordinates for alternative splicing events

**Note:** Designation of transcript isoforms 1 and 2 is completely arbitrary. Moreover, if the two transcript isoforms contained in the input GTF file exhibit a combination of multiple alternative splicing events, each event will be reported as its own line in the output file.

```
python classify_isoform_differences.py -h

usage: classify_isoform_differences.py [-h] --main-transcript-id
                                       MAIN_TRANSCRIPT_ID --updated-gtf
                                       UPDATED_GTF [--gencode-gtf GENCODE_GTF]
                                       --out-tsv OUT_TSV
                                       [--second-transcript-id SECOND_TRANSCRIPT_ID]

Compare the structures of isoforms within a gene

options:
  -h, --help            show this help message and exit
  --main-transcript-id MAIN_TRANSCRIPT_ID
                        The transcript_id of the main isoform in the .gtf file
  --updated-gtf UPDATED_GTF
                        The path to the updated.gtf file from ESPRESSO
  --gencode-gtf GENCODE_GTF
                        The path to a gencode annotation.gtf file. Can be used
                        to compare against isoforms not detected by ESPRESSO
  --out-tsv OUT_TSV     The path of the output file
  --second-transcript-id SECOND_TRANSCRIPT_ID
                        If given, only compare the main transcript to this
                        transcript
```

```
python FindAltTSEvents.py -h

usage: FindAltTSEvents.py [-h] -i /path/to/input/GTF -o /path/to/output/file

This is a script to enumerate all transcript structure differences between a
pair of transcript isoforms

options:
  -h, --help            show this help message and exit
  -i /path/to/input/GTF
                        path to GTF file describing structures of two
                        transcript isoforms
  -o /path/to/output/file
                        path to output file
```

### Example

Example data is provided in [example/data.tar.gz](example/data.tar.gz). Unpack that file with:
```
cd example/
tar -xvf ./data.tar.gz
```

The unpacked files are:
* `example/gencode.v43.annotation_filtered.gtf`
* `example/GRCh38.primary_assembly.genome_filtered.fa`
* `example/group_1.txt`
* `example/group_2.txt`
* `example/gs689_1_filtered.sam`
* `example/gs689_2_filtered.sam`
* `example/gs689_3_filtered.sam`
* `example/pc3e_1_filtered.sam`
* `example/pc3e_2_filtered.sam`
* `example/pc3e_3_filtered.sam`
* `example/samples_N2_R0_abundance.esp`
* `example/samples_N2_R0_updated.gtf`

The example data is based on cell line data from [https://doi.org/10.1126/sciadv.abq5072](https://doi.org/10.1126/sciadv.abq5072). The 1D cDNA sequencing for GS689 and PC3E was processed to get .sam files. The reference data (gencode .gtf and GRCh38 .fa) and the .sam files were filtered to a few different regions to get a small dataset

The first step is to run [ESPRESSO](https://github.com/Xinglab/espresso) using the provided reference data and alignment files. The result files from ESPRESSO are included in the example data

Next run [scripts/rmats_long.py](scripts/rmats_long.py) which will perform all of the analysis steps and write output files to `./example_out/`. It requires the sample names from the abundance file to be split into groups as done in `group_1.txt`:
```
pc3e_1,pc3e_2,pc3e_3
```

and `group_2.txt`:
```
gs689_1,gs689_2,gs689_3
```

Here is the main command:
```
python ./scripts/rmats_long.py --abundance ./example/samples_N2_R0_abundance.esp --updated-gtf ./example/samples_N2_R0_updated.gtf --gencode-gtf ./example/gencode.v43.annotation_filtered.gtf --group-1 ./example/group_1.txt --group-2 ./example/group_2.txt --group-1-name PC3E --group-2-name GS689 --out-dir ./example_out --plot-file-type .png
```

[scripts/rmats_long.py](scripts/rmats_long.py) will run other commands. For this example it first runs:
```
python ./scripts/detect_differential_isoforms.py --abundance ./example/samples_N2_R0_abundance.esp --out-dir ./example_out --group-1 ./example/group_1.txt --group-2 ./example/group_2.txt --adj-pvalue 0.05 --delta-proportion 0.1 --num-threads 1
```

That should print: `found 8 isoforms from 4 genes with adj_pvalue <= 0.05 and abs(delta_isoform_proportion) >= 0.1`. One significant row from `example_out/differential_transcripts_filtered.tsv` is:
```
gene_id	feature_id	lr	df	pvalue	adj_pvalue	pc3e_1_proportion	pc3e_2_proportion	pc3e_3_proportion	gs689_1_proportion	gs689_2_proportion	gs689_3_proportion	group_1_average_proportion	group_2_average_proportion	delta_isoform_proportion
ENSG00000204580.14	ENST00000418800.6	19.0795950208005	1	1.25378257460589e-05	3.60462490199193e-05	0.2476	0.3807	0.2559	0.0	0.0	0.0203	0.2947	0.0068	0.288
```

Next [scripts/rmats_long.py](scripts/rmats_long.py) will run a similar command to what is below (but using some temporary files):
```
python ./scripts/visualize_isoforms.py --gene-id ENSG00000204580.14 --abundance ./example/samples_N2_R0_abundance.esp --updated-gtf ./example/samples_N2_R0_updated.gtf --diff-transcripts ./example_out/differential_transcripts.tsv --out-dir ./example_out --group-1 ./example/group_1.txt --group-2 ./example/group_2.txt --group-1-name PC3E --group-2-name GS689 --plot-file-type .png --gencode-gtf ./example/gencode.v43.annotation_filtered.gtf
```

The `--gene-id` is the significant gene which can be found in `./example_out/differential_transcripts_filtered.tsv`. The command will produce `example_out/ENSG00000204580.14_abundance.png`:
![DDR1 abundance](example_out/ENSG00000204580.14_abundance.png)

And `example_out/ENSG00000204580.14_structure.png`
![DDR1 structure](example_out/ENSG00000204580.14_structure.png)

The plots show that ENST00000418800.6 is abundant in PC3E samples but not in GS689 samples. ENST00000376568.8 is the most abundant isoform in GS689 samples.

[scripts/rmats_long.py](scripts/rmats_long.py) will determine the differences among transcripts within that gene in terms of splicing events with:
```
python ./scripts/classify_isoform_differences.py --updated-gtf ./example/samples_N2_R0_updated.gtf --out-tsv ./example_out/ENSG00000204580.14_isoform_differences_ENST00000418800.6_to_ENST00000376568.8.tsv --main-transcript-id ENST00000418800.6 --second-transcript-id ENST00000376568.8 --gencode-gtf ./example/gencode.v43.annotation_filtered.gtf
```

ENST00000418800.6 is chosen as the main transcript because it is the most significant isoform for this gene from `./example_out/differential_transcripts.tsv`. ENST00000376568.8 is chosen as the second transcript because it is the most significant isoform with a delta proportion in the opposite direction of the main transcript for this gene

`./example_out/ENSG00000204580.14_isoform_differences_ENST00000418800.6_to_ENST00000376568.8.tsv` shows that those two isoforms differ by an alternative first exon event and an exon skipping event. This can also be seen in the isoform structure plot:
```
transcript1	transcript2	event	coordinates
ENST00000418800.6	ENST00000376568.8	AFE	chr6:30884519:30884710:+;chr6:30882613:30882983:+
ENST00000418800.6	ENST00000376568.8	SE	chr6:30895404:30895514:+
```

[scripts/rmats_long.py](scripts/rmats_long.py) also runs similar commands for the 3 other significant genes that were detected. A summary is written to `./example_out/summary.txt`:
```
## significant differential transcript usage
total significant isoforms: 8
total genes with significant isoforms: 4
adjusted pvalue threshold: 0.05
delta isoform proportion threshold: 0.1
## alternative splicing classifications between isoform pairs
total classified isoform pairs: 4
exon skipping: 1
alternative 5'-splice site: 0
alternative 3'-splice site: 0
mutually exclusive exons: 0
intron retention: 0
alternative first exon: 0
alternative last exon: 0
complex: 1
combinatorial: 2
```
