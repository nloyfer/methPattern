# Random Access
The _pat_ file is sorted by CpG index, and is gzipped. It is typically a large file, so accessing reads in a specific region may take a while.
Lets assume we are interested in all of the reads covering site x=CpG1000000.


Here is a naive approach:
```
zcat PAT_PATH | awk 'x=1000000 {if (x < $2) {exit}} {if ($2 + length($3) >= x) {print}}'
```
This idea: parse the _pat_ file from the beginning. Stop when we see a read starting at a site greater than x (`if (x < $2) {exit}`). Now, for each read we parse, we know it starts before x, so it's enough to check if it ends afterwards (`if ($2 + length($3) >= x) {print}`). This results in all the reads covering site x.
Running time can be significantly improved, for example using zgrep, but it is still slow comparing to _patPeek_.

# patPeek - 

Access reads in arbitary locations in the _pat_ file.
_patPeek_ uses [bgzip](http://www.htslib.org/doc/bgzip.html) to compress the pat file. From bgzip documentation:

>  Bgzip compresses files in a similar manner to, and compatible with, gzip(1). The file is compressed into a series of small (less than 64K) 'BGZF' blocks. This allows indexes to be built against the compressed file and used to retrieve portions of the data without having to decompress the entire file.

bgzip compresses the file in a blocks structure, which allows for accessing arbitary locations in the file, using a small index file it generates (`*pat.gz.gzi`). _patPeek_ exploits this feature to perform binary search on the _pat_ file, looking for the desired region.

We further improve performance by generating another index files (`pat.gz.pati`), listing the pat file offsets of multiple sites.

TODO: insert line to the github page of patPeek. Write usage and examples (inputs chr7:20,200-21,000, and 77000-77030 for example)


