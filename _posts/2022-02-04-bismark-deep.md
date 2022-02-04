---
layout: post
title: Going deep into Bismark
date: '2022-01-22'
categories: oyster
tags: Bismark
---


Taking a deeper look at every step. Note this is single-end sequence data.


```
# Directories and programs
bismark_dir="/gscratch/srlab/programs/Bismark-0.23.0"
bowtie2_dir="/gscratch/srlab/programs/bowtie2-2.3.4.1-linux-x86_64/"
samtools="/gscratch/srlab/programs/samtools-1.9/samtools"
reads_dir="/gscratch/srlab/sr320/data/oil/"
genome_folder="/gscratch/srlab/sr320/data/Cvirg-genome/"

source /gscratch/srlab/programs/scripts/paths.sh

# ${bismark_dir}/bismark_genome_preparation \
# --verbose \
# --parallel 28 \
# --path_to_aligner ${bowtie2_dir} \
# ${genome_folder}




find ${reads_dir}*fastq.gz \
| xargs basename -s fastq.gz | xargs -I{} ${bismark_dir}/bismark \
--path_to_bowtie ${bowtie2_dir} \
-genome ${genome_folder} \
-p 4 \
--non_directional \
${reads_dir}{}fastq.gz
```

Genome prep took about 15 minutes..



Two files are produced, eg


```
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.bam
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2_SE_report.txt
```

Here is what the report looks like


```
Bismark report for: /gscratch/srlab/sr320/data/oil/20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC.fastq.gz (version: v0.23.1)
Option '--non_directional' specified: alignments to all strands were being performed (OT, OB, CTOT, CTOB)
Bismark was run with Bowtie 2 against the bisulfite genome of /gscratch/srlab/sr320/data/Cvirg-genome/ with the specified options: -q --score-min L,0,-0.2 -p 4 --reorder --ignore-quals

Final Alignment report
======================
Sequences analysed in total:	21462840
Number of alignments with a unique best hit from the different alignments:	6553577
Mapping efficiency:	30.5%
Sequences with no alignments under any condition:	11327219
Sequences did not map uniquely:	3582044
Sequences which were discarded because genomic sequence could not be extracted:	0

Number of sequences with unique best (first) alignment came from the bowtie output:
CT/CT:	240748	((converted) top strand)
CT/GA:	242445	((converted) bottom strand)
GA/CT:	3021088	(complementary to (converted) top strand)
GA/GA:	3049296	(complementary to (converted) bottom strand)

Final Cytosine Methylation Report
=================================
Total number of C's analysed:	133369402

Total methylated C's in CpG context:	21023431
Total methylated C's in CHG context:	5237928
Total methylated C's in CHH context:	16143340
Total methylated C's in Unknown context:	38604

Total unmethylated C's in CpG context:	7515260
Total unmethylated C's in CHG context:	29065953
Total unmethylated C's in CHH context:	54383490
Total unmethylated C's in Unknown context:	77386

C methylated in CpG context:	73.7%
C methylated in CHG context:	15.3%
C methylated in CHH context:	22.9%
C methylated in Unknown context (CN or CHN):	33.3%

Bismark completed in 0d 0h 36m 16s
```


What happens if we make it directional



```
find ${reads_dir}*fastq.gz \
| xargs basename -s fastq.gz | xargs -I{} ${bismark_dir}/bismark \
--path_to_bowtie ${bowtie2_dir} \
-genome ${genome_folder} \
-p 4 \
${reads_dir}{}fastq.gz
```

Got me

```
Final Alignment report
======================
Sequences analysed in total:	21462840
Number of alignments with a unique best hit from the different alignments:	977102
Mapping efficiency:	4.6%
Sequences with no alignments under any condition:	19722987
Sequences did not map uniquely:	762751
Sequences which were discarded because genomic sequence could not be extracted:	0

Number of sequences with unique best (first) alignment came from the bowtie output:
CT/CT:	487779	((converted) top strand)
CT/GA:	489323	((converted) bottom strand)
GA/CT:	0	(complementary to (converted) top strand)
GA/GA:	0	(complementary to (converted) bottom strand)
```


Lets get the alignment up a bit and go back to non-directional



```
find ${reads_dir}*fastq.gz \
| xargs basename -s fastq.gz | xargs -I{} ${bismark_dir}/bismark \
--path_to_bowtie ${bowtie2_dir} \
-genome ${genome_folder} \
-score_min L,0,-0.6 \
-non_directional \
-p 4 \
${reads_dir}{}fastq.gz
```

