#Hotspot death
##Species: mice
##Run version: 2014_11_07
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

|Parameter             |Value                                                 |Description                                                              |
|:---------------------|:-----------------------------------------------------|:------------------------------------------------------------------------|
|mouseFilter           |ArrayNotFilteredAnnot295.0                            |What folder used for mice                                                |
|mrle                  |10                                                    |Maximum run length less than or equal to                                 |
|ndge                  |0                                                     |Nuclotide diversity greater than or equal to                             |
|pwmMinimum            |0.1                                                   |When a PWM has 0 entry, what to reset the 0 values to                    |
|Klist                 |10                                                    |Range of K to use                                                        |
|gcW1                  |100                                                   |Range to test for GC increase                                            |
|gcW2                  |1000                                                  |Range to plot GC increases                                               |
|gcW3                  |10                                                    |Smoothing window for AT to GC plots                                      |
|ctge                  |10                                                    |For a p-value to be generated, each cell must have at least this number  |
|rgte                  |50                                                    |For a row to count, there must be this many entries                      |
|testingNames          |losslin,lossat,gainlin,gainat                         |(Short) names of the tests we will perform                               |
|plottingNames         |Loss Lineage,Loss AT to GC,Gain Lineage,Gain AT to GC |Plotting names of the tests we will perform                              |
|nTests                |4                                                     |Number of tests to be performed                                          |
|pThresh               |0.000152439024390244                                  |Initial threshold for p-value clstering                                  |
|mouseMatrixDefinition |narrow                                                |How we define mice lineages                                              |
|nR1                   |12                                                    |If there are nR1 SNPs in nD1 bp, remove all SNPs                         |
|nD1                   |50                                                    |See above                                                                |
|nR2                   |7                                                     |For a lineage, if there are ge this many SNPs in nD2 bp, remove all SNPs |
|nD2                   |50                                                    |See above                                                                |
|removeNum             |0                                                     |How many samples are allowed to be uncallable                            |
|contingencyComparison |Same CpG, GC Content                                  |To what do we compare motifs                                             |

---


##Methods
###SNP Filtering

SNPs are filtered if they have any missingness among the lineages, do not agree with the species tree, are multi-allelic, or are heterozygous among homozygous animals. 

Subsequently, SNPs are removed if they are too close together. This is the last step in the procedure, ie after removing SNPs for reasons listed above. I removed any SNPs if there were nR1 SNPs within nD1 bases, and, following this, I removed lineage specific SNPs down any lineage if there were nR2 SNPs within nD2 bases. For example, if nR1 was 10 and nD1 was 50, and there was a cluster of 14 SNPs within 50 bases, all of the offending SNPs were removed. 

To get a sense of how many SNPs this removed for given parameter settings, I checked how many SNPs were filtered for a range of parameter settings for the smallest chromosome. I also compared this against expectations. To get an expectation, I simulated a pseudo-chromosome of results. I calculated the expected branch lengths of each lineage given the emprical data to this point, ie lineage 1 has 1 percent divergence against MRCA of the set of all lineages, lineage 2 has 0.5 percent divergence against MRCA, etc. Then, I decided whether each base was mutated according to the total branch length of the tree, and then, given a mutation, what branch it occured on with probabilities equal to each lineages share of the total tree length. 



```
## Warning: cannot open compressed file
## '/Net/dense/data/wildmice/motifLoss/input_A_2014_11_07/forFilteringPlot.RData',
## probable reason 'No such file or directory'
```

```
## Error: cannot open the connection
```

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


|        |FAM     |AMS     |Spretus  |AM      |PWKPhJ |CASTEiJ |WSBEiJ |WSBEiJ.CASTEiJ |
|:-------|:-------|:-------|:--------|:-------|:------|:-------|:------|:--------------|
|gainat  |44 (11) |0 (0)   |47 (9)   |11 (7)  |5 (4)  |1 (1)   |1 (1)  |0 (0)          |
|gainlin |45 (11) |46 (14) |30 (7)   |11 (3)  |1 (1)  |0 (0)   |0 (0)  |0 (0)          |
|lossat  |120 (7) |8 (5)   |443 (12) |106 (8) |0 (0)  |2 (2)   |13 (5) |0 (0)          |
|losslin |32 (7)  |83 (14) |368 (17) |383 (9) |2 (2)  |6 (3)   |9 (2)  |1 (1)          |
FAM


