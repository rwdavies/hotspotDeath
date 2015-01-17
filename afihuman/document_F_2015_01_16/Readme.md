#Hotspot death
##Species: afihuman
##Run version: 2015_01_16
---



##Changelog
Date refers to first version with changes. If there is a plus after, then all subsequent versions have the change
* **2015_01_09** Fixed bug in human pipeline where ancestral number of motifs was not being properly calculated and utilized
* **2015_01_06** Better normalization for calculating ancestral broad scale plots
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
|mouseFilter           |NA                                                    |What folder used for mice                                                |
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
## '/Net/dense/data/itch/rwdavies/motifLoss/afihuman/input_A_2015_01_16/forFilteringPlot.RData',
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
| 2.8810|  1.1077|  1.7733|             0.6828|

```
## Error: 'names' attribute [13] must be the same length as the vector [10]
```

Number of Derived Mutations down a specific lineage


|der        |der        |der        |der        |der        |der        |der        |der        |der        |der        |
|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|
|26,621,100 | 9,748,431 |12,008,463 | 5,789,106 | 5,754,222 | 1,306,032 | 4,297,254 | 3,620,908 | 3,610,379 |   520,460 |
Branch length as percent of alignable genome


|   der|   der|   der|   der|   der|   der|   der|   der|   der|   der|
|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|
| 2.403| 0.880| 1.084| 0.523| 0.519| 0.118| 0.388| 0.327| 0.326| 0.047|
Branch length compared to ancestral of all lineages in SNPs


|der        |der        |der        |der        |der        |der        |der        |der        |der        |der        |
|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|
|   520,460 | 3,610,379 | 3,620,908 | 4,297,254 | 1,306,032 | 5,754,222 | 5,789,106 |12,008,463 | 9,748,431 |26,621,100 |
Branch length compared to ancestral as percent of alignable genome


|   der|   der|   der|   der|   der|   der|   der|   der|   der|   der|
|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|-----:|
| 0.047| 0.326| 0.327| 0.388| 0.118| 0.519| 0.523| 1.084| 0.880| 2.403|


---

## Number of significant motifs per lineage per test

Note that the following numbers are only for motifs that have been clustered

Number of motifs (clusters) per test and lineage


|        |MACAQUE  |AHCGO      |ORANGUTAN |AHCG   |GORILLA |CHIMP     |AHN     |AHD     |HUMAN(DENI) |NEAN  |DENI    |
|:-------|:--------|:----------|:---------|:------|:-------|:---------|:-------|:-------|:-----------|:-----|:-------|
|gainat  |46 (14)  |0 (0)      |0 (0)     |5 (3)  |53 (21) |197 (101) |10 (9)  |8 (7)   |2 (2)       |1 (1) |1 (1)   |
|gainlin |326 (42) |2400 (141) |20 (3)    |0 (0)  |1 (1)   |80 (19)   |7 (2)   |3 (3)   |1 (1)       |1 (1) |44 (13) |
|lossat  |3 (2)    |3 (3)      |4 (3)     |15 (5) |4 (4)   |195 (115) |60 (8)  |60 (9)  |3 (3)       |0 (0) |0 (0)   |
|losslin |155 (45) |279 (43)   |140 (13)  |65 (5) |7 (3)   |65 (16)   |108 (7) |104 (7) |0 (0)       |2 (1) |3 (3)   |
MACAQUE


|        |AluSx1 |AluSx  |AluJb  |nonRepeat |AluSq2 |AluSz6 |AluJr  |AluSz  |AluSg  |THE1B  |THE1D |AluJo |THE1A |AluSp |AluSc |AluY  |THE1C |Tigger3a |AluSq |THE1B-int |Tigger3b |
|:-------|:------|:------|:------|:---------|:------|:------|:------|:------|:------|:------|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:--------|:-----|:---------|:--------|
|gainat  |32 (7) |9 (4)  |0 (0)  |0 (0)     |1 (1)  |0 (0)  |0 (0)  |4 (2)  |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)    |0 (0) |0 (0)     |0 (0)    |
|gainlin |69 (4) |64 (4) |49 (3) |6 (1)     |27 (5) |22 (4) |18 (4) |10 (2) |10 (2) |10 (1) |9 (1) |7 (3) |8 (1) |5 (2) |5 (1) |2 (1) |3 (1) |0 (0)    |1 (1) |1 (1)     |0 (0)    |
|lossat  |1 (1)  |0 (0)  |0 (0)  |2 (1)     |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)    |0 (0) |0 (0)     |0 (0)    |
|losslin |33 (7) |33 (8) |31 (6) |30 (10)   |4 (4)  |9 (1)  |3 (1)  |4 (1)  |2 (1)  |0 (0)  |0 (0) |1 (1) |0 (0) |1 (1) |0 (0) |1 (1) |0 (0) |2 (2)    |0 (0) |0 (0)     |1 (1)    |
AHCGO


