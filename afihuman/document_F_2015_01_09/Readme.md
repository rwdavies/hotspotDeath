#Hotspot death
##Species: afihuman
##Run version: 2015_01_09
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
## '/Net/dense/data/itch/rwdavies/motifLoss/afihuman/input_A_2015_01_09/forFilteringPlot.RData',
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
Number of Derived Mutations down a specific lineage


|MACAQUE    |AHCGO      |ORANGUTAN  |AHCG       |GORILLA    |AHC        |CHIMP      |AHN        |AHD        |HUMAN(NEAN) |HUMAN(DENI) |NEAN       |DENI       |
|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:-----------|:-----------|:----------|:----------|
|26,621,100 | 9,748,431 |12,008,463 | 5,789,106 | 5,754,222 | 1,306,032 | 4,297,254 | 3,620,908 | 3,610,379 |   520,460  |   531,019  |   725,802 |   785,327 |
Branch length as percent of alignable genome


| MACAQUE| AHCGO| ORANGUTAN|  AHCG| GORILLA|   AHC| CHIMP|   AHN|   AHD| HUMAN(NEAN)| HUMAN(DENI)|  NEAN|  DENI|
|-------:|-----:|---------:|-----:|-------:|-----:|-----:|-----:|-----:|-----------:|-----------:|-----:|-----:|
|   2.403| 0.880|     1.084| 0.523|   0.519| 0.118| 0.388| 0.327| 0.326|       0.047|       0.048| 0.066| 0.071|
Branch length compared to ancestral of all lineages in SNPs


|DENI       |NEAN       |HUMAN(DENI) |HUMAN(NEAN) |CHIMP      |GORILLA    |ORANGUTAN  |MACAQUE    |
|:----------|:----------|:-----------|:-----------|:----------|:----------|:----------|:----------|
|24,860,183 |24,800,658 |24,605,875  |24,595,316  |21,140,823 |21,291,759 |21,756,894 |26,621,100 |
Branch length compared to ancestral as percent of alignable genome


|  DENI|  NEAN| HUMAN(DENI)| HUMAN(NEAN)| CHIMP| GORILLA| ORANGUTAN| MACAQUE|
|-----:|-----:|-----------:|-----------:|-----:|-------:|---------:|-------:|
| 2.244| 2.239|       2.221|       2.220| 1.908|   1.922|     1.964|   2.403|


---

## Number of significant motifs per lineage per test

Note that the following numbers are only for motifs that have been clustered

Number of motifs (clusters) per test and lineage


|        |MACAQUE  |AHCGO      |ORANGUTAN |AHCG   |GORILLA |CHIMP     |AHN     |AHD    |HUMAN(NEAN) |HUMAN(DENI) |NEAN  |DENI    |
|:-------|:--------|:----------|:---------|:------|:-------|:---------|:-------|:------|:-----------|:-----------|:-----|:-------|
|gainat  |46 (14)  |1 (1)      |0 (0)     |5 (3)  |61 (25) |237 (123) |15 (12) |9 (8)  |0 (0)       |3 (3)       |2 (2) |1 (1)   |
|gainlin |237 (46) |1641 (141) |4 (1)     |0 (0)  |2 (2)   |45 (19)   |0 (0)   |0 (0)  |2 (1)       |2 (1)       |1 (1) |41 (12) |
|lossat  |5 (4)    |3 (3)      |5 (3)     |16 (6) |7 (6)   |232 (133) |65 (9)  |65 (9) |0 (0)       |3 (3)       |0 (0) |0 (0)   |
|losslin |74 (27)  |277 (45)   |41 (7)    |1 (1)  |7 (4)   |30 (10)   |14 (7)  |13 (7) |0 (0)       |0 (0)       |1 (1) |3 (3)   |
MACAQUE


