+++
date = '2024-12-03T14:50:20+01:00'
draft = false
title = 'Genomic File Formats'
tags = ['genomics', 'bioinformatics']
+++

In bioinformatics, various file formats are used to represent genomic data. In my analysis, I use BED files, GFF/GTF files, NarrowPeak files, BedGraph files, and BigWig files. I decided to create a quick reference that briefly explains each file type and includes an example table showing the columns of these files.

## BED Files (Browser Extensible Data)

BED files are tab-delimited text files that define genomic regions. They typically contain three mandatory columns and can have additional optional columns.

| Column | Description |
|--------|-------------|
| 1      | Chromosome (e.g., chr1) or Scaffold (e.g., scaffold10671) |
| 2      | Start position (0-based) |
| 3      | End position (1-based) |
| 4      | Name (optional) |
| 5      | Score (optional, 0-1000) |
| 6      | Strand (optional, "+", "-", or ".") |

### Example:
```bash
chr1    1000    5000    peak1   960     +
chr2    2000    6000    peak2   850     -
```

## GFF/GTF Files (General Feature Format / Gene Transfer Format)

GFF and GTF files describe genomic features with attributes and follow a nine-column format. GTF is a stricter version of GFF.

| Column | Description                     |
|--------|---------------------------------|
| 1      | Chromosome (e.g., chr1)         |
| 2      | Source (e.g., ENSEMBL)          |
| 3      | Feature type (e.g., gene, exon) |
| 4      | Start position (1-based)        |
| 5      | End position                    |
| 6      | Score (. if not applicable)     |
| 7      | Strand (+, -, or .)             |
| 8      | Frame (0, 1, 2, or .)           |
| 9      | Attributes (key-value pairs)    |

### Example:
```bash
chr1    ENSEMBL gene    1000    5000    .       +       .       gene_id "gene1"; gene_name "GeneA";
chr1    ENSEMBL exon    1000    1200    .       +       .       gene_id "gene1"; exon_id "exon1";
```

## NarrowPeak Files

