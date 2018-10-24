# Random Access
The _pat_ file is sorted by CpG index, and is gzipped. It is typically a large file, so accessing reads in a specific region may take a while.
Lets assume we are interested in all of the reads covering site CpG1000000.
Here is a naive approach:
```
zcat PAT_PATH | awk 'x=1000000 {if (x < $2) {exit}} {if ($2 + length($3) >= x) {print}}'
```
This idea: parsing the _pat_ file from the beginning, and stops when it gets to a read starting at a site greater than 1,000,000 (`if (x < $2) {exit}`). For each read, we know it starts before CpG1000000, and we check if it ends afterwards (`if ($2 + length($3) >= x) {print}`), which implies it includes site CpG1000000. This results in all the reads covering the desired site.


That is why we developed 
It is possible to access data in an arbitary location in the _pat_ file.
We developed a tool
using bgzip
http://www.htslib.org/doc/bgzip.html
gzi file
indexes files
