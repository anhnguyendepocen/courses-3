---
layout: default
title:  'Part 3: Single cell genome assembly using SPAdes'
---

# Part 3: Single cell genome assembly
---

<!-- <p class="bg-warning">If you get disconnected from Uppmax [click here](lostConnection) to know how to get back </p> -->
<div class="alert alert-info">
  <a href="#" class="close" data-dismiss="alert" aria-label="close">&times;</a>
  <strong>Info!</strong> If you get disconnected from Uppmax <a href="lostConnection"><strong>click here</strong></a> to know how to get back to it.
</div>

## 3.x.x Merging reads with SeqPrep

**This part is only for the ones that will work on Merged reads. Skip this if you do the assembly on the raw reads**  
To merge read pairs that have significant overlaps, we will use the tool called *'SeqPrep'*. 

First, create a folder where to store the output then merge the reads:

```sh
mkdir merged_reads
SeqPrep -q 30 -f G5_${sample}_1.fastq -r G5_${sample}_2.fastq \
-1 merged_reads/G5_${sample}_merge_1.fastq.gz \
-2 merged_reads/G5_${sample}_merge_2.fastq.gz \
-s merged_reads/G5_${sample}_merged.fastq.gz
```

Note that SeqPrep merges read pairs if overlaps between the read pairs are identified. Quality threshold of 30, for example, can be specified by ```-q 30``` flag 
for overlaps with some mismatches to be counted. It can also remove adapter sequences optionally.  


<div>
 <span style="float:left"><a class="btn btn-primary" onclick="javascript:history.go(-1)"> Previous page</a></span>
</div>