|        |AluSx   |AluSx1  |AluJb   |AluSq2  |AluSz   |AluSz6  |AluJr   |AluJo   |AluSp    |THE1B  |AluSg   |AluSq   |AluY    |AluSc  |L1PA8  |nonRepeat |THE1D  |AluSx3 |FLAM_C |L1PA7  |MSTA   |THE1C |AluJr4 |AluSc8 |MSTB  |L1PA16 |MER1B |MLT1A0 |THE1A |
|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:--------|:------|:-------|:-------|:-------|:------|:------|:---------|:------|:------|:------|:------|:------|:-----|:------|:------|:-----|:------|:-----|:------|:-----|
|gainlin |379 (1) |368 (3) |308 (1) |232 (3) |162 (4) |151 (4) |142 (6) |115 (4) |105 (12) |81 (9) |73 (10) |68 (12) |34 (11) |44 (8) |33 (5) |5 (2)     |26 (8) |18 (7) |12 (7) |11 (5) |10 (3) |9 (6) |6 (3)  |2 (2)  |2 (1) |1 (1)  |1 (1) |1 (1)  |1 (1) |
|lossat  |0 (0)   |2 (2)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)    |0 (0)  |0 (0)   |0 (0)   |0 (0)   |0 (0)  |0 (0)  |1 (1)     |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0) |0 (0)  |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0) |
|losslin |78 (5)  |84 (8)  |2 (1)   |31 (7)  |10 (1)  |10 (1)  |1 (1)   |1 (1)   |11 (2)   |5 (1)  |5 (3)   |0 (0)   |17 (7)  |1 (1)  |0 (0)  |23 (4)    |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0) |0 (0)  |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0) |
ORANGUTAN


|        |nonRepeat |MIR    |MIRb   |L1PA13 |MIRc  |MIR3  |AluJr |MLT1A0 |THE1D |
|:-------|:---------|:------|:------|:------|:-----|:-----|:-----|:------|:-----|
|gainlin |1 (1)     |15 (1) |4 (1)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |
|lossat  |4 (3)     |0 (0)  |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |
|losslin |85 (5)    |15 (1) |12 (1) |8 (1)  |7 (1) |6 (1) |5 (1) |1 (1)  |1 (1) |
AHCG


|        |nonRepeat |AluY  |L2a   |
|:-------|:---------|:-----|:-----|
|gainat  |1 (1)     |4 (2) |0 (0) |
|lossat  |12 (3)    |2 (1) |1 (1) |
|losslin |65 (5)    |0 (0) |0 (0) |
GORILLA


|        |AluSx  |AluSx1 |AluSz6 |nonRepeat |AluSz |AluSp |AluJo |AluSq2 |L1PA13 |L1PA14 |L1PA8A |
|:-------|:------|:------|:------|:---------|:-----|:-----|:-----|:------|:------|:------|:------|
|gainat  |21 (6) |20 (9) |5 (1)  |0 (0)     |3 (1) |2 (2) |1 (1) |1 (1)  |0 (0)  |0 (0)  |0 (0)  |
|gainlin |0 (0)  |1 (1)  |0 (0)  |0 (0)     |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0)  |0 (0)  |0 (0)  |
|lossat  |0 (0)  |0 (0)  |0 (0)  |2 (2)     |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0)  |1 (1)  |1 (1)  |
|losslin |3 (1)  |0 (0)  |0 (0)  |3 (1)     |0 (0) |0 (0) |0 (0) |0 (0)  |1 (1)  |0 (0)  |0 (0)  |
AHC
No significant results

CHIMP