|        |AluSx1 |AluSx  |AluSz6 |AluJr  |AluSz  |AluSq2 |AluSg  |AluJo |AluSp |nonRepeat |AluSc |AluY  |Tigger3a |AluSc8 |AluSq |L1MA2 |THE1B |THE1B-int |THE1C |THE1D |Tigger3b |
|:-------|:------|:------|:------|:------|:------|:------|:------|:-----|:-----|:---------|:-----|:-----|:--------|:------|:-----|:-----|:-----|:---------|:-----|:-----|:--------|
|gainat  |32 (7) |9 (4)  |0 (0)  |0 (0)  |4 (2)  |1 (1)  |0 (0)  |0 (0) |0 (0) |0 (0)     |0 (0) |0 (0) |0 (0)    |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0)     |0 (0) |0 (0) |0 (0)    |
|gainlin |75 (4) |48 (9) |24 (4) |22 (5) |14 (2) |16 (6) |10 (3) |9 (3) |6 (2) |0 (0)     |5 (1) |2 (1) |0 (0)    |1 (1)  |1 (1) |0 (0) |1 (1) |1 (1)     |1 (1) |1 (1) |0 (0)    |
|lossat  |1 (1)  |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0)  |0 (0) |0 (0) |4 (3)     |0 (0) |0 (0) |0 (0)    |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0)     |0 (0) |0 (0) |0 (0)    |
|losslin |35 (7) |8 (4)  |10 (2) |3 (1)  |4 (1)  |1 (1)  |2 (1)  |1 (1) |1 (1) |3 (3)     |0 (0) |1 (1) |3 (2)    |0 (0)  |0 (0) |1 (1) |0 (0) |0 (0)     |0 (0) |0 (0) |1 (1)    |
AHCGO


|        |AluSx1  |AluSx   |AluSq2  |AluSz6  |AluJr   |AluSz   |AluJo   |AluSp   |AluSg   |AluSq   |AluY    |AluSc   |AluSx3 |L1PA8  |THE1B  |nonRepeat |THE1D |AluJr4 |FLAM_C |L1PA7 |THE1C |MLT1A0 |AluJb |AluSc8 |L1PA16 |MER1B |
|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:-------|:------|:------|:------|:---------|:-----|:------|:------|:-----|:-----|:------|:-----|:------|:------|:-----|
|gainat  |0 (0)   |1 (1)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)  |0 (0)  |0 (0)  |0 (0)     |0 (0) |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0)  |0 (0) |
|gainlin |377 (3) |185 (4) |150 (7) |159 (4) |148 (6) |121 (7) |119 (5) |75 (14) |68 (11) |69 (12) |36 (11) |35 (13) |23 (7) |23 (3) |16 (8) |3 (1)     |8 (5) |7 (4)  |7 (4)  |3 (3) |3 (3) |2 (2)  |1 (1) |1 (1)  |1 (1)  |1 (1) |
|lossat  |2 (2)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)   |0 (0)  |0 (0)  |0 (0)  |1 (1)     |0 (0) |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0)  |0 (0) |
|losslin |89 (8)  |84 (9)  |37 (9)  |10 (1)  |1 (1)   |10 (1)  |1 (1)   |17 (2)  |5 (3)   |0 (0)   |17 (7)  |1 (1)   |0 (0)  |0 (0)  |0 (0)  |5 (2)     |0 (0) |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0)  |0 (0) |
ORANGUTAN


|        |MIRb   |L1PA13 |MIRc  |AluJr |MIR3  |nonRepeat |MIR   |
|:-------|:------|:------|:-----|:-----|:-----|:---------|:-----|
|gainlin |4 (1)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0)     |0 (0) |
|lossat  |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |5 (3)     |0 (0) |
|losslin |12 (1) |8 (1)  |7 (1) |6 (1) |6 (1) |1 (1)     |1 (1) |
AHCG


|        |nonRepeat |AluY  |L2a   |
|:-------|:---------|:-----|:-----|
|gainat  |1 (1)     |4 (2) |0 (0) |
|lossat  |13 (4)    |2 (1) |1 (1) |
|losslin |1 (1)     |0 (0) |0 (0) |
GORILLA


|        |AluSx  |AluSx1 |AluSz6 |AluSp |AluSz |nonRepeat |AluSq2 |L1PA8A |AluJb |AluJo |L1MB3 |L1PA11 |L1PA13 |L1PA14 |
|:-------|:------|:------|:------|:-----|:-----|:---------|:------|:------|:-----|:-----|:-----|:------|:------|:------|
|gainat  |26 (8) |21 (9) |5 (1)  |2 (2) |3 (1) |0 (0)     |2 (2)  |0 (0)  |1 (1) |1 (1) |0 (0) |0 (0)  |0 (0)  |0 (0)  |
|gainlin |0 (0)  |1 (1)  |0 (0)  |1 (1) |0 (0) |0 (0)     |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0)  |0 (0)  |
|lossat  |0 (0)  |1 (1)  |0 (0)  |0 (0) |0 (0) |2 (2)     |0 (0)  |2 (1)  |0 (0) |0 (0) |0 (0) |1 (1)  |0 (0)  |1 (1)  |
|losslin |4 (1)  |0 (0)  |0 (0)  |0 (0) |0 (0) |1 (1)     |0 (0)  |0 (0)  |0 (0) |0 (0) |1 (1) |0 (0)  |1 (1)  |0 (0)  |
AHC
No significant results