That got us up to reasonable percent mapping

```
Bismark report for: /gscratch/srlab/sr320/data/oil/20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC.fastq.gz (version: v0.23.1)
Option '--non_directional' specified: alignments to all strands were being performed (OT, OB, CTOT, CTOB)
Bismark was run with Bowtie 2 against the bisulfite genome of /gscratch/srlab/sr320/data/Cvirg-genome/ with the specified options: -q --score-min L,0,-0.6 -p 4 --reorder --ignore-quals

Final Alignment report
======================
Sequences analysed in total:	21462840
Number of alignments with a unique best hit from the different alignments:	10790387
Mapping efficiency:	50.3%
Sequences with no alignments under any condition:	5480515
Sequences did not map uniquely:	5191938
Sequences which were discarded because genomic sequence could not be extracted:	0

Number of sequences with unique best (first) alignment came from the bowtie output:
CT/CT:	459450	((converted) top strand)
CT/GA:	466042	((converted) bottom strand)
GA/CT:	4891423	(complementary to (converted) top strand)
GA/GA:	4973472	(complementary to (converted) bottom strand)

Final Cytosine Methylation Report
=================================
Total number of C's analysed:	212797610

Total methylated C's in CpG context:	31770372
Total methylated C's in CHG context:	8211495
Total methylated C's in CHH context:	25943978
Total methylated C's in Unknown context:	310267

Total unmethylated C's in CpG context:	14009418
Total unmethylated C's in CHG context:	46372561
Total unmethylated C's in CHH context:	86489786
Total unmethylated C's in Unknown context:	682173

C methylated in CpG context:	69.4%
C methylated in CHG context:	15.0%
C methylated in CHH context:	23.1%
C methylated in Unknown context (CN or CHN):	31.3%


Bismark completed in 0d 0h 55m 8s
```


What is the efficiency?

```
cat *_SE_report.txt | grep "Mapping efficiency"
Mapping efficiency:	50.3%
Mapping efficiency:	47.2%
Mapping efficiency:	50.9%
Mapping efficiency:	38.1%
```




Next up is deduplicating

```
find *.bam | \
xargs basename -s .bam | \
xargs -I{} ${bismark_dir}/deduplicate_bismark \
--bam \
--samtools_path ${samtools} \
--single \
{}.bam
``

This provides two files

```
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.bam
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplication_report.txt
```


Report

```
Total number of alignments analysed in 20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.bam:	10790387
Total number duplicated alignments removed:	5600409 (51.90%)
Duplicated alignments were found at:	1680694 different position(s)

Total count of deduplicated leftover sequences: 5189978 (48.10% of total)

```



```
${bismark_dir}/bismark_methylation_extractor \
--bedGraph \
--multicore 28 \
--buffer_size 75% \
--samtools_path ${samtools} \
*deduplicated.bam
```

This produces


```
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.bedGraph.gz
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.bismark.cov.gz
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.M-bias.txt
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated_splitting_report.txt
```
Plus a lot of other files


```
CHG_CTOB_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CHG_CTOT_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CHG_OB_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CHG_OT_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CHH_CTOB_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CHH_CTOT_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CHH_OB_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CHH_OT_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CpG_CTOB_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CpG_CTOT_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CpG_OB_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
CpG_OT_20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.txt
```



Splitting report

```
20150414_trimmed_2112_lane1_HB16_Oil_25000ppm_TTAGGC_bismark_bt2.deduplicated.bam

Parameters used to extract methylation information:
Bismark Extractor Version: v0.23.1
Bismark result file: single-end (SAM format)
Output specified: strand-specific (default)


Processed 5189978 lines in total
Total number of methylation call strings processed: 5189978

Final Cytosine Methylation Report
=================================
Total number of C's analysed:	100573067

Total methylated C's in CpG context:	12227583
Total methylated C's in CHG context:	5548341
Total methylated C's in CHH context:	18517042

Total C to T conversions in CpG context:	7392033
Total C to T conversions in CHG context:	18592605
Total C to T conversions in CHH context:	38295463

C methylated in CpG context:	62.3%
C methylated in CHG context:	23.0%
C methylated in CHH context:	32.6%
```

Lets see if we can get simpler here. (Just dropping bedgraph we will be making ourselves later).

```
${bismark_dir}/bismark_methylation_extractor \
--multicore 28 \
--buffer_size 75% \
--samtools_path ${samtools} \
*deduplicated.bam
```
