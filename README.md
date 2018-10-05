# methPattern
*methPattern* is a pipeline for working with Whole Genome Bisulfite Sequenced (WGBS) data.
It converts bam files to compact presentation formats, keeping only data regarding the CpG sites.
The CpG sites are fixed, indexed and named CpG1, CpG2, …, CpG28217448, according to their order on the human genome ([hg19](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.13/)).
 
### Input format:
[bam file](https://samtools.github.io/hts-specs/SAMv1.pdf), generated from fastq using bwa-meth, or bismark, for example.
### Output formats:
methPattern generates 3 output files from a given bam file:
1. beta
2. pat.gz
3. unq.gz

## beta file
beta file is the simplest representation of methylation data. It is a binary file with a fixed size of 56,434,896 bytes (~54MB), holding a
 matrix of uint8 values with dimensions of (28,217,448 x 2).
For each of the 28,217,448 CpG sites in hg19, it holds 2 values: the #meth and #covered. Meaning, the i'th row in the matrix corresponds to the i'th CpG site:
- *#meth*: the number of times the i'th site (CpGi) site is observed in a methylated state.
- *#coverage*: the total number of times i'th site (CpGi) site is observed. #coverage==0 is equivalent to a missing value (NaN).

CpGi's beta value is obtained by dividing *#meth*/*#coverage*.

### reading a beta file, using
### python
```python
>>> import numpy as np
>>> content = np.fromfile(PATH, dtype=np.uint8).reshape((-1, 2))
>>> np.mean(content[:, 1])   # print average coverage
94.841405643770472
```

### matlab
```matlab
fid=fopen(PATH,'r'); content=fread(fid); fclose(fid); A=reshape(content,2,[])';
```


### R
```R
> fname <- PATH
> N <- file.info(fname)$size
> content <- matrix(readBin(fname, "integer", N, size=1), N / 2, 2, byrow=TRUE)
```

**Note**: this uint8 format limits values to range [0,255]. In case a CpG site appears over 255 times in the input bam, its representation is normalized to this range. For example, if site CpG100 appeared 510 times, from which 100 times it was methylated, the 100'th row will be (50, 255).


## pat.gz file
A gzipped tab separated file with four columns: chromosome, CpG_index, methylation_pattern, and count.
Each line in the file stands for a single read from the bam file. A line shows the methylation pattern of the read. Every CpG site from the read is represented as a single character: 
- 'C' for methylated
- 'T' for unmethylated
- '.' for unknown state

The rest of the nucleotides, and therefore the genomic distances between adjacent CpG sites, are ignored. If a read contains no CpG sites with known methylation state, it is ignored.
In case it's pair-end, the two paired lines from the bam file are merged into a single line in the pat file (see example).

- *chrom*: value is a string from _{chr1, chr2, …, chrX, chrY, chrM}_
- *CpG_index*: integer in range [1,28217448]. The index of the first site occurring on the current read. The file is sorted by this column.
- *methylation pattern*: a string of characters from {'C', 'T', '.'} representing the methylation pattern on the current read.
- *count*: The number of times a read with these exact values (chrom, CpG_index and methylation pattern) occurred.


#### Examples
#### Single-end file
```
// bam file:
r001	2	chr1	10751	1	12M	=	0	12	CCGCGCCAACGC	JF-JF<<7<A7J

// pat file:
chr1	47	CCTC	1
```
The read covers 4 CpG sites: CpG47 (methylated), CpG48 (methylated), CpG49 (*unmethylated*), and CpG50 (methylated)

#### Paired-end file
```
// bam file:
r001	163	chr1	10751	1	12M	=	10778	40	CCGCGCCAACGC	JF-JF<<7<A7J
r001	83	chr1	10778	1	12M	=	10751	-40	CACCGCGCCGAC	FJJFJJFFA-AA

// pat file:
chr1	47	CCTC..TCCC	1
```
The pair of reads covers 40bp and 10 CpG sites. However, there is a small gap between them, so two sites (CpG51, CpG52) have unknown methylation state.


**Note:** The pat file is sorted by CpG_index column, which is different from the genome browser order (sort -k1,1 -k2,2n)

## unq.gz file
similar to pat.gz, but with two extra fields:
- *start_loci*: the genomic position this reads starts at.
- *read_length*: read length in base-pairs.
The count field was removed, since we do not expect to see two (or more) reads with identical values.
// todo: right? should we return the count field? should we drop duplicated lines?

#### remarks / future work:
- patter makes no quality filtering. Ignors both mapping quality (MAPQ) and base quality (QUAL).
- in case 2 reads from the same pair differ, it outputs '.', instead of taking the one with higher quality
- should skip the first/last X characters in every read, because of the methylation bias
- remove leading dots in pat format (e.g '..CC' shoud be 'CC') todo: patter should take the genoma.fa reference as input