CHIMP


|        |nonRepeat |L2c     |AluSx1 |L1MEc  |AluJb |AluSx |MIR   |AluJo |AluSz6 |L1PA13 |L2a   |MIRb  |AluJr |AluSq |L1MB3 |L1ME2 |L1PA17 |MLT1D |AluSg |AluSz |L1PA15 |MLT2B1 |AluSq2 |L1M1  |L1M5  |L1MA9 |L1MB7 |L1ME1 |L1PA14 |L2b   |MER41B |MLT1C |THE1B-int |
|:-------|:---------|:-------|:------|:------|:-----|:-----|:-----|:-----|:------|:------|:-----|:-----|:-----|:-----|:-----|:-----|:------|:-----|:-----|:-----|:------|:------|:------|:-----|:-----|:-----|:-----|:-----|:------|:-----|:------|:-----|:---------|
|gainat  |179 (87)  |15 (8)  |10 (4) |2 (2)  |6 (6) |6 (3) |2 (2) |5 (1) |6 (5)  |0 (0)  |1 (1) |2 (2) |0 (0) |3 (2) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0)     |
|gainlin |17 (1)    |0 (0)   |16 (7) |0 (0)  |0 (0) |5 (5) |0 (0) |1 (1) |1 (1)  |0 (0)  |0 (0) |0 (0) |2 (1) |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |2 (2) |0 (0) |0 (0)  |0 (0)  |1 (1)  |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0)     |
|lossat  |128 (67)  |31 (16) |0 (0)  |13 (8) |8 (4) |2 (1) |8 (7) |0 (0) |0 (0)  |5 (2)  |4 (4) |3 (2) |0 (0) |0 (0) |4 (3) |4 (1) |4 (2)  |3 (2) |0 (0) |1 (1) |2 (1)  |2 (2)  |0 (0)  |1 (1) |1 (1) |1 (1) |1 (1) |1 (1) |1 (1)  |1 (1) |1 (1)  |1 (1) |1 (1)     |
|losslin |17 (1)    |0 (0)   |1 (1)  |0 (0)  |0 (0) |1 (1) |0 (0) |2 (1) |0 (0)  |2 (1)  |1 (1) |1 (1) |3 (1) |1 (1) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0) |1 (1) |0 (0)  |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)  |0 (0) |0 (0)  |0 (0) |0 (0)     |
AHN


|        |nonRepeat |MSTA   |MSTB  |MER20 |AluSx |
|:-------|:---------|:------|:-----|:-----|:-----|
|gainat  |15 (12)   |0 (0)  |0 (0) |0 (0) |0 (0) |
|lossat  |44 (6)    |12 (1) |8 (1) |1 (1) |0 (0) |
|losslin |4 (2)     |0 (0)  |4 (2) |4 (2) |2 (1) |
AHD


|        |nonRepeat |MSTA   |MSTB  |MER20 |AluY  |AluSx |
|:-------|:---------|:------|:-----|:-----|:-----|:-----|
|gainat  |8 (7)     |0 (0)  |0 (0) |0 (0) |1 (1) |0 (0) |
|lossat  |44 (6)    |12 (1) |8 (1) |0 (0) |1 (1) |0 (0) |
|losslin |4 (2)     |0 (0)  |4 (2) |4 (2) |0 (0) |1 (1) |
HUMAN(NEAN)


|        |AluY  |
|:-------|:-----|
|gainlin |2 (1) |
HUMAN(DENI)


|        |nonRepeat |AluY  |
|:-------|:---------|:-----|
|gainat  |3 (3)     |0 (0) |
|gainlin |0 (0)     |2 (1) |
|lossat  |3 (3)     |0 (0) |
NEAN


|        |nonRepeat |AluSx1 |
|:-------|:---------|:------|
|gainat  |2 (2)     |0 (0)  |
|gainlin |0 (0)     |1 (1)  |
|losslin |1 (1)     |0 (0)  |
DENI