|        |nonRepeat |B1_Mus1 |RSINE1 |B2_Mm2 |ID_B1 |Lx8   |B1_Mm |(CA)n |RMER6A |(TG)n |
|:-------|:---------|:-------|:------|:------|:-----|:-----|:-----|:-----|:------|:-----|
|gainat  |27 (7)    |9 (1)   |0 (0)  |5 (2)  |3 (1) |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |
|gainlin |28 (5)    |3 (1)   |6 (1)  |0 (0)  |0 (0) |1 (1) |3 (1) |2 (1) |0 (0)  |2 (1) |
|lossat  |118 (5)   |0 (0)   |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |2 (2)  |0 (0) |
|losslin |24 (4)    |0 (0)   |4 (1)  |0 (0)  |1 (1) |3 (1) |0 (0) |0 (0) |0 (0)  |0 (0) |
AMS


|        |nonRepeat |B2_Mm2 |RSINE1 |B1_Mus1 |B3    |MTC   |B4A   |ORR1C2 |RLTR23 |(TC)n |B1_Mus2 |B4    |Lx5   |(TAGA)n |
|:-------|:---------|:------|:------|:-------|:-----|:-----|:-----|:------|:------|:-----|:-------|:-----|:-----|:-------|
|gainlin |0 (0)     |29 (4) |6 (1)  |6 (5)   |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0)  |2 (1) |1 (1)   |0 (0) |1 (1) |1 (1)   |
|lossat  |4 (3)     |0 (0)  |1 (1)  |0 (0)   |0 (0) |0 (0) |3 (1) |0 (0)  |0 (0)  |0 (0) |0 (0)   |0 (0) |0 (0) |0 (0)   |
|losslin |51 (7)    |0 (0)  |17 (1) |1 (1)   |4 (1) |4 (1) |0 (0) |3 (1)  |2 (1)  |0 (0) |0 (0)   |1 (1) |0 (0) |0 (0)   |
Spretus


|        |nonRepeat |B3A    |B4A    |Lx3C   |RMER17C |B2_Mm2 |B3    |Lx9   |RSINE1 |B1_Mus1 |(CA)n |(TAGA)n |(TCTA)n |(TG)n |B1_Mus2 |ID_B1 |Lx2B  |URR1A |URR1B |
|:-------|:---------|:------|:------|:------|:-------|:------|:-----|:-----|:------|:-------|:-----|:-------|:-------|:-----|:-------|:-----|:-----|:-----|:-----|
|gainat  |43 (5)    |0 (0)  |0 (0)  |0 (0)  |0 (0)   |1 (1)  |2 (2) |0 (0) |0 (0)  |0 (0)   |0 (0) |0 (0)   |0 (0)   |0 (0) |1 (1)   |0 (0) |0 (0) |0 (0) |0 (0) |
|gainlin |19 (3)    |0 (0)  |0 (0)  |0 (0)  |0 (0)   |4 (1)  |1 (1) |0 (0) |0 (0)  |4 (1)   |0 (0) |0 (0)   |2 (1)   |0 (0) |0 (0)   |0 (0) |0 (0) |0 (0) |0 (0) |
|lossat  |401 (4)   |19 (1) |12 (1) |0 (0)  |0 (0)   |0 (0)  |2 (1) |4 (1) |1 (1)  |0 (0)   |0 (0) |2 (1)   |0 (0)   |0 (0) |0 (0)   |1 (1) |0 (0) |0 (0) |1 (1) |
|losslin |328 (6)   |10 (1) |2 (2)  |11 (1) |6 (1)   |0 (0)  |0 (0) |1 (1) |4 (1)  |0 (0)   |2 (1) |0 (0)   |0 (0)   |2 (1) |0 (0)   |0 (0) |1 (1) |1 (1) |0 (0) |
AM


