# Random Access
_pat_ files are often large files, so in order to quickly access an arbitary region, we use the fact it's sorted, [bgzipped](http://www.htslib.org/doc/bgzip.html) and indexed.
From bgzip documentation:
>  Bgzip compresses files in a similar manner to, and compatible with, gzip(1). The file is compressed into a series of small (less than 64K) 'BGZF' blocks. This allows indexes to be built against the compressed file and used to retrieve portions of the data without having to decompress the entire file.

bgzip compresses the file in a blocks structure, which allows for accessing arbitary locations in the file, using a small index file it generates (`*pat.gz.csi`). _patPeek_ exploits this feature to perform binary search on the _pat_ file, looking for the desired region.