|        |nonRepeat |L2c     |AluSx1 |L1MEc  |MLT1C  |AluSx |AluJb |AluJo |L1PA13 |MIR   |MIRb  |AluJr |AluSz6 |L1MB3 |L1PA17 |AluSq |AluSq2 |L1ME2 |L2a   |AluSz |L1PA15 |MLT1D |MLT2B1 |L1M1  |L1M5  |L1MA9 |L1MB7 |L1ME1 |L1PA14 |L2b   |MER41B |
|:-------|:---------|:-------|:------|:------|:------|:-----|:-----|:-----|:------|:-----|:-----|:-----|:------|:-----|:------|:-----|:------|:-----|:-----|:-----|:------|:-----|:------|:-----|:-----|:-----|:-----|:-----|:------|:-----|:------|
|gainat  |153 (75)  |12 (6)  |8 (4)  |2 (2)  |0 (0)  |5 (2) |5 (5) |5 (1) |0 (0)  |0 (0) |2 (2) |0 (0) |3 (2)  |0 (0) |0 (0)  |2 (2) |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |
|gainlin |54 (5)    |0 (0)   |16 (7) |0 (0)  |1 (1)  |2 (2) |0 (0) |1 (1) |0 (0)  |0 (0) |0 (0) |2 (1) |1 (1)  |0 (0) |0 (0)  |0 (0) |3 (1)  |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |
|lossat  |105 (56)  |27 (15) |0 (0)  |12 (7) |1 (1)  |2 (1) |5 (3) |0 (0) |5 (2)  |6 (5) |3 (2) |0 (0) |0 (0)  |4 (3) |4 (2)  |0 (0) |0 (0)  |3 (1) |3 (3) |1 (1) |2 (1)  |2 (2) |2 (2)  |1 (1) |1 (1) |1 (1) |1 (1) |1 (1) |1 (1)  |1 (1) |1 (1)  |
|losslin |44 (8)    |0 (0)   |0 (0)  |0 (0)  |11 (1) |2 (2) |0 (0) |2 (1) |2 (1)  |0 (0) |0 (0) |2 (1) |0 (0)  |0 (0) |0 (0)  |1 (1) |0 (0)  |0 (0) |0 (0) |1 (1) |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |
AHN


|        |nonRepeat |MSTA   |MSTB  |MER20  |AluSx |MER5A |AluSx1 |LTR8  |
|:-------|:---------|:------|:-----|:------|:-----|:-----|:------|:-----|
|gainat  |10 (9)    |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0) |0 (0)  |0 (0) |
|gainlin |0 (0)     |6 (1)  |0 (0) |0 (0)  |0 (0) |0 (0) |1 (1)  |0 (0) |
|lossat  |42 (6)    |11 (1) |7 (1) |0 (0)  |0 (0) |0 (0) |0 (0)  |0 (0) |
|losslin |65 (1)    |16 (1) |8 (1) |14 (1) |2 (1) |2 (1) |0 (0)  |1 (1) |
AHD


|        |nonRepeat |MSTA   |MER20  |MSTB  |AluSx |AluY  |MER5A |AluSx1 |
|:-------|:---------|:------|:------|:-----|:-----|:-----|:-----|:------|
|gainat  |7 (6)     |0 (0)  |0 (0)  |0 (0) |0 (0) |1 (1) |0 (0) |0 (0)  |
|gainlin |0 (0)     |2 (2)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |1 (1)  |
|lossat  |41 (6)    |12 (1) |0 (0)  |6 (1) |0 (0) |1 (1) |0 (0) |0 (0)  |
|losslin |63 (2)    |15 (1) |14 (1) |8 (1) |2 (1) |0 (0) |2 (1) |0 (0)  |
HUMAN(NEAN)
No significant results

HUMAN(DENI)


|        |nonRepeat |AluY  |
|:-------|:---------|:-----|
|gainat  |2 (2)     |0 (0) |
|gainlin |0 (0)     |1 (1) |
|lossat  |3 (3)     |0 (0) |
NEAN


|        |nonRepeat |AluSq2 |
|:-------|:---------|:------|
|gainat  |1 (1)     |0 (0)  |
|gainlin |0 (0)     |1 (1)  |
|losslin |2 (1)     |0 (0)  |
DENI


|        |AluSx1 |AluY   |AluSx  |nonRepeat |AluSg |AluJb |AluSq2 |
|:-------|:------|:------|:------|:---------|:-----|:-----|:------|
|gainat  |0 (0)  |0 (0)  |0 (0)  |1 (1)     |0 (0) |0 (0) |0 (0)  |
|gainlin |15 (3) |14 (3) |10 (3) |1 (1)     |2 (1) |1 (1) |1 (1)  |
|losslin |1 (1)  |1 (1)  |0 (0)  |1 (1)     |0 (0) |0 (0) |0 (0)  |

---

## Features of significant motifs

