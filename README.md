# SpliceVaultPE

This repo contains code to infer likely pseudoexons from snapcount-processed GTEx RNA-seq data. Possible pseudo-exons are inferred from splice-junctions with one end corresponding to an annotated intron and the other end falling within that intron. 

## Source files

[data/utils](data/utils)/ contains required files. 

| Data source | URL                                                  | Notes                                                        | Where to place the file? |
| ----------- | ---------------------------------------------------- | ------------------------------------------------------------ | ------------------------ |
| GTEx v8     | http://snaptron.cs.jhu.edu/data/gtexv2/junctions.bgz | Filtered to chromosomes 1-22, X, Y (filename junctions_1to22XY_gtexv2.tsv.gz and junctions_1to22XY_gtexv2_samples.tsv.gz) | src / gtex /             |

Filtered samples file ([data/utils/gtexv2/junctions_1to22XY_gtexv2_samples.tsv.gz](data/utils/gtexv2/junctions_1to22XY_gtexv2_samples.tsv.gz)/) was further split into single chromosomes to allow easier processing in the [finding_pseudos](finding_pseudos.Rmd) notebook, using the linux command(s):

gunzip -c junctions_1to22XY_gtexv2_samples.tsv.gz | tr ' ' '\t' | awk -F "\t" '$1 == "chr1" {print}' | gzip -c > sample_files/junctions_chr1_gtexv2_samples.tsv.gz

etc. All samples files split into chromosomes were placed in the [data/utils/gtexv2/sample_files/](data/utils/gtexv2/sample_files/) directory. 

## Steps to create SpliceVaultPE

**filter_junctions.Rmd**

Pairs of splice-junctions are filtered to match 4 criteria: 

1. both start and end of the split-read are within an annotated MANE intron
2. split-read represents an annotated donor and unannotated acceptor falling within the intron. OR, split-read represents an unannotated donor falling within the intron and annotated acceptor.
3. use of unannotated splice-junctions in both split-reads could theoretically result in a pseudo-exon of non-negative length between them. 

output stored in [data/interim/candidate_PEs.tsv.gz](data/interim/candidate_PEs.tsv.gz)/.

**finding_pseudos.Rmd**

Possible pseudo-exons are then ranked according to how many GTEx samples they co-occur in (i.e. number of samples that contain at least 1 read representing both splice-junctions).

## Results

Final calculated candidate pseudo-exons are stored in [data/final/SpliceVaultPE_gtexv2.tsv.gz](data/final/SpliceVaultPE_gtexv2.tsv.gz)/. Calculated in hg38



| Column Name          | Description                                                  |
| -------------------- | ------------------------------------------------------------ |
| border_match_intron  | The MANE annotated intron within which the candidate pseudo-exon would be. format: *transcriptId*_ int _*intronNumber* e.g. ENST00000357033.9_int_1 for the first intron of DMD. |
| chrom                | Chromosome                                                   |
| strand               | strand (+ or -)                                              |
| annotated_start      | genomic position of the start of the annotated intron        |
| annotated_end        | genomic position of the end of the annotated intron          |
| PE_start             | genomic position of the start of the candidate pseudo-exon   |
| PE_end               | genomic position of the end of the candidate pseudo-exon     |
| PE_length            | distance between PE_start and PE_end                         |
| sample_overlap       | Each pseudo-exon is represented by two split-reads. sample_overlap is the number of samples in which both of these split-reads are seen in at least 1 read. |
| prop_samples_overlap | sample_overlap / min(samples containing first split-read, samples containing second split-read) |
| sample_count_ann     | The number of samples in which the annotated intron was seen in at least 1 read. |
| prop_samples_ann     | sample_overlap / sample_count_ann                            |



## 
