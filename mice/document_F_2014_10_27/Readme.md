#Hotspot death
##Species: mice
##Run version: 2014_10_27
---



##Changelog
Date refers to first version with changes. If there is a plus after, then all subsequent versions have the change
* **2014_10_27** Redo significant value selection - do not require to the best on a lineage, but if OR>1, require that does not exist OR<1 with more significant p-value, for both clustering and motif selection
* **2014_10_27** Added variable pwmMinimum to control PWM plotting
* **2014_10_27** Use OR for better PWM plotting
* **2014_10_24** Added filtering folder for mice (ArrayNotFiltered95.0 used in all prior runs)
* **2014_10_01** Changed clustering method - increase difficulty such that current threshold for iteration is Bonferonni for this round and all other previous rounds. Also, only allow a motif to be included in one cluster per repeat background
* **2014_10_01** Fixed major repeat name problem - assigning motifs to repeats was fine, but got name wrong between assigning and reporting. Also affected generation of QQ plots in this summary document
* **2014_09_22** Added number of motif clusters in summary section  
* **2014_09_22** Add new AT  to GC plots, and details of making the plots
* **2014_09_18** Give the option to specify how to compare against other motifs for making 2x2 tables
* **2014_09_18** Don't perform motif loss counting if the the repeat background changes during the motif loss region
* **2014_09_17** For mice, set regions of the genome as uncallable if insufficient number of samples are callable in that region
* **2014_09_16** Clustering - changed it so motifs are only eligible to be added to a cluster if they have the best p-value among lineages for that motif
* **2014_09_15** added removal of A) nearby SNPs on the same lineage and B) too many SNPs in general locally, as well as check against expectations
* **2014_09_15** added proper support for mice for smaller number of musculus lineages
* **2014_09_13** x axis in left plot for comparison between two test p-values

------


##Important Parameters

|Parameter             |Value                      |Description                                                              |
|:---------------------|:--------------------------|:------------------------------------------------------------------------|
|mouseFilter           |ArrayNotFilteredAnnot295.0 |What folder used for mice                                                |
|mrle                  |10                         |Maximum run length less than or equal to                                 |
|ndge                  |0                          |Nuclotide diversity greater than or equal to                             |
|pwmMinimum            |0.1                        |When a PWM has 0 entry, what to reset the 0 values to                    |
|Klist                 |10                         |Range of K to use                                                        |
|gcW1                  |100                        |Range to test for GC increase                                            |
|gcW2                  |1000                       |Range to plot GC increases                                               |
|gcW3                  |10                         |Smoothing window for AT to GC plots                                      |
|ctge                  |10                         |For a p-value to be generated, each cell must have at least this number  |
|rgte                  |50                         |For a row to count, there must be this many entries                      |
|testingNames          |lin,shared,at              |(Short) names of the tests we will perform                               |
|plottingNames         |Lineage,Shared,AT to GC    |Plotting names of the tests we will perform                              |
|nTests                |3                          |Number of tests to be performed                                          |
|pThresh               |0.000152439024390244       |Initial threshold for p-value clstering                                  |
|mouseMatrixDefinition |narrow                     |How we define mice lineages                                              |
|nR1                   |12                         |If there are nR1 SNPs in nD1 bp, remove all SNPs                         |
|nD1                   |50                         |See above                                                                |
|nR2                   |7                          |For a lineage, if there are ge this many SNPs in nD2 bp, remove all SNPs |
|nD2                   |50                         |See above                                                                |
|removeNum             |0                          |How many samples are allowed to be uncallable                            |
|contingencyComparison |Same CpG, GC Content       |To what do we compare motifs                                             |

---


##Methods
###SNP Filtering

SNPs are filtered if they have any missingness among the lineages, do not agree with the species tree, are multi-allelic, or are heterozygous among homozygous animals. 

Subsequently, SNPs are removed if they are too close together. This is the last step in the procedure, ie after removing SNPs for reasons listed above. I removed any SNPs if there were nR1 SNPs within nD1 bases, and, following this, I removed lineage specific SNPs down any lineage if there were nR2 SNPs within nD2 bases. For example, if nR1 was 10 and nD1 was 50, and there was a cluster of 14 SNPs within 50 bases, all of the offending SNPs were removed. 

To get a sense of how many SNPs this removed for given parameter settings, I checked how many SNPs were filtered for a range of parameter settings for the smallest chromosome. I also compared this against expectations. To get an expectation, I simulated a pseudo-chromosome of results. I calculated the expected branch lengths of each lineage given the emprical data to this point, ie lineage 1 has 1 percent divergence against MRCA of the set of all lineages, lineage 2 has 0.5 percent divergence against MRCA, etc. Then, I decided whether each base was mutated according to the total branch length of the tree, and then, given a mutation, what branch it occured on with probabilities equal to each lineages share of the total tree length. 


![plot of chunk filterPlot](figure/filterPlot.png) 

###Testing - general

3 tests are run based on motifs which are lost at every SNP down a lineage. Note that where there are multiple SNPs within a motif size (ie two SNPs 5 bases away), I considered all motifs lost over that range. If there were any missing bases within that range, the motifs were not counted towards the testing. Further, if the repeat background changed during that range, the motifs were not counted towards the testing.

###Test 1 - Lineage - Motif loss within a lineage versus other lineages

Here I look at the number of motifs lost down a lineage versus all other lineages. The 2 * 2 test is therefore


|                     |Lineage |All other lineages |
|:--------------------|:-------|:------------------|
|Motif                |n1      |n2                 |
|Same CpG, GC Content |n3      |n4                 |

###Test 2 - Shared - Motif loss within a lineage versus ancestral counts

Here, I look at the number of motifs lost down a lineage versus the number present in its ancestor in the species tree. Then, we test the number of motifs lost vs the number which are not lost. The 2 * 2 test is therefore