|        |AluSx1 |AluY   |AluSx |nonRepeat |AluSg |AluSq2 |
|:-------|:------|:------|:-----|:---------|:-----|:------|
|gainat  |0 (0)  |0 (0)  |0 (0) |1 (1)     |0 (0) |0 (0)  |
|gainlin |17 (3) |12 (3) |8 (3) |1 (1)     |2 (1) |1 (1)  |
|losslin |1 (1)  |1 (1)  |0 (0) |1 (1)     |0 (0) |0 (0)  |

---

## Features of significant motifs

Test: losslin 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|    8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|----:|---:|---:|
|not significant | 0.0| 0.2| 1.7|  6.7| 14.3| 19.4| 19.9| 16.9| 12.0| 6.9| 2.0|
|significant     | 0.0| 0.0| 2.1| 14.8| 31.1| 25.9| 13.2|  5.8|  1.8| 1.8| 3.4|
Number of CpGs 


|                |    0|   1|   2|   3|
|:---------------|----:|---:|---:|---:|
|not significant | 95.8| 4.0| 0.2| 0.0|
|significant     | 92.1| 7.9| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|    4|   5|   6|   7|   8|
|:---------------|---:|----:|----:|----:|---:|---:|---:|---:|
|not significant | 4.8| 46.2| 32.8| 12.1| 3.6| 0.5| 0.1| 0.0|
|significant     | 5.5| 62.0| 24.0|  3.4| 2.9| 1.6| 0.5| 0.0|
Nucleotide diversity 


|                |   2|    3|    4|
|:---------------|---:|----:|----:|
|not significant | 4.3| 36.3| 59.4|
|significant     | 4.5| 13.5| 82.1|
Test: lossat 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.5| 2.6|  8.5| 17.0| 23.1| 22.2| 14.9| 7.3| 2.9| 0.8|
|significant     | 0.0| 1.8| 3.5| 12.3| 23.4| 21.6| 16.1| 10.8| 7.0| 3.2| 0.3|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 75.9| 23.6| 0.5| 0.0| 0.0|
|significant     | 72.2| 26.9| 0.9| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|    4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|----:|---:|---:|---:|---:|---:|---:|
|not significant | 6.0| 52.4| 30.0|  8.9| 2.2| 0.4| 0.0| 0.0| 0.0| 0.0|
|significant     | 5.3| 44.4| 36.5| 11.1| 2.0| 0.6| 0.0| 0.0| 0.0| 0.0|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.9| 26.8| 71.3|
|significant     | 0.0| 0.9| 28.1| 71.1|
Test: gainlin 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|    8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|----:|---:|---:|
|not significant | 0.0| 0.2| 1.9|  7.5| 16.7| 21.6| 19.4| 14.2| 10.4| 6.1| 2.0|
|significant     | 0.0| 0.7| 6.4| 21.0| 31.9| 21.4| 10.4|  2.6|  1.2| 2.4| 1.9|
Number of CpGs 


|                |    0|    1|   2|   3|
|:---------------|----:|----:|---:|---:|
|not significant | 91.9|  7.8| 0.3| 0.0|
|significant     | 64.6| 34.7| 0.7| 0.0|
Maximum run length 


|                |   1|    2|    3|    4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|----:|---:|---:|---:|---:|---:|---:|
|not significant | 5.1| 48.1| 31.8| 11.0| 3.3| 0.7| 0.1| 0.0| 0.0| 0.0|
|significant     | 4.4| 59.3| 28.3|  4.1| 2.3| 0.8| 0.5| 0.2| 0.0| 0.1|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 4.6| 33.2| 62.2|
|significant     | 0.1| 2.2| 18.2| 79.5|
Test: gainat 
AT content 


|                |   0|   1|    2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|----:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.6|  3.1|  9.6| 19.5| 25.2| 21.5| 12.6| 5.5| 1.9| 0.5|
|significant     | 0.6| 2.3| 10.5| 16.4| 30.4| 20.8| 10.2|  5.0| 2.6| 0.6| 0.6|
Number of CpGs 


|                |    0|    1|    2|   3|   4|
|:---------------|----:|----:|----:|---:|---:|
|not significant | 64.2| 34.2|  1.5| 0.0| 0.0|
|significant     | 48.2| 41.5| 10.2| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 6.6| 55.2| 28.3| 7.6| 1.8| 0.4| 0.1| 0.0| 0.0| 0.0|
|significant     | 5.3| 51.8| 31.6| 6.4| 2.6| 0.3| 0.3| 1.2| 0.3| 0.3|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.4| 22.9| 75.7|
|significant     | 0.3| 2.3| 23.4| 74.0|
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


















