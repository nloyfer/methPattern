# Random Access
The _pat_ file is sorted by CpG index, and is gzipped. It is typically a large file, so accessing reads in a specific region may take a while.

Access reads in arbitary locations in the _pat_ file, using [bgzip](http://www.htslib.org/doc/bgzip.html) to compress the pat file. From bgzip documentation:

>  Bgzip compresses files in a similar manner to, and compatible with, gzip(1). The file is compressed into a series of small (less than 64K) 'BGZF' blocks. This allows indexes to be built against the compressed file and used to retrieve portions of the data without having to decompress the entire file.

bgzip compresses the file in a blocks structure, which allows for accessing arbitary locations in the file, using a small index file it generates (`*pat.gz.csi`). _patPeek_ exploits this feature to perform binary search on the _pat_ file, looking for the desired region.



