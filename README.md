## Docker container
This is deicated to run the Docker container for RNA Seq. Data analysis Pipeline which runs with different kinds of softwares and images as follows:

## Pipeline summary

1. Download FastQ files via SRA, ENA or GEO ids and auto-create input samplesheet ([`ENA FTP`](https://ena-docs.readthedocs.io/en/latest/retrieval/file-download.html); *if required*)
2. Merge re-sequenced FastQ files ([`cat`](http://www.linfo.org/cat.html))
3. Read QC ([`FastQC`](https://www.bioinformatics.babraham.ac.uk/projects/fastqc/))
4. UMI extraction ([`UMI-tools`](https://github.com/CGATOxford/UMI-tools))
5. Adapter and quality trimming ([`Trim Galore!`](https://www.bioinformatics.babraham.ac.uk/projects/trim_galore/), [`Cutadapt`](https://cutadapt.readthedocs.io/en/stable/))
6. Choice of multiple alignment and quantification routes:
    1. [`STAR`](https://github.com/alexdobin/STAR) -> [`featureCounts`](http://bioinf.wehi.edu.au/featureCounts/)
    2. [`Tophat2`](http://ccb.jhu.edu/software/tophat/manual.shtml#toph)
    3. [`HiSAT2`](https://ccb.jhu.edu/software/hisat2/index.shtml) -> [`featureCounts`](http://bioinf.wehi.edu.au/featureCounts/)
7. Sort and index alignments ([`SAMtools`](https://sourceforge.net/projects/samtools/files/samtools/))
8. UMI-based deduplication ([`UMI-tools`](https://github.com/CGATOxford/UMI-tools))
9. Create bigWig coverage files ([`BEDTools`](https://github.com/arq5x/bedtools2/), [`bedGraphToBigWig`](http://hgdownload.soe.ucsc.edu/admin/exe/))
10. Extensive quality control:
    1. [`RSeQC`](http://rseqc.sourceforge.net/)
    2. [`DESeq2`](https://bioconductor.org/packages/release/bioc/html/DESeq2.html)
11. Present QC for raw read, alignment, gene biotype, sample similarity, and strand-specificity checks ([`R`](https://www.r-project.org/))