|        |nonRepeat |B3    |ID_B1 |RSINE1 |B1_Mus2 |B4A   |ID4   |
|:-------|:---------|:-----|:-----|:------|:-------|:-----|:-----|
|gainat  |10 (6)    |0 (0) |0 (0) |0 (0)  |1 (1)   |0 (0) |0 (0) |
|gainlin |5 (1)     |3 (1) |3 (1) |0 (0)  |0 (0)   |0 (0) |0 (0) |
|lossat  |101 (6)   |4 (1) |0 (0) |0 (0)  |0 (0)   |0 (0) |1 (1) |
|losslin |363 (4)   |5 (1) |7 (2) |7 (1)  |0 (0)   |1 (1) |0 (0) |
PWKPhJ


|        |nonRepeat |B1_Mur4 |
|:-------|:---------|:-------|
|gainat  |3 (3)     |2 (1)   |
|gainlin |1 (1)     |0 (0)   |
|losslin |2 (2)     |0 (0)   |
CASTEiJ


|        |nonRepeat |MTD   |
|:-------|:---------|:-----|
|gainat  |1 (1)     |0 (0) |
|lossat  |2 (2)     |0 (0) |
|losslin |5 (2)     |1 (1) |
CASTEiJ.PWKPhJ
No significant results

WSBEiJ


|        |nonRepeat |
|:-------|:---------|
|gainat  |1 (1)     |
|lossat  |13 (5)    |
|losslin |9 (2)     |
WSBEiJ.PWKPhJ
No significant results

WSBEiJ.CASTEiJ


|        |RSINE1 |
|:-------|:------|
|losslin |1 (1)  |

---

## Features of significant motifs

Test: losslin 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.6| 3.1| 10.0| 20.1| 25.5| 21.6| 12.5| 5.0| 1.4| 0.3|
|significant     | 0.3| 1.2| 1.2|  3.3| 11.8| 27.6| 27.8| 13.7| 3.4| 6.1| 3.4|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 69.9| 28.0| 2.1| 0.0| 0.0|
|significant     | 99.8|  0.2| 0.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 7.2| 54.5| 28.4| 7.7| 1.8| 0.3| 0.1| 0.0| 0.0| 0.0|
|significant     | 8.0| 60.4| 14.8| 1.7| 1.6| 2.1| 6.0| 4.2| 1.1| 0.2|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 2.0| 23.6| 74.4|
|significant     | 0.2| 9.3| 33.7| 56.8|
Test: lossat 
AT content 


|                |   0|   1|   2|   3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|---:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.3| 2.5| 9.2| 19.3| 25.7| 22.8| 13.4| 5.1| 1.3| 0.3|
|significant     | 0.0| 0.0| 0.3| 2.9|  8.4| 26.4| 30.5| 21.9| 6.0| 1.9| 1.6|
Number of CpGs 


|                |    0|    1|   2|   3|
|:---------------|----:|----:|---:|---:|
|not significant | 68.2| 31.7| 0.1| 0.0|
|significant     | 99.5|  0.5| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 7.2| 55.2| 28.2| 7.4| 1.6| 0.3| 0.1| 0.0| 0.0| 0.0|
|significant     | 4.8| 59.4| 26.6| 2.6| 1.4| 1.5| 1.8| 1.4| 0.5| 0.1|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.3| 22.3| 76.3|
|significant     | 0.1| 6.0| 36.2| 57.6|
Test: gainlin 
AT content 


|                |   0|   1|    2|   3|    4|    5|    6|    7|    8|    9|  10|
|:---------------|---:|---:|----:|---:|----:|----:|----:|----:|----:|----:|---:|
|not significant | 0.0| 0.5|  3.0| 9.6| 19.7| 25.5| 21.9| 12.7|  5.0|  1.5| 0.4|
|significant     | 0.6| 5.6| 10.4| 6.2|  7.3| 10.1|  3.7|  5.6| 20.3| 21.7| 8.5|
Number of CpGs 


