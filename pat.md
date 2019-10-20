# Random Access
_pat_ files are often large files, so in order to quickly access an arbitary region, we use the fact it's sorted, [bgzipped](http://www.htslib.org/doc/bgzip.html) and indexed.
From bgzip documentation:
>  Bgzip compresses files in a similar manner to, and compatible with, gzip(1). The file is compressed into a series of small (less than 64K) 'BGZF' blocks. This allows indexes to be built against the compressed file and used to retrieve portions of the data without having to decompress the entire file.

tabix is able to quickly retrieve data lines overlapping regions specified in the format "chr:beginPos-endPos". We abuse _tabix_ a little bit, by setting the CpG index as both beginPos and endPos, and add some extra work to include reads that start before, but end within, the requested region.
`--strict` flag means we trim reads such that only the part covering the region is returned.



