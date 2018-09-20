# methPattern
'methPattern' is a pipeline for working with Whole Genome Bisulfite Sequenced (WGBS) data.
It converts bam files to compact presentation formats.

### Notations
The CpG sites are indexed and named CpG1, CpG2, …, CpG28217448, according to their order on the human genome ([hg19](https://www.ncbi.nlm.nih.gov/assembly/GCF_000001405.13/)). In order to save space, our formats only keep data regarding these sites, ignoring other nucleotides.

### Input format:
[bam file](https://samtools.github.io/hts-specs/SAMv1.pdf), generated from fastq using bwa-meth, or bismark, for example.
### Output formats:
methPattern generates 3 output files from a given bam file:
1. beta
2. pat.gz
3. unq.gz

## beta file
beta file is the simplest representation of methylation data. It is a binary file with a fixed size of 56434896 bytes (~54MB), holding a
(28,217,448 x 2) matrix of uint8 values. The i'th row in the matrix contains two values, corresponding to the i'th CpG site on the human genome (hg19).

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

### matlab    // todo: test this, and ask Tommy to look at it (is there a more elegant way of reading it?)
```matlab
f = dir(PATH); 
fid=fopen(f.name);
content = fread(fid, f.bytes, 'uint8');
fclose(fid);
N = length(content) / 2,;
content = reshape(content, 2, N);
```


### R
```R
> fname <- PATH
> N <- file.info(fname)$size
> content <- matrix(readBin(fname, "integer", N, size=1), N / 2, 2, byrow=TRUE)
```

**Note**: this uint8 format limits the values to be under 256. In case a CpG site appear over 255 times in the input bam, its representation is normalized. For example, if site CpG100 appeared 510 times, from which 100 times it was methylated, the 100'th row will be (50, 255).

For each of the 28,217,448 CpG sites in the human genome (hg19), it holds 2 values: the #meth and #covered.
a binary file, 


## pat.gz file
A gzipped tab separated file with four columns: chromosome, CpG_index, methylation_pattern, count.
Each line in the file stands for a single read from the bam file. In case it's pair-end, the two paired lines from the bam file are merged into a single line in the pat file (see example).
A line shows the methylation pattern of the read. Every CpG site from the read is represented as a single character: 'C' for methylated, 'T' for unmethylated, or '.' for unknown. The rest of the nucleotides are ignored. If a read contains no CpG sites with known methylation state, it is ignored.

Example:

- chrom: value is a string from {chr1, chr2, …, chrX, chrY or chrM}
- CpG_index: integer in range [1-28217448]. The index of the first site occurring on the current read. The file is sorted by this column.
- methylation pattern: a string of characters from {'C', 'T', '.'} representing the methylation pattern on the current read.

Note: order of pat file is different from the genome browser order (sort -k1,1 -k2,2n)
Sorted by CpG_index column
Add example of paired end
Remove 'D' character from patter.



## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/nloyfer/methPattern/edit/master/README.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/nloyfer/methPattern/settings). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://help.github.com/categories/github-pages-basics/) or [contact support](https://github.com/contact) and we’ll help you sort it out.