|                |    0|    1|   2|   3|
|:---------------|----:|----:|---:|---:|
|not significant | 64.6| 32.9| 2.5| 0.0|
|significant     | 94.4|  5.4| 0.3| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|    6|    7|    8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|----:|----:|----:|---:|---:|
|not significant | 7.6| 55.2| 27.7| 7.4| 1.7|  0.4|  0.1|  0.0| 0.0| 0.0|
|significant     | 5.6| 18.6|  3.7| 0.8| 2.3| 13.5| 39.2| 13.0| 2.8| 0.6|
Nucleotide diversity 


|                |   1|    2|    3|    4|
|:---------------|---:|----:|----:|----:|
|not significant | 0.0|  1.9| 22.5| 75.6|
|significant     | 0.6| 34.9| 40.3| 24.2|
Test: gainat 
AT content 


|                |   0|   1|   2|   3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|---:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.4| 2.8| 9.6| 19.8| 25.9| 22.4| 12.8| 4.8| 1.2| 0.2|
|significant     | 0.0| 0.0| 1.8| 6.2| 17.0| 38.4| 28.6|  6.2| 0.0| 0.0| 1.8|
Number of CpGs 


|                |    0|    1|   2|   3|
|:---------------|----:|----:|---:|---:|
|not significant | 61.4| 37.9| 0.7| 0.0|
|significant     | 88.4| 11.6| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 7.4| 56.1| 27.6| 7.0| 1.5| 0.3| 0.1| 0.0| 0.0| 0.0|
|significant     | 6.2| 62.5| 24.1| 1.8| 1.8| 1.8| 0.9| 0.0| 0.0| 0.9|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.0| 21.1| 78.0|
|significant     | 0.9| 2.7| 16.1| 80.4|
---





## PWK results at CAST top results

Other studies have found that M. m. musculus has a PRDM9 allele similar to the CAST motif, but while we found a denovo CAST motif similar to published work, here we found no de novo PWK motifs. Here, we look at the p-values for the motifs in the top CAST cluster in the other strains

Test: losslin


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
Test: lossat


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
Test: gainlin


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|
|CCAAGTCTGC |0.55     |0.91      |0.39     |1.12      |1         |0.99       |
|CAGACTTGGC |0.67     |1.06      |0.83     |0.96      |0.57      |0.91       |
|AGACTTGGAT |0.48     |1.09      |0.15     |0.81      |0.65      |0.93       |
|AGACTTGGCT |0.81     |0.96      |0.38     |0.89      |0.12      |1.19       |
|AGACTTTGCT |0.79     |1.02      |0.43     |0.91      |0.79      |1.03       |
|AATCCAAGTC |0.78     |1.03      |0.78     |0.95      |0.12      |0.77       |
|AAGCCAAGTC |0.56     |0.92      |0.11     |1.21      |0.52      |0.91       |
|AATCCCAGTC |0.77     |0.94      |0.037    |1.32      |0.66      |0.92       |
|AAGCAAAGTC |0.83     |1.02      |0.83     |1.02      |0.29      |0.88       |
|AAGCCCAGTC |0.94     |0.97      |0.78     |0.95      |0.086     |0.76       |
|AAATCCAAGT |0.96     |1         |0.34     |0.89      |0.58      |0.94       |
|ACTTGGATTC |0.11     |1.22      |0.73     |1.04      |0.58      |0.91       |
Test: gainat


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|
|CCAAGTCTGC |0.53     |1.16      |NA       |NA        |0.57      |1.15       |
|CAGACTTGGC |0.88     |1.03      |0.75     |1.1       |0.2       |1.4        |
|AGACTTGGAT |0.9      |1.03      |0.46     |1.22      |0.89      |1.04       |
|AGACTTGGCT |0.086    |1.46      |1        |0.93      |0.079     |1.41       |
|AGACTTTGCT |0.91     |0.94      |0.91     |1.01      |1         |0.96       |
|AATCCAAGTC |NA       |NA        |0.17     |1.42      |0.39      |1.26       |
|AAGCCAAGTC |0.19     |1.39      |NA       |NA        |0.25      |1.31       |
|AATCCCAGTC |NA       |NA        |1        |0.94      |NA        |NA         |
|AAGCAAAGTC |0.42     |0.78      |1        |1         |0.45      |1.18       |
|AAGCCCAGTC |0.019    |1.8       |NA       |NA        |NA        |NA         |
|AAATCCAAGT |0.42     |0.78      |0.72     |0.87      |0.1       |1.4        |
|ACTTGGATTC |NA       |NA        |0.67     |1.11      |NA        |NA         |
---








