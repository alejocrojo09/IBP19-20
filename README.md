# Integrated Bioinformatics Project 19-20

"SASpector: a tool for analysis of missing regions in (bacterial) draft genomes"
Alejandro Correa Rojo, 
Deniz Sinar (https://github.com/dsinar) and 
Emma Verkinderen (https://github.com/emmaver)

Advisor: Cédric Lood

Master of Bioinformatics - KU Leuven, Belgium

## Tool: SASpector (Short-read Assembly inSpector)
[![License: MIT](https://img.shields.io/apm/l/vim-mode)](https://mit-license.org/)
[![PyPI version](https://img.shields.io/pypi/v/SASpector)](https://pypi.org/project/SASpector/)
<p align="center">
  <img src="https://github.com/alejocrojo09/IBP19-20/blob/master/SASPector-final.png?raw=true" width="250" height="250"/>
</p>

A bioinformatics tool to extract and analyze missing regions of short-read assemblies by mapping the contigs to a reference genome.

## Introduction

SASpector is a tool that compares a short-reads assembly with a reference bacterial genome (for example obtained via hybrid assembly) by extracting missing (unmapped) regions from the reference and analyzing them to see functional and compositional pattern. The aim of the analysis is to explain why these regions are missed by the short-read assembly and if important parts of the genome are missed when a resolved genome is lacking.

The tool takes as global inputs the reference genome and a short-read assembly as contigs/draft genome, both in FASTA format. This repository contains a command-line tool `SASpector` to obtain missing regions and several script for evaluation
and analysis for those missing regions:

- `SASpectorcheck`: checks if the third-party tools used by SASpector are installed in the system.
- `mapper.py`: mapping of the short-reads assembly against the reference assembly using progressiveMauve (Mauve aligner).
- `summary.py`: extraction of the mapped, unmapped (missing) and conflict regions to FASTA files. Also, creates summary statistic for the missing regions and reference which are written to separate tab-delimited files.
- `gene_predict.py`: prediction of genes in the missing regions using Prokka. Optionally, run BLASTX to the predicted genes with an user-defined protein database in FASTA file. In this repository you can find protein database retrieved from UniprotKB for bacterial species: Escherichia coli, Pseudomonas aeruginosa, Klebsiella pneumoniae, Bacillus subtillis and Staphylococcus aureus.

Optionally, some scripts provide additional analysis:

- `kmer.py`: extraction of kmers and tandem repeats in the missing regions. Additionally, performs a pairwise comparison between kmers of missing and mapped regions using Sourmash.
- `coverage.py`: calculates the average coverage of the missing and mapped regions with a provided BED file.
- `quastunmap.py`: performs a genome quality assessment for the missing regions to the reference and provides a genome viewer using QUAST and Icarus.

## Getting Started

### System Requirements

- Linux 64-bit and OS X are supported.

#### General dependencies

- Python3 (3.4+)
- shutil
- shlex
- Seaborn
- BioPython
- progressbar
- matplotlib
- Pandas
- numpy
- sourmash

#### Third-party tools

- Mauve or pogressiveMauve (v2.4.0) (https://darlinglab.org/mauve/mauve.html)
- Prokka (v1.14.5) (https://github.com/tseemann/prokka)
- BLAST+
- QUAST (v5.02) (https://github.com/ablab/quast)
- SAMtools (v1.7) (http://samtools.sourceforge.net/)
- Tandem Repeats Finder (https://tandem.bu.edu/trf/trf.download.html) (v4.09) Note: SASpector run TRF with the name `trf409.linux64`.

### Installation

We recommend to install SASpector using pip:

`pip install SASpector`

## Command line options

For basic functionalities, run `SASpector`:

```
usage: SASpector - Short-read Assembly inSpector [-h] [-p PREFIX]
                                                 [-dir OUTDIR] [-f [LENGTH]]
                                                 [-db [PROTEIN-DB]] [-k [K]]
                                                 [-q] [-c [BAMFILE]]
                                                 reference contigs

positional arguments:
  reference             Reference genome FASTA file, e.g. from hybrid
                        assembly. If the file contains multiple seqences, only
                        the first one is used, so make sure to concatenate if
                        needed.
  contigs               Short-read assembly FASTA file as contigs/draft genome.

optional arguments:
  -h, --help            show this help message and exit
  -p PREFIX, --prefix PREFIX
                        Genome ID
  -dir OUTDIR, --outdir OUTDIR
                        Output directory
  -f [LENGTH], --flanking [LENGTH]
                        Add flanking regions to the extracted missing regions
                        [Default = 100 bp]
  -db [PROTEIN-DB], --proteindb [PROTEIN-DB]
                        BLAST protein database FASTA file to use for checking
                        the Prokka gene prediction
  -k [K], --kmers [K]   Choose k to calculate kmer frequencies
  -q, --quast           Run QUAST with unmapped regions against reference
                        assembly
  -c [BAMFILE], --coverage [BAMFILE]
                        Run SAMtools bedcov to look at short-read coverage in
                        the missing regions. Needs alignment of reads to the
                        reference genome in BAM format

  ```

First, Mauve performs an alignment of both genomes with the progressiveMauve algorithm. It will generate a subdirectory prefix.alignment with several output files but most importantly the backbone file with coordinates of the mapped and unmapped regions in the reference genome.

Afterwards, this script will parse the backbone file and extract the sequences that are not covered in the short-read assembly from the reference genome. They are written to a multi-fasta file with the prefix and coordinates in the headers, which is done equally for the mapped and conflict regions (regions that didn't map correctly due to gaps or indels). Two tab-delimited summary files are generated in a subdirectory called summary. One for the reference, with the amount of gapped and ungapped regions, the fraction of the reference genome that they represent, the GC content and the length. The other one for the unmapped regions, with for each region the GC content and length and then for each amino acid the occurence frequency averaged over all 6 reading frames. As an optional input, the user can add flanking regions to the extracted missing regions.

Finally, SASpector will predict genes that are in the missing regions using Prokka and if a protein FASTA file database is provided, SASpector will BLAST the output sequences from Prokka to the database generating a tab-delimited summary with the hits of the sequences. You can use our defined database `saspector_proteindb.fasta`.

As optional analysis:

- kmer analysis and tandem repeats: if a kmer size is provided, SASpector will calculate the frequency of the kmers per missing regions and will generate summary tables and barplots for those kmers. Additionally, it will run Tandem Repeats Finder and will generate HTML reports for the missing regions with tandem repeats. Finally, SASpector will perform a pairwise comparison between kmers of missing regions and mapped regions (k-size = 31) for comparative studies, using sourmash.

- Coverage analysis: if a BAM file is provided, SASpector will calculate the average coverage of the missing and the mapped regions, using SAMtools. It will generate a sorted BAM file and tab-delimited reports of the coverage for both regions.

- QUAST: SASpector will run QUAST for the missing regions against the reference genome for genome quality assessment and will provide Icarus as genome viewer.

## Usage

Before running SASpector, be sure to have your reference genome as a single FASTA file not a multi-FASTA file. If your reference genome is a multi-FASTA file (e.g. output from Unicycler), you can concatenate your sequences using Union command by EMBOSS.

```
SASpector [Reference genome].fasta [Contigs].fasta -p [Genome ID] -dir [Output directory] -f [Length] -db [Protein database].fasta -k [kmer size] -c [reference genome].bam -q

```

## Output

```
[Output directory]/
  [Genome ID]_unmappedregions.fasta    FASTA file of the missing regions
  [Genome ID]_mappedregions.fasta      FASTA file of the mapped regions
  [Genome ID]_conflictregions.fasta    FASTA file of regions that did not map correctly
  [Genome ID]_referencesummary.tsv     tab-delimited summary report of the reference genome
  [Genome ID]_unmapsummary.tsv         tab-delimited summary report of the missing regions
  [Genome ID]_length_missing.jpg       Distribution plot of the length of the missing missing regions
  [Genome ID]_gc_content_missing.jpg   Distribution plot of the GC content of the missing regions
  [Genome ID]_codons_missing.jpg       Boxplot of the averaged frequency for each amino acid (for all 6 reading frames) of the missing regions
  alignment/
    [Genome ID].alignment              Alignment output from progressiveMauve
    [Genome ID].bbcols                 Coordinates of the mapped and unmapped regions from Mauve (not used)
    [Genome ID].backbone               Coordinates of the mapped, unmapped and conflicts regions from progressiveMauve
    [Genome ID].sslist                 SSlists of short-reads assembly and reference genome
  genesprediction/
    [Genome ID].predictedgenes.gff                    Genes annotation GFF3 file
    [Genome ID].predictedgenes.gbk                    Genbank file
    [Genome ID].predictedgenes.fna                    Nucleotide FASTA file of the missing regions
    [Genome ID].predictedgenes.faa                    Protein FASTA file of the translated CDS sequences
    [Genome ID].predictedgenes.ffn                    Nucleotide FASTA file of all the prediction transcripts
    [Genome ID].predictedgenes.sqn                    Sequin file for submission to Genbank
    [Genome ID].predictedgenes.fsa                    Nucleotide FASTA file of the missing regions, used by 'tbl2asn' for the .sqn file
    [Genome ID].predictedgenes.tbl                    Feature table file, used by 'tbl2asn' for the .sqn file
    [Genome ID].predictedgenes.err                    NCBI discrepancy report
    [Genome ID].predictedgenes.log                    Output report of Prokka during its run
    [Genome ID].predictedgenes.txt                    Statistics of the annotated features
    [Genome ID].predictedgenes.tsv                    tab-delimited report of all features
    [Genome ID]_blastxresults.tsv      tab-delimited report of BLASTX
  kmer/
    *.tsv                   tab-delimited reports of the frequency of kmer per missing region
    *.jpg                   Barplots of the frequency of kmer per missing region
    [Genome ID]_sourmash               tab-delimited output of pairwise comparison between missing regions and mapped regions
    _distances.tsv
    sourmash                Clustermap of pairwise comparison
    _clustermap.jpg
  trf/
    *.html                  Tandem Repeat Finder HTML interactive reports
  coverage/
    [Genome ID].sorted.bam             Sorted BAM file of the reference genome
    [Genome ID].sorted.bam.bai         Sorted BAM index file
    [Genome ID]_mappedregions.bed      BED file of the mapped regions
    [Genome ID]_unmappedregions.bed    BED file of the missing regions
    [Genome ID]_mapcvg.tsv             tab-delimited report of the average coverage of the mapped regions
    [Genome ID]_unmappedcvg.tsv        tab-delimited report of the average coverage of the missing regions
    [Genome ID]_coverageresults.tsv    tab-delimited summary report of the average coverage, total depth per base and locations for both regions
    coverage_boxplots.jpg      Boxplot comparison of the average coverage for both regions
  quast/
    report.txt              QUAST summary table
    report.tsv              tab-delimited summary report
    report.tex              LaTex summary report
    report.html             HTML interactive report, includes all tables and plots for statistics
    report.pdf              PDF report
    icarus.html             Icarus genome viewer

```
## Support

For questions or issues, go to this repository Issues tab.

## References
- Altschul, S. F., Gish, W., Miller, W., Myers, E. W., & Lipman, D. J. (1990). Basic local alignment search tool. Journal of Molecular Biology, 215(3), 403–410.
- Darling, A. C. E. (2004). Mauve: Multiple Alignment of Conserved Genomic Sequence With Rearrangements. Genome Research, 14(7), 1394–1403.
- Brown and Irber (2016), sourmash: a library for MinHash sketching of DNA Journal of Open Source Software, 1(5), 27.
- Pierce NT, Irber L, Reiter T et al. Large-scale sequence comparisons with sourmash [version 1; peer review: 2 approved]. F1000Research 2019, 8:1006
- Gurevich, A., Saveliev, V., Vyahhi, N., & Tesler, G. (2013). QUAST: Quality assessment tool for genome assemblies. Bioinformatics, 29(8), 1072–1075.
- Li H.*, Handsaker B.*, Wysoker A., Fennell T., Ruan J., Homer N., Marth G., Abecasis G., Durbin R. and 1000 Genome Project Data Processing Subgroup (2009) The Sequence alignment/map (SAM) format and SAMtools. Bioinformatics, 25, 2078-9. 
- Torsten Seemann, Prokka: rapid prokaryotic genome annotation, Bioinformatics, Volume 30, Issue 14, 15 July 2014, Pages 2068–2069.
- Benson G. (1999). Tandem repeats finder: a program to analyze DNA sequences. Nucleic acids research, 27(2), 573–580. 