Test: losslin 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.7| 3.1|  9.0| 17.8| 23.3| 21.3| 13.9| 7.1| 2.9| 0.7|
|significant     | 0.0| 0.0| 2.8| 16.8| 29.1| 26.1| 13.8|  5.9| 1.3| 1.8| 2.4|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 73.2| 25.1| 1.6| 0.1| 0.0|
|significant     | 94.2|  5.8| 0.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 6.1| 53.0| 29.7| 8.6| 2.1| 0.4| 0.1| 0.0| 0.0| 0.0|
|significant     | 5.9| 53.5| 31.5| 4.7| 2.0| 1.1| 0.3| 0.3| 0.7| 0.1|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.8| 26.9| 71.3|
|significant     | 0.1| 3.8| 20.6| 75.5|
Test: lossat 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.5| 2.6|  8.5| 17.0| 23.1| 22.2| 14.9| 7.3| 2.9| 0.8|
|significant     | 0.0| 1.7| 3.1| 12.5| 24.4| 20.3| 16.6| 10.5| 7.1| 3.4| 0.3|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 75.9| 23.6| 0.5| 0.0| 0.0|
|significant     | 72.9| 26.1| 1.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|    4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|----:|---:|---:|---:|---:|---:|---:|
|not significant | 6.0| 52.4| 30.0|  8.9| 2.2| 0.4| 0.0| 0.0| 0.0| 0.0|
|significant     | 5.1| 45.4| 36.3| 10.8| 2.0| 0.3| 0.0| 0.0| 0.0| 0.0|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.9| 26.8| 71.3|
|significant     | 0.0| 0.7| 27.5| 71.9|
Test: gainlin 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.7| 3.5|  9.9| 19.4| 24.7| 21.0| 12.4| 5.7| 2.1| 0.5|
|significant     | 0.1| 1.5| 5.9| 17.8| 28.9| 22.6| 10.5|  3.8| 2.7| 3.6| 2.6|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 66.0| 31.1| 2.9| 0.1| 0.0|
|significant     | 67.8| 31.6| 0.6| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 6.6| 55.0| 28.6| 7.6| 1.8| 0.4| 0.1| 0.0| 0.0| 0.0|
|significant     | 5.6| 54.7| 26.2| 4.2| 2.2| 1.5| 2.0| 2.7| 0.7| 0.1|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.5| 24.2| 74.4|
|significant     | 0.1| 6.3| 21.0| 72.5|
Test: gainat 
AT content 


|                |   0|   1|    2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|----:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.6|  3.1|  9.6| 19.5| 25.2| 21.5| 12.6| 5.5| 1.9| 0.5|
|significant     | 0.7| 2.4| 11.1| 17.3| 29.8| 20.8| 10.7|  3.1| 2.8| 0.7| 0.7|
Number of CpGs 


|                |    0|    1|    2|   3|   4|
|:---------------|----:|----:|----:|---:|---:|
|not significant | 64.2| 34.2|  1.5| 0.0| 0.0|
|significant     | 45.7| 44.3| 10.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 6.6| 55.2| 28.3| 7.6| 1.8| 0.4| 0.1| 0.0| 0.0| 0.0|
|significant     | 5.5| 51.9| 30.8| 6.6| 2.4| 0.3| 0.3| 1.4| 0.3| 0.3|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.4| 22.9| 75.7|
|significant     | 0.3| 2.4| 22.5| 74.7|
---





## PWK results at CAST top results

Only for the mouse analysis


---








## Giant AT to GC plot

All clustered AT to GC plots collapsed together for the loss lineage test

![plot of chunk atToGCAll](figure/atToGCAll.png) 



---

## Correlation between some top motifs

Currently only for mice














---

## Links to pages of significant motifs


|            |losslin                                                                                                                       |lossat                                                                                                                       |gainlin                                                                                                                       |gainat                                                                                                                       |
|:-----------|:-----------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------|
|MACAQUE     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/MACAQUE/)     |
|AHCGO       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/AHCGO/)       |
|ORANGUTAN   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/ORANGUTAN/)   |
|AHCG        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/AHCG/)        |
|GORILLA     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/GORILLA/)     |
|AHC         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/AHC/)         |
|CHIMP       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/CHIMP/)       |
|AHN         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/AHN/)         |
|AHD         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/AHD/)         |
|HUMAN(NEAN) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/HUMAN_NEAN_/) |
|HUMAN(DENI) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/HUMAN_DENI_/) |
|NEAN        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/NEAN/)        |
|DENI        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/losslin/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/lossat/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainlin/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_16/motifPvalues/gainat/DENI/)        |


---


<!--

## All significant motifs




---


## QQ plots - seperate

![plot of chunk pValuesQQPlots](figure/pValuesQQPlots1.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots2.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots3.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots4.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots5.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots6.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots7.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots8.png) 
---




## Compare p-values between methods

![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods1.png) ![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods2.png) ![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods3.png) 
---


## Compare p-values between lineages within a method

![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage1.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage2.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage3.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage4.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage5.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage6.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage7.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage8.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage9.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage10.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage11.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage12.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage13.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage14.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage15.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage16.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage17.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage18.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage19.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage20.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage21.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage22.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage23.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage24.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage25.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage26.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage27.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage28.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage29.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage30.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage31.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage32.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage33.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage34.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage35.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage36.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage37.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage38.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage39.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage40.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage41.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage42.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage43.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage44.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage45.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage46.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage47.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage48.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage49.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage50.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage51.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage52.png) 
---





-->