|                     |Lost down this lineage |Not lost down this lineage |
|:--------------------|:----------------------|:--------------------------|
|Motif                |n1                     |n2                         |
|Same CpG, GC Content |n3                     |n4                         |

###Test 3 - AT to GC - Motif loss AT to GC counts versus local AT bases
  
Let gcW1=100 be the distance to test for increased presence of AT to GC
bases. Then around every motif that is lost, we count three numbers

* The number of AT to GC changes in nearest gcW bases away from the motif, not including the motif itself. For example, with K=10, for a motif from 1,000 to 1,009 inclusive, then the following 100 bp are from 1,010 to 1,109, inclusive. So an A to a G at 1,109 is within range while an A to a C at 1,110 is out of range.
* The number of AT bases within gcW bases, not including motif
* The number of callable bases within gcW bases, not including motif

We are currently not using the third value (the number of callable bases), only the total number of AT bases. The 2 * 2 test per motif is


|                     |AT to GC bases |Number of AT bases |
|:--------------------|:--------------|:------------------|
|Motif                |n1             |n2                 |
|Same CpG, GC Content |n3             |n4                 |
            
###Motif Filtering

Motifs were kept if the longest run of a specific nucleotide was less than or equal to mrle = 10. Motifs were also kept if the nucleotide diversity (ie number of A,C,G,T) bases was greater than or equal to ndge =0. For each test seperately, motifs were removed as well as individual results masked if the counts were too low. Motif level results were not calculated if the counts among all lineages for a motif was less than rgte = 50. Lineage specific results were also masked (ie not calculated) in each test seperately if the motif lineage value was less than cgte = 10 (ie the n1 entry in the contingency tables).

###Clustering

To be eligible for clustering, a motif had to be Bonferonni significant on its lineage, have an OR>1, and not have a lineage with an OR<1 which had a more significant p-value. For clustering, a motif would have to meet a p-value threshold (explained below), as well as meet the criteria that there not be a lineage with OR<1 with a more significant p-value.

Given a set of eligible motifs for a given lineage for a given test for a given repeat background, start with the motif with the most significant p-value. Add to the cluster all eligible motifs beating a certain p-value within a certain distance (defined below), keeping track of alignment. Continue recursively for each added motif until exhausted. Build a position weight matrix by adding together the influence of each motif in the cluster, using a weight of the odds ratio minus 1, and adding it to the bases in the motif. Replace bases in the PWM after the summing with an entry of 0 with 0.1, and divide by the column totals to have each entry scaled between 0 and 1. 

Let K be the motif length we are interested in. We define as acceptably close for clustering all motifs which align perfectly and are off by 1 base (sum of K*4), or that are off by 1 base in the alignment left or right (2 for left vs right) with any new base in the gap (4 bases) with any one of the remaining K-1 bases allowed to change as well (4). In total there are (K-1)*4*4*2+K*4 = 328 possible acceptably close motifs.

For the first iteration, take a p-value threshold of the number of motifs to be searched, or (K-1)*4*4*2+K*4 = 328. For each subsequent iteration, take a p-value threshold where the deminator is the number of tests already performed plus the number to be searched on this iteration. For example, if on the first iteration 3 close motifs were found to meet the iteration 1 p-value threshold, then on the second iteration the p-value threhsold would be 0.05 divided by the number of motifs searched on the first iteration, 328, plus the 3 * 328 to be searched on the second iteration.

For plotting, we summed the collapsed motif clusters counts by base and added 0.5 to each count. Then, for a motif cluster of length $m$, letting $j \in \{1,2,3,4\}$ be the four nucleotides and $i \in \{1,2,...,m\}$ be the position within the motif cluster, we set $H_i = \sum_{j=1}^4 f_{i,j} \log(f_{i,j})$, and define the height of each base as $h_{i,j}=H_{i} f_{i,j}$. Bases are then ordered from smallest to largest entropy and plotted. We used elements of the seqLogo R package to draw the PWM.