NarrowPeak files are an extension of BED files, primarily used for narrow peak data in ChIP-seq. For example, the peak calling algorithm [MACS](https://github.com/macs3-project/MACS) produces **.narrowPeak** formatted files.

| Column | Description                      |
|--------|----------------------------------|
| 1      | Chromosome                       |
| 2      | Start position                   |
| 3      | End position                     |
| 4      | Name                             |
| 5      | Score                            |
| 6      | Strand                           |
| 7      | Signal value (enrichment score)  |
| 8      | p-value (-log10 p-value)         |
| 9      | q-value (-log10 q-value)         |
| 10     | Peak position within the region  |

### Example

```bash
chr1    1000    1500    peak1   960     .       5.2     3.5     2.3     200
chr1    2000    2500    peak2   850     .       4.8     3.2     1.9     150
```

## BedGraph Files

BedGraph files are used to display continuous-valued data over genomic regions. This format represents the coverage data as discrete genomic intervals. 

BigWig files are a binary version of BedGraph files, optimized for fast access and visualization in genome browsers. They cannot be viewed directly but can be generated from BedGraph files.

| Column | Description                    |
|--------|--------------------------------|
| 1      | Chromosome                     |
| 2      | Start position                 |
| 3      | End position                   |
| 4      | Value (e.g., coverage, intensity) |

### Example

```bash
chr1    1000    1500    3.5
chr1    1500    2000    2.1
```

## Converting File Formats

Can one convert one file to another? yes of course. you need some convertion tools or workflows to achieve such task (e.g., [BEDTools](https://bedtools.readthedocs.io/en/latest/), [UCSC tools](https://hgdownload.soe.ucsc.edu/admin/exe/), [deepTools](https://deeptools.readthedocs.io/en/latest/) ..).

### BED to GTF Conversion

Converting a BED file to a GTF file involves transforming BED regions into GTF features.

- **Example using awk**:

```bash
awk 'BEGIN{OFS="\t"} {print $1, "custom", "exon", $2+1, $3, ".", ".", ".", "gene_id \""$4"\";"}' input.bed > output.gtf
```
  
- ```BEGIN{OFS="\t"}``` sets the Output Field Separator (OFS) to a tab character.
- The ```print``` statement creates the 9 GTF columns which are first column from BED (```$1```), hard-coded source field (```"custom"```), hard-coded feature type (```"exon"```), second column from BED plus 1 (```$2+1```), third column from BED (```$3```), three dots for score, strand, and frame fields (```".", ".", "."```) and attributes field (```"gene_id \""$4"\";"```).

### BAM/BED to BedGraph Conversion

To convert a BED file to a BedGraph file, you need to include a value for each region. This can represent signal intensity or read depth or covarage.

**Example of BAM to BedGraph using BEDTools**

```bash
bedtools genomecov -ibam input.bam -bg > output.bedgraph
```

- `bedtools genomecov` is a command from the BEDTools suite used to compute the coverage of features in a BED file across a genome.
- ```-ibam ``` flag specifies that the input is in BAM format and ```input.bam``` is the input BAM file containing aligned sequencing reads. 
- ```-bg``` option tells the tool to output in BedGraph format. 

**Example of BAM to BedGraph using Samtools and awk**

```bash
# First, sort the bam file
samtools sort -o sorted_input.bam input.bam
# Second, calculate depth and convert to BedGraph
samtools depth -a sorted_input.bam | awk 'BEGIN {OFS="\t"} {print $1, $2-1, $2, $3}' > output.bedgraph
```

- First step is Sorting the BAM file. 
  - ```samtools sort``` is a command used to sort a BAM file. 
  - ``-o`` flag pecifies the output of sorted BAM file (```sorted_input.bam```). 
  - ```input.bam``` is the BAM file to be sorted. 

- Second step is converting to BedGraph.
  - ```samtools depth``` is a command for calculating covarage at each position and ```-a``` flag reports coverage at all positions, including those with zero coverage. 
  - As in BED to GTF conversion, ```BEGIN {OFS="\t"}``` sets output field separator to tab, ```{print $1, $2-1, $2, $3}```prints namely chromosome, start position (converted to 0-based), end position and coverage value.

**Example of BED to BedGraph using BEDTools**

```bash
bedtools genomecov -i input.bed -bg -g genome.txt > output.bedgraph
```

- This command follows the same principle as the BAM to BedGraph conversion using BEDTools, with one key difference: when using a BED file as input, you must provide chromosome sizes. 
- The flag ```-i``` specifies the input BED file (input.bed) containing the genomic regions for which coverage is to be calculated. 
- Last option ```-g``` specifies a genome file (```genome.txt```) that contains the sizes of the chromosomes.

### BedGraph to BigWig/BigWig to BedGraph Conversion

BigWig is a compressed, indexed format designed for efficient visualization in genome browsers. You can convert between BigWig and BedGraph formats using UCSC Genome Browser tools such as `bedGraphToBigWig` and `bigWigToBedGraph`.

**Example using UCSC tool**:

```bash
# Convert BedGraph to BigWig
bedGraphToBigWig input.bedgraph genome.txt output.bw

# Convert BigWig to BedGraph
bigWigToBedGraph input.bw output.bedgraph
```

- `bedGraphToBigWig` and `bigWigToBedGraph` are command-line tools from the UCSC Genome Browser utilities. 
- ```input.bedgraph``` file contains continuous-valued data. 
- As with previous conversions, the genome file (`genome.txt`) containing chromosome sizes is mandatory for the BedGraph to BigWig conversion
- ```output.bw``` is the resulting BigWig file, which stores the same data in an indexed binary format.

> Since NarrowPeak files share the same column structure as BED files, you can use similar tools to convert them into BedGraph, BigWig, or GTF formats.

### Convert NarrowPeak to BedGraph

```bash
awk 'BEGIN{OFS="\t"} {print $1, $2, $3, $7}' input.narrowPeak > output.bedgraph
```
- This `awk` command extracts the chromosome, start position, end position, and signal value (`$7`) to create a BedGraph file.

### Convert NarrowPeak to GTF
```bash
awk 'BEGIN{OFS="\t"} {print $1, "custom", "peak", $2+1, $3, ".", $6, ".", "peak_id \""$4"\"; signal_value \""$7"\";"}' input.narrowPeak > output.gtf
```
- This `awk` command formats the NarrowPeak data into GTF format, adjusting the start position to 1-based and including attributes like `peak_id` and `signal_value`.

> If needed, sort BedGraph files using the Unix sort command:

```bash
sort -k1,1 -k2,2n input.bedgraph > sorted.bedgraph
```