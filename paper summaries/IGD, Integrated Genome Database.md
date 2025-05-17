---
modified: 2025-05-16T21:21:48-06:00
---
A follow-up to giggle which is based on B+ trees, but their tool is based on linear binning.

https://academic.oup.com/bioinformatics/article/37/1/118/6050710
> IGD: high-performance search for large-scale genomic interval datasets.
# Abstract
> Databases of large-scale genome projects now contain thousands of genomic interval datasets. These data are a critical resource for understanding the function of DNA. However, our ability to examine and integrate interval data of this scale is limited. Here, we introduce the integrated genome database (IGD), a method and tool for searching genome interval datasets more than three orders of magnitude faster than existing approaches, while using only one hundredth of the memory. IGD uses a novel linear binning method that allows us to scale analysis to billions of genomic regions.
# Why
The common type of data when comparing experimental results involving genomic analysis is interval sets (a set of genomic intervals)
> genomic intervals, defined by a chromosome plus start and end coordinates

import operation that requires optimization:
> computing the intersections of two interval sets.

why?
> common for a variety of use cases such as determining overlaps between sets of transcription factor binding sites, plotting the genome-wide distribution of a set of SNPs or regions, or assigning genomic annotations to query regulatory elements

that operation would allow us to compare: *bed file to bed file* or more importantly: *bed file to bed **files***(interval database search). <u>How giggle comes in</u>:
> take advantage of integrating across interval sets in the database, which can be done by creating a combined index that spans individual interval sets. This is the approach taken by GIGGLE, which is much faster for searching a large database than a naive file-to-file loop
# Methods
- An analogy comes to mind: hash maps vs trees as underlying approaches to implement a dictionary.
  The dictionary-like lookup operation is "what intervals exist at this region?". Giggle is analogous to a tree-based approach and IGD is analogous to a hash-based approach.
- Both approaches do not operate per each file, but instead (more efficiently) operate across the entire database of files. They each do a loop through the amount of intervals in the query set.

**Index**
1) Consider the genome as broken into fixed-size tiles. Then for each interval in the database of bed files, find the tile that contains their start position. (Probably with an O(1) lookup based on rounding the start coordinate)
2) Iterate until the end-tile (that contains the end coordinate), adding the interval to all tiles en route. **This means intervals can be repeatedly stored which increases index size.**
3) All intervals within the tile must be sorted based on start coordinate.
**Query**
- We've already got a finished index of tiles that each contain a set of "IGD data elements".
<u>IGD data elements:</u>
> the original genomic region (start and end coordinates) and a value (signal level). For compatibility to other existing search tools, IGD also supports a data element type without the signal value.

The paper does not clarify the utility of the signal value.

1) for the query set, for each interval within the query set:
   find the tile that contains the start coordinate
2) within the start tile, binary scan to the first interval with a start coordinate later than the query's start coordinate (note that the intervals within a tile are sorted by start coordinate)
3) iterate to the tile that contains query interval's the end coordinate, taking intersecting interval en route;
   **Skipping intervals (within a tile) that intersect the left boundary of that tile** because…
![[Pasted image 20250515135121.png]]
In order to avoid double-counting (which could occur due to a problem interval being stored in both tiles on either side of a tile-boundary), we only take the interval on the left (that does not intersect the left boundary of its tile whereas the right interval does intersect the left boundary of its tile)
# Results
Overall, approaches that attempt to do a search per file (not leveraging efficiency gains due to searching across the whole database at once) pale in comparison.
> For query sets with 100 or fewer intervals, IGD is four to five orders of magnitudes faster than file-to-file approaches AIList and BEDTools

But for giggle (which they are competing with primarily)
> IGD is two orders of magnitude faster still

and
> For larger query intervals, IGD is also several orders of magnitude faster than any existing approach

in terms of disk space and index-creation time,
> For the full set UCSC data source, GIGGLE uses 640 GB disc space while IGD only needs 98 GB. IGD is also faster in database creation: the complete database takes IGD about 2 hours to create, while GIGGLE takes more than 4 hours

![[Screenshot 2025-05-15 at 1.56.13 PM.png]]