The math for drawing the PWM is taken from [here](http://en.wikipedia.org/wiki/Sequence_logo).


###AT to GC plots
To better visualize the localization of AT to GC changes surrounding a motif cluster, we plotted whether bases changed from AT to GC or vice versa, with respect to their distance from the motif. We first scanned through the chromosomes to find instances where motifs in the motif cluster were lost. To ensure we weren't oversampling SNP changes due to similar motifs, we limited ourselves to counting only a single motif loss instance among a run of motif losses each one within the length of the motif cluster in distance from each other. 

Next, taking care to get both the correct strand as well as the position of the motif within the cluster of motifs correct, we catalogued both the position and base composition of any changes within a neighbourhood of 1000 bases. By summing across all loss instances of motifs in the cluster, and normalizing to the local sequence context, we could plot any type of ancestral to derived base change. Smoothing was done over 21 bases, ie taking the value at the flanking bases and over the prior and aft 10 bases. 

These plots also feature a PWM for the forward and reverse forms of the motif, as well as a series of line plots which show the number of motifs and their p-values for the motifs in the cluster. The middle line is for the test under consideration, while p-values for the other two tests are highlighted above and below, with grey lines linking the same motif (motifs with undefiend p-values on the other two tests are omitted from the plots for those tests and are not linked). These are stratefied into those which are Bonferonni significant on their test to the right of the red line, those which are between the initial clustering p-value threshold and the Bonferonni thresold in the middle, and those which do not meet the initial clustering p-value threhsold on the left. Numbers of motifs falling into each category are given as well.

---


## Some summary numbers

Aligned Genome (Gbp)


|  Total| Pass QC| Fail QC| Pass QC Non Repeat|
|------:|-------:|-------:|------------------:|
| 2.4723|  1.6449|  0.8274|             1.1138|
Number of Derived Mutations down a specific lineage


|FAM        |AMS        |Spretus    |AM         |WSBEiJ     |CASTEiJ    |PWKPhJ     |
|:----------|:----------|:----------|:----------|:----------|:----------|:----------|
|17,935,113 | 8,439,998 |11,800,414 | 4,728,452 | 6,066,739 | 6,178,547 | 6,347,677 |
Branch length as percent of alignable genome


|   FAM|   AMS| Spretus|    AM| WSBEiJ| CASTEiJ| PWKPhJ|
|-----:|-----:|-------:|-----:|------:|-------:|------:|
| 1.090| 0.513|   0.717| 0.287|  0.369|   0.376|  0.386|
Branch length compared to ancestral of all lineages in SNPs


|PWKPhJ     |CASTEiJ    |WSBEiJ     |Spretus    |FAM        |
|:----------|:----------|:----------|:----------|:----------|
|19,516,127 |19,346,997 |19,235,189 |20,240,412 |17,935,113 |
Branch length compared to ancestral as percent of alignable genome


| PWKPhJ| CASTEiJ| WSBEiJ| Spretus|   FAM|
|------:|-------:|------:|-------:|-----:|
|  1.186|   1.176|  1.169|   1.230| 1.090|


---

## Number of significant motifs per lineage per test

Note that the following numbers are only for motifs that have been clustered

Number of motifs (clusters) per test and lineage


|       |FAM       |AMS      |Spretus  |AM       |PWKPhJ |CASTEiJ |CASTEiJ.PWKPhJ |WSBEiJ |WSBEiJ.PWKPhJ |WSBEiJ.CASTEiJ |
|:------|:---------|:--------|:--------|:--------|:------|:-------|:--------------|:------|:-------------|:--------------|
|at     |124 (5)   |8 (5)    |449 (12) |109 (8)  |0 (0)  |2 (2)   |0 (0)          |13 (5) |0 (0)         |0 (0)          |
|lin    |32 (7)    |82 (13)  |373 (18) |386 (9)  |2 (2)  |6 (3)   |0 (0)          |9 (2)  |0 (0)         |1 (1)          |
|shared |1633 (45) |431 (43) |923 (40) |548 (19) |26 (1) |34 (5)  |9 (3)          |27 (7) |2 (2)         |2 (2)          |
FAM


|       |nonRepeat |(CA)n  |(TG)n  |B1_Mus1 |B1_Mus2 |RSINE1 |B2_Mm2 |B1_Mm  |B3     |B2_Mm1t |ID_B1 |(TATG)n |(TCTA)n |AT_rich |Lx8   |URR1A |MTD   |(TAGA)n |URR1B |B4    |RLTR15 |B1_Mur4 |RMER1B |
|:------|:---------|:------|:------|:-------|:-------|:------|:------|:------|:------|:-------|:-----|:-------|:-------|:-------|:-----|:-----|:-----|:-------|:-----|:-----|:------|:-------|:------|
|at     |124 (5)   |0 (0)  |0 (0)  |0 (0)   |0 (0)   |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0)   |0 (0) |0 (0)   |0 (0)   |0 (0)   |0 (0) |0 (0) |0 (0) |0 (0)   |0 (0) |0 (0) |0 (0)  |0 (0)   |0 (0)  |
|lin    |24 (4)    |0 (0)  |0 (0)  |0 (0)   |0 (0)   |4 (1)  |0 (0)  |0 (0)  |0 (0)  |0 (0)   |1 (1) |0 (0)   |0 (0)   |0 (0)   |3 (1) |0 (0) |0 (0) |0 (0)   |0 (0) |0 (0) |0 (0)  |0 (0)   |0 (0)  |
|shared |1216 (4)  |88 (2) |81 (1) |61 (5)  |39 (1)  |25 (2) |24 (7) |17 (4) |14 (1) |8 (4)   |7 (2) |8 (1)   |8 (1)   |7 (1)   |4 (1) |6 (1) |5 (1) |5 (1)   |4 (1) |2 (1) |2 (1)  |1 (1)   |1 (1)  |
AMS


|       |nonRepeat |RSINE1 |B1_Mus1 |B1_Mus2 |B1_Mm  |B3    |B2_Mm2 |B4A   |ORR1C2 |B4    |RLTR23 |AT_rich |B3A   |(TAGA)n |(CA)n |(TG)n |ID_B1 |RMER19C |(TCTA)n |
|:------|:---------|:------|:-------|:-------|:------|:-----|:------|:-----|:------|:-----|:------|:-------|:-----|:-------|:-----|:-----|:-----|:-------|:-------|
|at     |4 (3)     |1 (1)  |0 (0)   |0 (0)   |0 (0)  |0 (0) |0 (0)  |3 (1) |0 (0)  |0 (0) |0 (0)  |0 (0)   |0 (0) |0 (0)   |0 (0) |0 (0) |0 (0) |0 (0)   |0 (0)   |
|lin    |53 (7)    |17 (1) |1 (1)   |0 (0)   |0 (0)  |5 (1) |0 (0)  |0 (0) |3 (1)  |1 (1) |2 (1)  |0 (0)   |0 (0) |0 (0)   |0 (0) |0 (0) |0 (0) |0 (0)   |0 (0)   |
|shared |253 (6)   |45 (1) |30 (1)  |30 (5)  |22 (5) |6 (1) |10 (6) |6 (3) |5 (1)  |5 (1) |3 (1)  |3 (1)   |3 (2) |3 (2)   |2 (2) |2 (2) |1 (1) |1 (1)   |1 (1)   |
Spretus