## Giant AT to GC plot

All clustered AT to GC plots collapsed together for the loss lineage test

![plot of chunk atToGCAll](figure/atToGCAll.png) 



---

## Correlation between some top motifs

This plot shows correlation of some pre-selected top motifs on different lineages and backgrounds with each other and with 3 linkage-disequilibrium based maps, at fine and broad scales.


Correlation at 5 megabase scale between LD rate maps and hotspot death estimates


|         |   dom|  cast| indian|  FAM1|  AMS2| Spretus1| Spretus2| Spretus4|    AM1|
|:--------|-----:|-----:|------:|-----:|-----:|--------:|--------:|--------:|------:|
|dom      | 0.000| 0.670|  0.835| 0.319| 0.249|    0.108|    0.463|   -0.064|  0.580|
|cast     | 0.000| 0.000|  0.645| 0.248| 0.208|    0.097|    0.344|   -0.013|  0.392|
|indian   | 0.000| 0.000|  0.000| 0.360| 0.229|    0.087|    0.544|   -0.091|  0.691|
|FAM1     | 0.000| 0.000|  0.000| 0.000| 0.419|    0.476|    0.207|    0.237|  0.322|
|AMS2     | 0.000| 0.000|  0.000| 0.000| 0.000|    0.395|    0.073|    0.256|  0.144|
|Spretus1 | 0.000| 0.000|  0.000| 0.000| 0.000|    0.000|    0.020|    0.479|  0.028|
|Spretus2 | 0.000| 0.000|  0.000| 0.000| 0.000|    0.000|    0.000|   -0.198|  0.681|
|Spretus4 | 0.000| 0.000|  0.000| 0.000| 0.000|    0.000|    0.000|    0.000| -0.215|
|AM1      | 0.000| 0.000|  0.000| 0.000| 0.000|    0.000|    0.000|    0.000|  0.000|
![plot of chunk corrTopMotifs](figure/corrTopMotifs.png) 

The following plot shows the breakdown chromosome wide


```
## Error: invalid color name 'fuschia'
```

![plot of chunk corrTopMotifsGenomeWide](figure/corrTopMotifsGenomeWide.png) 



---

## Links to pages of significant motifs


|               |losslin                                                                                                                      |lossat                                                                                                                      |gainlin                                                                                                                      |gainat                                                                                                                      |
|:--------------|:----------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|
|FAM            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/FAM/)            |
|AMS            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/AMS/)            |
|Spretus        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/Spretus/)        |
|AM             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/AM/)             |
|PWKPhJ         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/PWKPhJ/)         |
|CASTEiJ        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/CASTEiJ/)        |
|CASTEiJ.PWKPhJ |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/CASTEiJ.PWKPhJ/) |
|WSBEiJ         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/WSBEiJ/)         |
|WSBEiJ.PWKPhJ  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/WSBEiJ.PWKPhJ/)  |
|WSBEiJ.CASTEiJ |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/losslin/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/lossat/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainlin/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2014_11_07/motifPvalues/gainat/WSBEiJ.CASTEiJ/) |


---


<!--

## All significant motifs




---


## QQ plots - seperate

![plot of chunk pValuesQQPlots](figure/pValuesQQPlots1.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots2.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots3.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots4.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots5.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots6.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots7.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots8.png) 
---




## Compare p-values between methods


```
## Warning: calling par(new=TRUE) with no plot
```

![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods1.png) ![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods2.png) ![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods3.png) 
---


## Compare p-values between lineages within a method

![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage1.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage2.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage3.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage4.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage5.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage6.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage7.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage8.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage9.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage10.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage11.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage12.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage13.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage14.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage15.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage16.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage17.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage18.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage19.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage20.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage21.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage22.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage23.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage24.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage25.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage26.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage27.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage28.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage29.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage30.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage31.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage32.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage33.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage34.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage35.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage36.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage37.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage38.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage39.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage40.png) 
---





-->