|       |nonRepeat |B3A    |Lx3C   |B4A    |RMER17C |AT_rich |B3     |(TG)n  |RSINE1 |URR1A |(CA)n |B2_Mm2 |URR1B |Lx9   |B1_Mus1 |B1_Mus2 |Lx2B  |(TAGA)n |B1_Mm |B4    |ID_B1 |MTEa  |
|:------|:---------|:------|:------|:------|:-------|:-------|:------|:------|:------|:-----|:-----|:------|:-----|:-----|:-------|:-------|:-----|:-------|:-----|:-----|:-----|:-----|
|at     |407 (4)   |19 (1) |0 (0)  |12 (1) |0 (0)   |0 (0)   |2 (1)  |0 (0)  |1 (1)  |0 (0) |0 (0) |0 (0)  |1 (1) |4 (1) |0 (0)   |0 (0)   |0 (0) |2 (1)   |0 (0) |0 (0) |1 (1) |0 (0) |
|lin    |332 (7)   |10 (1) |11 (1) |2 (2)  |7 (1)   |0 (0)   |0 (0)  |2 (1)  |4 (1)  |1 (1) |2 (1) |0 (0)  |0 (0) |1 (1) |0 (0)   |0 (0)   |1 (1) |0 (0)   |0 (0) |0 (0) |0 (0) |0 (0) |
|shared |788 (6)   |29 (2) |12 (1) |4 (3)  |9 (1)   |15 (1)  |12 (1) |11 (5) |5 (1)  |8 (1) |6 (5) |7 (4)  |6 (1) |1 (1) |4 (2)   |2 (1)   |1 (1) |0 (0)   |1 (1) |1 (1) |0 (0) |1 (1) |
AM


|       |nonRepeat |ID_B1  |RSINE1 |B3    |B1_Mm |B1_Mus2 |B4A   |AT_rich |(TCTA)n |ID4   |
|:------|:---------|:------|:------|:-----|:-----|:-------|:-----|:-------|:-------|:-----|
|at     |103 (6)   |0 (0)  |0 (0)  |5 (1) |0 (0) |0 (0)   |0 (0) |0 (0)   |0 (0)   |1 (1) |
|lin    |366 (4)   |7 (2)  |7 (1)  |5 (1) |0 (0) |0 (0)   |1 (1) |0 (0)   |0 (0)   |0 (0) |
|shared |489 (3)   |17 (2) |14 (2) |8 (2) |6 (3) |6 (3)   |3 (2) |3 (1)   |2 (1)   |0 (0) |
PWKPhJ


|       |nonRepeat |
|:------|:---------|
|lin    |2 (2)     |
|shared |26 (1)    |
CASTEiJ


|       |nonRepeat |AT_rich |MTD   |
|:------|:---------|:-------|:-----|
|at     |2 (2)     |0 (0)   |0 (0) |
|lin    |5 (2)     |0 (0)   |1 (1) |
|shared |29 (3)    |4 (1)   |1 (1) |
CASTEiJ.PWKPhJ


|       |AT_rich |nonRepeat |
|:------|:-------|:---------|
|shared |1 (1)   |8 (2)     |
WSBEiJ


|       |nonRepeat |(TG)n |
|:------|:---------|:-----|
|at     |13 (5)    |0 (0) |
|lin    |9 (2)     |0 (0) |
|shared |26 (6)    |1 (1) |
WSBEiJ.PWKPhJ


|       |nonRepeat |
|:------|:---------|
|shared |2 (2)     |
WSBEiJ.CASTEiJ


|       |RSINE1 |nonRepeat |
|:------|:------|:---------|
|lin    |1 (1)  |0 (0)     |
|shared |1 (1)  |1 (1)     |

---

## Features of significant motifs

Test: lin 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.6| 3.2| 10.0| 20.1| 25.6| 21.6| 12.4| 4.9| 1.3| 0.3|
|significant     | 0.3| 1.2| 1.2|  3.3| 11.8| 27.9| 27.8| 13.4| 3.5| 6.1| 3.4|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 69.3| 28.6| 2.1| 0.0| 0.0|
|significant     | 99.8|  0.2| 0.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 7.2| 54.7| 28.3| 7.6| 1.8| 0.3| 0.1| 0.0| 0.0| 0.0|
|significant     | 8.1| 59.8| 15.3| 1.7| 1.5| 2.1| 5.9| 4.2| 1.1| 0.2|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.9| 23.4| 74.7|
|significant     | 0.2| 9.2| 33.4| 57.1|
Test: shared 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|    8|    9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|----:|----:|---:|
|not significant | 0.0| 0.6| 3.2| 10.1| 20.2| 25.6| 21.6| 12.4|  4.8|  1.2| 0.3|
|significant     | 0.2| 1.2| 2.0|  3.8|  7.6| 16.3| 19.3| 19.6| 16.7| 10.1| 3.3|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 69.1| 28.8| 2.1| 0.0| 0.0|
|significant     | 97.5|  2.2| 0.3| 0.0| 0.0|
Maximum run length 


|                |    1|    2|    3|    4|   5|   6|   7|   8|   9|  10|
|:---------------|----:|----:|----:|----:|---:|---:|---:|---:|---:|---:|
|not significant |  7.2| 54.8| 28.4|  7.6| 1.7| 0.3| 0.0| 0.0| 0.0| 0.0|
|significant     | 10.5| 37.0| 25.1| 13.8| 5.0| 3.8| 3.6| 0.9| 0.2| 0.0|
Nucleotide diversity 


|                |   1|    2|    3|    4|
|:---------------|---:|----:|----:|----:|
|not significant | 0.0|  1.8| 23.2| 75.0|
|significant     | 0.0| 12.2| 45.7| 42.1|
Test: at 
AT content 


|                |   0|   1|   2|   3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|---:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.3| 2.5| 9.2| 19.4| 25.8| 22.8| 13.3| 5.1| 1.3| 0.3|
|significant     | 0.0| 0.0| 0.4| 2.8|  8.7| 26.4| 30.7| 21.7| 5.9| 1.9| 1.6|
Number of CpGs 


|                |    0|    1|   2|   3|
|:---------------|----:|----:|---:|---:|
|not significant | 67.9| 32.0| 0.1| 0.0|
|significant     | 99.5|  0.5| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 7.2| 55.2| 28.2| 7.4| 1.6| 0.3| 0.1| 0.0| 0.0| 0.0|
|significant     | 4.7| 59.4| 26.8| 2.5| 1.3| 1.5| 1.7| 1.3| 0.5| 0.1|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.3| 22.3| 76.5|
|significant     | 0.1| 6.0| 36.7| 57.2|
---


## PWK results at CAST top results

Other studies have found that M. m. musculus has a PRDM9 allele similar to the CAST motif, but while we found a denovo CAST motif similar to published work, here we found no de novo PWK motifs. Here, we look at the p-values for the motifs in the top CAST cluster in the other strains

Test: lin


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|
|CCAAGTCTGC |0.63     |1.06      |0.28     |0.86      |4.6e-06   |1.62       |
|CAGACTTGGC |0.14     |1.2       |0.53     |0.91      |1.3e-05   |1.63       |
|AGACTTGGAT |0.57     |1.07      |0.21     |1.15      |4e-08     |1.8        |
|AGACTTGGCT |0.048    |1.23      |0.038    |0.77      |8.6e-07   |1.63       |
|AGACTTTGCT |0.78     |1.03      |0.91     |1.01      |3.2e-10   |1.82       |
|AATCCAAGTC |0.68     |0.93      |0.89     |0.97      |2.4e-15   |2.38       |
|AAGCCAAGTC |0.038    |1.25      |0.73     |0.95      |4.6e-11   |1.89       |
|AATCCCAGTC |0.78     |1.03      |0.18     |0.81      |5.5e-08   |1.87       |
|AAGCAAAGTC |0.38     |1.1       |0.51     |0.92      |1.1e-11   |1.87       |
|AAGCCCAGTC |0.13     |1.2       |1        |1         |1.9e-08   |1.83       |
|AAATCCAAGT |0.66     |0.94      |0.3      |1.11      |2.6e-05   |1.51       |
|ACTTGGATTC |0.78     |0.95      |0.49     |0.89      |3.9e-05   |1.65       |
Test: shared


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|
|CCAAGTCTGC |0.3      |1.13      |0.63     |0.93      |6.5e-07   |1.67       |
|CAGACTTGGC |0.22     |1.15      |0.43     |0.9       |6.5e-05   |1.53       |
|AGACTTGGAT |0.25     |1.15      |0.067    |1.23      |3.1e-09   |1.85       |
|AGACTTGGCT |0.035    |1.24      |0.068    |0.8       |1e-06     |1.6        |
|AGACTTTGCT |0.78     |1.03      |0.91     |1.01      |2.2e-09   |1.73       |
|AATCCAAGTC |1        |0.99      |0.83     |1.03      |1.1e-15   |2.33       |
|AAGCCAAGTC |0.009    |1.31      |0.86     |1.02      |5.9e-12   |1.91       |
|AATCCCAGTC |0.034    |1.33      |0.65     |1.06      |1.2e-12   |2.28       |
|AAGCAAAGTC |0.21     |1.14      |0.83     |0.97      |4.6e-12   |1.86       |
|AAGCCCAGTC |0.0029   |1.42      |0.14     |1.2       |5.6e-12   |2.08       |
|AAATCCAAGT |0.46     |0.91      |0.53     |1.07      |0.00026   |1.42       |
|ACTTGGATTC |0.89     |1.02      |0.83     |0.96      |9.6e-06   |1.7        |
Test: at


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|
|CCAAGTCTGC |0.24     |1.29      |NA       |NA        |0.013     |1.57       |
|CAGACTTGGC |0.11     |1.43      |0.89     |1         |0.26      |1.26       |
|AGACTTGGAT |0.8      |0.9       |0.33     |1.22      |0.1       |1.34       |
|AGACTTGGCT |0.021    |1.54      |0.89     |0.93      |6.3e-05   |1.86       |
|AGACTTTGCT |1        |0.99      |1        |0.97      |9e-08     |2.1        |
|AATCCAAGTC |0.00093  |2.17      |0.89     |0.91      |0.0041    |1.62       |
|AAGCCAAGTC |0.0028   |1.72      |1        |1         |1.9e-09   |2.29       |
|AATCCCAGTC |0.015    |1.75      |0.34     |1.28      |1e-04     |1.96       |
|AAGCAAAGTC |0.085    |1.38      |0.16     |0.65      |3.9e-07   |1.97       |
|AAGCCCAGTC |0.8      |1.06      |1        |0.96      |0.019     |1.53       |
|AAATCCAAGT |0.00077  |1.85      |0.062    |1.4       |0.58      |0.88       |
|ACTTGGATTC |0.05     |1.61      |0.46     |1.23      |0.068     |1.44       |
            used   (Mb) gc trigger (Mb)  max used   (Mb)
Ncells   5275307  281.8  2.040e+07 1090   5275307  281.8
Vcells 446566771 3407.1  1.061e+09 8095 446566771 3407.1
---



## Giant AT to GC plot

All clustered AT to GC plots collapsed together 

![plot of chunk atToGCAll](figure/atToGCAll.png) 
---


## QQ plots - seperate

![plot of chunk pValuesQQPlots](figure/pValuesQQPlots1.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots2.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots3.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots4.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots5.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots6.png) 
---

## Compare p-values between methods

![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods1.png) ![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods2.png) ![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods3.png) 
---


## Compare p-values between lineages within a method

![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage1.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage2.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage3.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage4.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage5.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage6.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage7.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage8.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage9.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage10.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage11.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage12.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage13.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage14.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage15.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage16.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage17.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage18.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage19.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage20.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage21.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage22.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage23.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage24.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage25.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage26.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage27.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage28.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage29.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage30.png) 
---




## All significant motifs


```
## Lineage, FAM, ID_B1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters1.png) 

```
## Lineage, FAM, Lx8, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters2.png) 

```
## Lineage, FAM, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters3.png) 

```
## Lineage, FAM, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters4.png) 

```
## Lineage, FAM, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters5.png) 

```
## Lineage, FAM, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters6.png) 

```
## Lineage, FAM, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters7.png) 

```
## Lineage, AMS, B1_Mus1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters8.png) 

```
## Lineage, AMS, B3, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters9.png) 

```
## Lineage, AMS, B4, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters10.png) 

```
## Lineage, AMS, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters11.png) 

```
## Lineage, AMS, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters12.png) 

```
## Lineage, AMS, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters13.png) 

```
## Lineage, AMS, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters14.png) 

```
## Lineage, AMS, nonRepeat, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters15.png) 

```
## Lineage, AMS, nonRepeat, motifNumber = 6
```

![plot of chunk motifClusters](figure/motifClusters16.png) 

```
## Lineage, AMS, nonRepeat, motifNumber = 7
```

![plot of chunk motifClusters](figure/motifClusters17.png) 

```
## Lineage, AMS, ORR1C2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters18.png) 

```
## Lineage, AMS, RLTR23, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters19.png) 

```
## Lineage, AMS, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters20.png) 

```
## Lineage, Spretus, B3A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters21.png) 

```
## Lineage, Spretus, B4A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters22.png) 

```
## Lineage, Spretus, B4A, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters23.png) 

```
## Lineage, Spretus, (CA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters24.png) 

```
## Lineage, Spretus, Lx2B, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters25.png) 

```
## Lineage, Spretus, Lx3C, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters26.png) 

```
## Lineage, Spretus, Lx9, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters27.png) 

```
## Lineage, Spretus, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters28.png) 

```
## Lineage, Spretus, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters29.png) 

```
## Lineage, Spretus, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters30.png) 

```
## Lineage, Spretus, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters31.png) 

```
## Lineage, Spretus, nonRepeat, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters32.png) 

```
## Lineage, Spretus, nonRepeat, motifNumber = 6
```

![plot of chunk motifClusters](figure/motifClusters33.png) 

```
## Lineage, Spretus, nonRepeat, motifNumber = 7
```

![plot of chunk motifClusters](figure/motifClusters34.png) 

```
## Lineage, Spretus, RMER17C, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters35.png) 

```
## Lineage, Spretus, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters36.png) 

```
## Lineage, Spretus, (TG)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters37.png) 

```
## Lineage, Spretus, URR1A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters38.png) 

```
## Lineage, AM, B3, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters39.png) 

```
## Lineage, AM, B4A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters40.png) 

```
## Lineage, AM, ID_B1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters41.png) 

```
## Lineage, AM, ID_B1, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters42.png) 

```
## Lineage, AM, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters43.png) 

```
## Lineage, AM, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters44.png) 

```
## Lineage, AM, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters45.png) 

```
## Lineage, AM, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters46.png) 

```
## Lineage, AM, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters47.png) 

```
## Lineage, PWKPhJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters48.png) 

```
## Lineage, PWKPhJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters49.png) 

```
## Lineage, CASTEiJ, MTD, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters50.png) 

```
## Lineage, CASTEiJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters51.png) 

```
## Lineage, CASTEiJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters52.png) 

```
## Lineage, WSBEiJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters53.png) 

```
## Lineage, WSBEiJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters54.png) 

```
## Lineage, WSBEiJ.CASTEiJ, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters55.png) 

```
## Shared, FAM, AT_rich, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters56.png) 

```
## Shared, FAM, B1_Mm, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters57.png) 

```
## Shared, FAM, B1_Mm, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters58.png) 

```
## Shared, FAM, B1_Mm, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters59.png) 

```
## Shared, FAM, B1_Mm, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters60.png) 

```
## Shared, FAM, B1_Mur4, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters61.png) 

```
## Shared, FAM, B1_Mus1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters62.png) 

```
## Shared, FAM, B1_Mus1, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters63.png) 

```
## Shared, FAM, B1_Mus1, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters64.png) 

```
## Shared, FAM, B1_Mus1, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters65.png) 

```
## Shared, FAM, B1_Mus1, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters66.png) 

```
## Shared, FAM, B1_Mus2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters67.png) 

```
## Shared, FAM, B2_Mm1t, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters68.png) 

```
## Shared, FAM, B2_Mm1t, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters69.png) 

```
## Shared, FAM, B2_Mm1t, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters70.png) 

```
## Shared, FAM, B2_Mm1t, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters71.png) 

```
## Shared, FAM, B2_Mm2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters72.png) 

```
## Shared, FAM, B2_Mm2, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters73.png) 

```
## Shared, FAM, B2_Mm2, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters74.png) 

```
## Shared, FAM, B2_Mm2, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters75.png) 

```
## Shared, FAM, B2_Mm2, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters76.png) 

```
## Shared, FAM, B2_Mm2, motifNumber = 6
```

![plot of chunk motifClusters](figure/motifClusters77.png) 

```
## Shared, FAM, B2_Mm2, motifNumber = 7
```

![plot of chunk motifClusters](figure/motifClusters78.png) 

```
## Shared, FAM, B3, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters79.png) 

```
## Shared, FAM, B4, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters80.png) 

```
## Shared, FAM, (CA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters81.png) 

```
## Shared, FAM, (CA)n, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters82.png) 

```
## Shared, FAM, ID_B1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters83.png) 

```
## Shared, FAM, ID_B1, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters84.png) 

```
## Shared, FAM, Lx8, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters85.png) 

```
## Shared, FAM, MTD, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters86.png) 

```
## Shared, FAM, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters87.png) 

```
## Shared, FAM, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters88.png) 

```
## Shared, FAM, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters89.png) 

```
## Shared, FAM, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters90.png) 

```
## Shared, FAM, RLTR15, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters91.png) 

```
## Shared, FAM, RMER1B, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters92.png) 

```
## Shared, FAM, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters93.png) 

```
## Shared, FAM, RSINE1, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters94.png) 

```
## Shared, FAM, (TAGA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters95.png) 

```
## Shared, FAM, (TATG)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters96.png) 

```
## Shared, FAM, (TCTA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters97.png) 

```
## Shared, FAM, (TG)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters98.png) 

```
## Shared, FAM, URR1A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters99.png) 

```
## Shared, FAM, URR1B, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters100.png) 

```
## Shared, AMS, AT_rich, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters101.png) 

```
## Shared, AMS, B1_Mm, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters102.png) 

```
## Shared, AMS, B1_Mm, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters103.png) 

```
## Shared, AMS, B1_Mm, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters104.png) 

```
## Shared, AMS, B1_Mm, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters105.png) 

```
## Shared, AMS, B1_Mm, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters106.png) 

```
## Shared, AMS, B1_Mus1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters107.png) 

```
## Shared, AMS, B1_Mus2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters108.png) 

```
## Shared, AMS, B1_Mus2, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters109.png) 

```
## Shared, AMS, B1_Mus2, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters110.png) 

```
## Shared, AMS, B1_Mus2, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters111.png) 

```
## Shared, AMS, B1_Mus2, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters112.png) 

```
## Shared, AMS, B2_Mm2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters113.png) 

```
## Shared, AMS, B2_Mm2, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters114.png) 

```
## Shared, AMS, B2_Mm2, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters115.png) 

```
## Shared, AMS, B2_Mm2, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters116.png) 

```
## Shared, AMS, B2_Mm2, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters117.png) 

```
## Shared, AMS, B2_Mm2, motifNumber = 6
```

![plot of chunk motifClusters](figure/motifClusters118.png) 

```
## Shared, AMS, B3, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters119.png) 

```
## Shared, AMS, B3A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters120.png) 

```
## Shared, AMS, B3A, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters121.png) 

```
## Shared, AMS, B4, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters122.png) 

```
## Shared, AMS, B4A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters123.png) 

```
## Shared, AMS, B4A, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters124.png) 

```
## Shared, AMS, B4A, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters125.png) 

```
## Shared, AMS, (CA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters126.png) 

```
## Shared, AMS, (CA)n, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters127.png) 

```
## Shared, AMS, ID_B1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters128.png) 

```
## Shared, AMS, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters129.png) 

```
## Shared, AMS, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters130.png) 

```
## Shared, AMS, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters131.png) 

```
## Shared, AMS, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters132.png) 

```
## Shared, AMS, nonRepeat, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters133.png) 

```
## Shared, AMS, nonRepeat, motifNumber = 6
```

![plot of chunk motifClusters](figure/motifClusters134.png) 

```
## Shared, AMS, ORR1C2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters135.png) 

```
## Shared, AMS, RLTR23, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters136.png) 

```
## Shared, AMS, RMER19C, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters137.png) 

```
## Shared, AMS, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters138.png) 

```
## Shared, AMS, (TAGA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters139.png) 

```
## Shared, AMS, (TAGA)n, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters140.png) 

```
## Shared, AMS, (TCTA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters141.png) 

```
## Shared, AMS, (TG)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters142.png) 

```
## Shared, AMS, (TG)n, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters143.png) 

```
## Shared, Spretus, AT_rich, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters144.png) 

```
## Shared, Spretus, B1_Mm, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters145.png) 

```
## Shared, Spretus, B1_Mus1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters146.png) 

```
## Shared, Spretus, B1_Mus1, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters147.png) 

```
## Shared, Spretus, B1_Mus2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters148.png) 

```
## Shared, Spretus, B2_Mm2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters149.png) 

```
## Shared, Spretus, B2_Mm2, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters150.png) 

```
## Shared, Spretus, B2_Mm2, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters151.png) 

```
## Shared, Spretus, B2_Mm2, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters152.png) 

```
## Shared, Spretus, B3, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters153.png) 

```
## Shared, Spretus, B3A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters154.png) 

```
## Shared, Spretus, B3A, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters155.png) 

```
## Shared, Spretus, B4, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters156.png) 

```
## Shared, Spretus, B4A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters157.png) 

```
## Shared, Spretus, B4A, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters158.png) 

```
## Shared, Spretus, B4A, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters159.png) 

```
## Shared, Spretus, (CA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters160.png) 

```
## Shared, Spretus, (CA)n, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters161.png) 

```
## Shared, Spretus, (CA)n, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters162.png) 

```
## Shared, Spretus, (CA)n, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters163.png) 

```
## Shared, Spretus, (CA)n, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters164.png) 

```
## Shared, Spretus, Lx2B, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters165.png) 

```
## Shared, Spretus, Lx3C, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters166.png) 

```
## Shared, Spretus, Lx9, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters167.png) 

```
## Shared, Spretus, MTEa, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters168.png) 

```
## Shared, Spretus, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters169.png) 

```
## Shared, Spretus, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters170.png) 

```
## Shared, Spretus, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters171.png) 

```
## Shared, Spretus, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters172.png) 

```
## Shared, Spretus, nonRepeat, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters173.png) 

```
## Shared, Spretus, nonRepeat, motifNumber = 6
```

![plot of chunk motifClusters](figure/motifClusters174.png) 

```
## Shared, Spretus, RMER17C, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters175.png) 

```
## Shared, Spretus, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters176.png) 

```
## Shared, Spretus, (TG)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters177.png) 

```
## Shared, Spretus, (TG)n, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters178.png) 

```
## Shared, Spretus, (TG)n, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters179.png) 

```
## Shared, Spretus, (TG)n, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters180.png) 

```
## Shared, Spretus, (TG)n, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters181.png) 

```
## Shared, Spretus, URR1A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters182.png) 

```
## Shared, Spretus, URR1B, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters183.png) 

```
## Shared, AM, AT_rich, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters184.png) 

```
## Shared, AM, B1_Mm, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters185.png) 

```
## Shared, AM, B1_Mm, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters186.png) 

```
## Shared, AM, B1_Mm, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters187.png) 

```
## Shared, AM, B1_Mus2, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters188.png) 

```
## Shared, AM, B1_Mus2, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters189.png) 

```
## Shared, AM, B1_Mus2, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters190.png) 

```
## Shared, AM, B3, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters191.png) 

```
## Shared, AM, B3, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters192.png) 

```
## Shared, AM, B4A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters193.png) 

```
## Shared, AM, B4A, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters194.png) 

```
## Shared, AM, ID_B1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters195.png) 

```
## Shared, AM, ID_B1, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters196.png) 

```
## Shared, AM, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters197.png) 

```
## Shared, AM, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters198.png) 

```
## Shared, AM, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters199.png) 

```
## Shared, AM, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters200.png) 

```
## Shared, AM, RSINE1, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters201.png) 

```
## Shared, AM, (TCTA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters202.png) 

```
## Shared, PWKPhJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters203.png) 

```
## Shared, CASTEiJ, AT_rich, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters204.png) 

```
## Shared, CASTEiJ, MTD, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters205.png) 

```
## Shared, CASTEiJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters206.png) 

```
## Shared, CASTEiJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters207.png) 

```
## Shared, CASTEiJ, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters208.png) 

```
## Shared, CASTEiJ.PWKPhJ, AT_rich, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters209.png) 

```
## Shared, CASTEiJ.PWKPhJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters210.png) 

```
## Shared, CASTEiJ.PWKPhJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters211.png) 

```
## Shared, WSBEiJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters212.png) 

```
## Shared, WSBEiJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters213.png) 

```
## Shared, WSBEiJ, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters214.png) 

```
## Shared, WSBEiJ, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters215.png) 

```
## Shared, WSBEiJ, nonRepeat, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters216.png) 

```
## Shared, WSBEiJ, nonRepeat, motifNumber = 6
```

![plot of chunk motifClusters](figure/motifClusters217.png) 

```
## Shared, WSBEiJ, (TG)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters218.png) 

```
## Shared, WSBEiJ.PWKPhJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters219.png) 

```
## Shared, WSBEiJ.PWKPhJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters220.png) 

```
## Shared, WSBEiJ.CASTEiJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters221.png) 

```
## Shared, WSBEiJ.CASTEiJ, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters222.png) 

```
## AT to GC, FAM, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters223.png) 

```
## AT to GC, FAM, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters224.png) 

```
## AT to GC, FAM, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters225.png) 

```
## AT to GC, FAM, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters226.png) 

```
## AT to GC, FAM, nonRepeat, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters227.png) 

```
## AT to GC, AMS, B4A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters228.png) 

```
## AT to GC, AMS, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters229.png) 

```
## AT to GC, AMS, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters230.png) 

```
## AT to GC, AMS, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters231.png) 

```
## AT to GC, AMS, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters232.png) 

```
## AT to GC, Spretus, B3, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters233.png) 

```
## AT to GC, Spretus, B3A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters234.png) 

```
## AT to GC, Spretus, B4A, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters235.png) 

```
## AT to GC, Spretus, ID_B1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters236.png) 

```
## AT to GC, Spretus, Lx9, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters237.png) 

```
## AT to GC, Spretus, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters238.png) 

```
## AT to GC, Spretus, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters239.png) 

```
## AT to GC, Spretus, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters240.png) 

```
## AT to GC, Spretus, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters241.png) 

```
## AT to GC, Spretus, RSINE1, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters242.png) 

```
## AT to GC, Spretus, (TAGA)n, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters243.png) 

```
## AT to GC, Spretus, URR1B, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters244.png) 

```
## AT to GC, AM, B3, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters245.png) 

```
## AT to GC, AM, ID4, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters246.png) 

```
## AT to GC, AM, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters247.png) 

```
## AT to GC, AM, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters248.png) 

```
## AT to GC, AM, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters249.png) 

```
## AT to GC, AM, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters250.png) 

```
## AT to GC, AM, nonRepeat, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters251.png) 

```
## AT to GC, AM, nonRepeat, motifNumber = 6
```

![plot of chunk motifClusters](figure/motifClusters252.png) 

```
## AT to GC, CASTEiJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters253.png) 

```
## AT to GC, CASTEiJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters254.png) 

```
## AT to GC, WSBEiJ, nonRepeat, motifNumber = 1
```

![plot of chunk motifClusters](figure/motifClusters255.png) 

```
## AT to GC, WSBEiJ, nonRepeat, motifNumber = 2
```

![plot of chunk motifClusters](figure/motifClusters256.png) 

```
## AT to GC, WSBEiJ, nonRepeat, motifNumber = 3
```

![plot of chunk motifClusters](figure/motifClusters257.png) 

```
## AT to GC, WSBEiJ, nonRepeat, motifNumber = 4
```

![plot of chunk motifClusters](figure/motifClusters258.png) 

```
## AT to GC, WSBEiJ, nonRepeat, motifNumber = 5
```

![plot of chunk motifClusters](figure/motifClusters259.png) 


---

