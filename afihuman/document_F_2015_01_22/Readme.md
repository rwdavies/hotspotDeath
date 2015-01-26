#Hotspot death
##Species: afihuman
##Run version: 2015_01_22
---



##Changelog
Date refers to first version with changes. If there is a plus after, then all subsequent versions have the change
* **2015_01_22** For primates, remove all Alus from consideration during the summarization procedure
* **2015_01_22** Choose a p-value threshold which isn't necessarily genome-wide significant - now a parameter
* **2015_01_22** Choosing significant p-values - require that the other test - AT to GC or lineage - also be a value, now a parameter
* **2015_01_22** Change the way normalization is done for p-value calculation - only remove motifs from consideration after using them for normalizing the p-value calculations (still don't test them though)
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

|Parameter             |Value                                                 |Description                                                                 |
|:---------------------|:-----------------------------------------------------|:---------------------------------------------------------------------------|
|mouseFilter           |NA                                                    |What folder used for mice                                                   |
|mrle                  |10                                                    |Maximum run length less than or equal to                                    |
|ndge                  |0                                                     |Nuclotide diversity greater than or equal to                                |
|pwmMinimum            |0.1                                                   |When a PWM has 0 entry, what to reset the 0 values to                       |
|Klist                 |10                                                    |Range of K to use                                                           |
|gcW1                  |100                                                   |Range to test for GC increase                                               |
|gcW2                  |1000                                                  |Range to plot GC increases                                                  |
|gcW3                  |10                                                    |Smoothing window for AT to GC plots                                         |
|ctge                  |10                                                    |For a p-value to be generated, each cell must have at least this number     |
|rgte                  |50                                                    |For a row to count, there must be this many entries                         |
|testingNames          |losslin,lossat,gainlin,gainat                         |(Short) names of the tests we will perform                                  |
|plottingNames         |Loss Lineage,Loss AT to GC,Gain Lineage,Gain AT to GC |Plotting names of the tests we will perform                                 |
|nTests                |4                                                     |Number of tests to be performed                                             |
|globalPThresh         |9.52743902439024e-08                                  |P-value threshold (if NA use total number of tests performed)               |
|pThresh               |0.000152439024390244                                  |Initial threshold for p-value clstering                                     |
|otherPThresh          |0.05                                                  |Every p-value for a given test must exceed this p-value down the other test |
|mouseMatrixDefinition |narrow                                                |How we define mice lineages                                                 |
|nR1                   |12                                                    |If there are nR1 SNPs in nD1 bp, remove all SNPs                            |
|nD1                   |50                                                    |See above                                                                   |
|nR2                   |7                                                     |For a lineage, if there are ge this many SNPs in nD2 bp, remove all SNPs    |
|nD2                   |50                                                    |See above                                                                   |
|removeNum             |0                                                     |How many samples are allowed to be uncallable                               |
|contingencyComparison |Same CpG, GC Content                                  |To what do we compare motifs                                                |

---


##Methods
###SNP Filtering

SNPs are filtered if they have any missingness among the lineages, do not agree with the species tree, are multi-allelic, or are heterozygous among homozygous animals. 

Subsequently, SNPs are removed if they are too close together. This is the last step in the procedure, ie after removing SNPs for reasons listed above. I removed any SNPs if there were nR1 SNPs within nD1 bases, and, following this, I removed lineage specific SNPs down any lineage if there were nR2 SNPs within nD2 bases. For example, if nR1 was 10 and nD1 was 50, and there was a cluster of 14 SNPs within 50 bases, all of the offending SNPs were removed. 

To get a sense of how many SNPs this removed for given parameter settings, I checked how many SNPs were filtered for a range of parameter settings for the smallest chromosome. I also compared this against expectations. To get an expectation, I simulated a pseudo-chromosome of results. I calculated the expected branch lengths of each lineage given the emprical data to this point, ie lineage 1 has 1 percent divergence against MRCA of the set of all lineages, lineage 2 has 0.5 percent divergence against MRCA, etc. Then, I decided whether each base was mutated according to the total branch length of the tree, and then, given a mutation, what branch it occured on with probabilities equal to each lineages share of the total tree length. 



```
## Warning: cannot open compressed file
## '/Net/dense/data/itch/rwdavies/motifLoss/afihuman/input_A_2015_01_22/forFilteringPlot.RData',
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


### motifSuperResults description
Results for clustering are contained in an RData object, one seperately for each test that has been performed. The files are a series of lists of lists, with the RData object being named motifSuperResults.

List level 1 is by lineage name. So motifSuperResults[[1]] are the results for the first lineage.

Within lineage, results are sorted by repeat background. For example, there is usually results available for the non-repeat background under motifSuperResults[[1]]$nonRepeat.

Within lineage and within repeat there are the following variables


|Variable name     |Description of Variable                                                                                                                                                      |
|:-----------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|hashSigR          |Matrix with rows equal to each Bonferonni significant motif and K columns. 0=A, 1=C, 2=G, 3=T                                                                                |
|pSigR             |Matrix where each row corresponds to a motif, and the columns are the different lineages, in the same order as the included linName variable and/or names(motifSuperResults) |
|pSigOR            |Same as above but the original p-value for the test in each lineage                                                                                                          |
|orSigR            |Same as above but odds ratio for that test                                                                                                                                   |
|motifClusters     |For each Bonferonni significant motif, what cluster was it eventually included in                                                                                            |
|clusteredResults  |Deprecated                                                                                                                                                                   |
|clusteringResults |List of length equal to the number of significant clusters, containing the result of clustering                                                                              |

So for example, length(motifSuperResults[[1]]$nonRepeat) gives you the number of clusters found for that lineage for the repeat background for that test. Entries for the first result are given in motifSuperResults[[1]]$nonRepeat$clusteringResults[[1]] and contain the following categories


|Variable name       |Description of Variable                                                                                                                                                                                                                                                                                                                                                   |
|:-------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|matchesN            |For each motif (Bonferonni significant or not) in that cluster, the K length integer based motif (0=A, 1=C, 2=G, 3=T)                                                                                                                                                                                                                                                     |
|whereN              |In the alignment, for the left edge, where it aligns. For instance, the original motif is at position 0, while anything aligned over one to the left is at -1                                                                                                                                                                                                             |
|namesN              |For each motif in the cluster, the hash of the 0-based integer motif. In R, if x is the K length integer motif, then this is sum(x*4^(0:(K-1)))                                                                                                                                                                                                                           |
|whereItMatchesToN   |Internal variable relating motifs to subsetted matrix                                                                                                                                                                                                                                                                                                                     |
|textOut             |Aligned textual representation of output                                                                                                                                                                                                                                                                                                                                  |
|sum                 |Take the matrix sumNum described below and set each cell with 0 to pwmMinimum. Then divide each column by the column sum                                                                                                                                                                                                                                                  |
|sumNum              |Summed across each motif in the cluster the sum of its sequence multiplied by its OR minus 1                                                                                                                                                                                                                                                                              |
|fromWhereN          |At what level of the recursive addition procedure was this motif added in (0=first Bonferonni significant seed motif, 1=second layer, etc)                                                                                                                                                                                                                                |
|atgc                |Itself a list with two entries, loss or gain. Each one of loss or gain contains another list, with each entry corresponding to a type of base change, like A/T->A/T. Each of these also contains a list of length 2, with the first entry being the numerator, the number of changed bases, and the second being the denominator reflecting the number of available bases |
|motifListF          |Motif hashes sorted from most significant p-value to least significant p-value                                                                                                                                                                                                                                                                                            |
|motifLossPositions  |Itself a list with two entries, loss or gain. Each one contains, for either loss or gain, the positions on each of the chromosomes of the loss or gain of any motif in the cluster                                                                                                                                                                                        |
|correlationMatrices |If defined, information on correlation between this cluster and broad scale rate maps                                                                                                                                                                                                                                                                                     |
|motifPValues        |P-values for the tests, ordered from most significant down this test, to least signicicant                                                                                                                                                                                                                                                                                |




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


|        |MACAQUE |AHCGO |ORANGUTAN |AHCG   |GORILLA |AHC   |CHIMP   |AHN     |AHD     |NEAN  |
|:-------|:-------|:-----|:---------|:------|:-------|:-----|:-------|:-------|:-------|:-----|
|gainat  |0 (0)   |0 (0) |0 (0)     |0 (0)  |0 (0)   |0 (0) |32 (27) |0 (0)   |0 (0)   |1 (1) |
|gainlin |1 (1)   |7 (6) |5 (4)     |0 (0)  |1 (1)   |0 (0) |15 (6)  |13 (4)  |11 (5)  |0 (0) |
|lossat  |22 (11) |2 (1) |12 (5)    |16 (3) |9 (4)   |0 (0) |50 (33) |80 (7)  |77 (7)  |0 (0) |
|losslin |50 (24) |4 (1) |139 (27)  |59 (8) |7 (4)   |1 (1) |54 (12) |138 (8) |134 (9) |0 (0) |
MACAQUE


|        |nonRepeat |L1MA1 |Tigger3b |L1MB3 |Tigger3a |L1MA2 |L1MB2 |L1PA7 |L1PA8 |MLT1B |MSTD  |THE1B |Tigger1 |
|:-------|:---------|:-----|:--------|:-----|:--------|:-----|:-----|:-----|:-----|:-----|:-----|:-----|:-------|
|gainlin |1 (1)     |0 (0) |0 (0)    |0 (0) |0 (0)    |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)   |
|lossat  |9 (4)     |7 (2) |1 (1)    |2 (2) |0 (0)    |0 (0) |1 (1) |0 (0) |2 (1) |0 (0) |0 (0) |0 (0) |0 (0)   |
|losslin |24 (11)   |0 (0) |6 (1)    |3 (2) |4 (2)    |2 (1) |1 (1) |2 (2) |0 (0) |2 (1) |2 (1) |2 (1) |2 (1)   |
AHCGO


|        |nonRepeat |L1PA8 |L1PB1 |MSTA  |THE1B |THE1D |
|:-------|:---------|:-----|:-----|:-----|:-----|:-----|
|gainlin |0 (0)     |3 (2) |1 (1) |1 (1) |1 (1) |1 (1) |
|lossat  |2 (1)     |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |
|losslin |4 (1)     |0 (0) |0 (0) |0 (0) |0 (0) |0 (0) |
ORANGUTAN


|        |nonRepeat |MIR    |MIRb   |L1PA13 |MIRc  |MLT1B |THE1D |MLT1D |
|:-------|:---------|:------|:------|:------|:-----|:-----|:-----|:-----|
|gainlin |0 (0)     |2 (2)  |3 (2)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |
|lossat  |8 (4)     |4 (1)  |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0) |0 (0) |
|losslin |95 (17)   |13 (1) |12 (1) |9 (1)  |5 (3) |2 (1) |2 (2) |1 (1) |
AHCG


|        |nonRepeat |
|:-------|:---------|
|lossat  |16 (3)    |
|losslin |59 (8)    |
GORILLA


|        |L1PA13 |nonRepeat |L1MB3 |
|:-------|:------|:---------|:-----|
|gainlin |1 (1)  |0 (0)     |0 (0) |
|lossat  |6 (2)  |3 (2)     |0 (0) |
|losslin |2 (1)  |3 (2)     |2 (1) |
AHC


|        |L1MB5 |
|:-------|:-----|
|losslin |1 (1) |
CHIMP


|        |nonRepeat |MLT1C  |L2c   |L1PA13 |MLT2A1 |MIR   |THE1B |
|:-------|:---------|:------|:-----|:------|:------|:-----|:-----|
|gainat  |32 (27)   |0 (0)  |0 (0) |0 (0)  |0 (0)  |0 (0) |0 (0) |
|gainlin |14 (5)    |1 (1)  |0 (0) |0 (0)  |0 (0)  |0 (0) |0 (0) |
|lossat  |34 (25)   |6 (1)  |4 (3) |2 (1)  |2 (1)  |1 (1) |1 (1) |
|losslin |38 (9)    |15 (2) |0 (0) |1 (1)  |0 (0)  |0 (0) |0 (0) |
AHN


|        |nonRepeat |MSTA   |MER20  |MSTB   |L1PREC2 |LTR39 |LTR8  |MER5A |
|:-------|:---------|:------|:------|:------|:-------|:-----|:-----|:-----|
|gainlin |0 (0)     |7 (3)  |6 (1)  |0 (0)  |0 (0)   |0 (0) |0 (0) |0 (0) |
|lossat  |45 (3)    |13 (1) |8 (1)  |10 (1) |4 (1)   |0 (0) |0 (0) |0 (0) |
|losslin |83 (2)    |19 (1) |20 (1) |9 (1)  |0 (0)   |3 (1) |2 (1) |2 (1) |
AHD


|        |nonRepeat |MSTA   |MER20  |MSTB   |L1PREC2 |LTR39 |LTR8  |MER5A |
|:-------|:---------|:------|:------|:------|:-------|:-----|:-----|:-----|
|gainlin |0 (0)     |6 (3)  |5 (2)  |0 (0)  |0 (0)   |0 (0) |0 (0) |0 (0) |
|lossat  |44 (3)    |13 (1) |8 (1)  |10 (1) |2 (1)   |0 (0) |0 (0) |0 (0) |
|losslin |82 (3)    |18 (1) |19 (1) |9 (1)  |0 (0)   |2 (1) |2 (1) |2 (1) |
HUMAN(NEAN)
No significant results

HUMAN(DENI)
No significant results

NEAN


|       |nonRepeat |
|:------|:---------|
|gainat |1 (1)     |
DENI
No significant results

---


## Features of significant motifs

Test: losslin 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.7| 3.1|  9.0| 17.8| 23.3| 21.3| 13.9| 7.1| 2.9| 0.7|
|significant     | 0.0| 0.0| 2.4| 17.3| 26.0| 28.4| 18.2|  5.9| 1.4| 0.5| 0.0|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 73.2| 25.1| 1.6| 0.1| 0.0|
|significant     | 97.9|  2.1| 0.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 6.1| 53.0| 29.7| 8.6| 2.1| 0.4| 0.1| 0.0| 0.0| 0.0|
|significant     | 7.3| 47.8| 38.8| 5.2| 0.9| 0.0| 0.0| 0.0| 0.0| 0.0|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.8| 26.9| 71.3|
|significant     | 0.0| 1.7| 24.3| 74.0|
Test: lossat 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.5| 2.6|  8.5| 17.0| 23.1| 22.2| 14.9| 7.3| 2.9| 0.8|
|significant     | 0.0| 1.6| 0.5| 11.5| 25.1| 28.4| 21.9|  8.7| 1.6| 0.5| 0.0|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 75.9| 23.6| 0.5| 0.0| 0.0|
|significant     | 94.5|  5.5| 0.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 6.0| 52.4| 30.0| 8.9| 2.2| 0.4| 0.0| 0.0| 0.0| 0.0|
|significant     | 6.6| 48.1| 39.9| 5.5| 0.0| 0.0| 0.0| 0.0| 0.0| 0.0|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.9| 26.8| 71.3|
|significant     | 0.0| 1.1| 22.4| 76.5|
Test: gainlin 
AT content 


|                |   0|   1|    2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|----:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.7|  3.5|  9.9| 19.4| 24.7| 20.9| 12.4| 5.7| 2.1| 0.5|
|significant     | 0.0| 9.5| 14.3| 21.4|  9.5| 16.7|  9.5|  9.5| 4.8| 4.8| 0.0|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 65.9| 31.1| 2.9| 0.1| 0.0|
|significant     | 88.1| 11.9| 0.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|    4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|----:|---:|---:|---:|---:|---:|---:|
|not significant | 6.5| 55.0| 28.6|  7.6| 1.8| 0.4| 0.1| 0.0| 0.0| 0.0|
|significant     | 4.8| 31.0| 16.7| 16.7| 2.4| 9.5| 4.8| 9.5| 4.8| 0.0|
Nucleotide diversity 


|                |   1|    2|    3|    4|
|:---------------|---:|----:|----:|----:|
|not significant | 0.0|  1.5| 24.1| 74.4|
|significant     | 0.0| 26.2| 16.7| 57.1|
Test: gainat 
AT content 


|                |   0|   1|    2|   3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|----:|---:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.1| 0.6|  3.1| 9.6| 19.5| 25.2| 21.5| 12.6| 5.5| 1.9| 0.5|
|significant     | 0.0| 9.1| 12.1| 9.1| 30.3| 18.2|  6.1|  3.0| 9.1| 0.0| 3.0|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 64.2| 34.2| 1.6| 0.0| 0.0|
|significant     | 57.6| 36.4| 6.1| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|    4|   5|   6|   7|    8|   9|  10|
|:---------------|---:|----:|----:|----:|---:|---:|---:|----:|---:|---:|
|not significant | 6.6| 55.2| 28.3|  7.6| 1.8| 0.4| 0.1|  0.0| 0.0| 0.0|
|significant     | 6.1| 33.3| 18.2| 12.1| 3.0| 6.1| 3.0| 12.1| 3.0| 3.0|
Nucleotide diversity 


|                |   1|    2|    3|    4|
|:---------------|---:|----:|----:|----:|
|not significant | 0.0|  1.4| 22.9| 75.7|
|significant     | 3.0| 15.2| 27.3| 54.5|
---





## PWK results at CAST top results

Only for the mouse analysis


---




## AT to GC changes along lineages

For each lineage, stratify into percents of the bases that are the same from its ancestor, and those which changed from the ancestor into this lineage, what they are towards


|            |   sameA|   sameC|   sameG|   sameT| A/T->A/T| C/G->A/T| A/T->C/G| C/G->C/G|
|:-----------|-------:|-------:|-------:|-------:|--------:|--------:|--------:|--------:|
|MACAQUE     | 29.2122| 19.5588| 19.5762| 29.2657|   0.1674|   0.9999|   1.0382|   0.1815|
|AHCGO       | 29.6090| 19.9238| 19.9421| 29.6616|   0.0561|   0.3933|   0.3568|   0.0573|
|ORANGUTAN   | 29.5558| 19.8648| 19.8827| 29.6088|   0.0701|   0.4759|   0.4489|   0.0930|
|AHCG        | 29.6877| 20.0155| 20.0339| 29.7404|   0.0338|   0.2242|   0.2217|   0.0429|
|GORILLA     | 29.6810| 20.0223| 20.0405| 29.7338|   0.0331|   0.2053|   0.2357|   0.0484|
|AHC         | 29.7840| 20.1214| 20.1394| 29.8365|   0.0075|   0.0455|   0.0555|   0.0102|
|CHIMP       | 29.7146| 20.0534| 20.0718| 29.7670|   0.0282|   0.1557|   0.1737|   0.0356|
|AHN         | 29.7290| 20.0683| 20.0863| 29.7814|   0.0226|   0.1327|   0.1504|   0.0292|
|AHD         | 29.7293| 20.0685| 20.0865| 29.7817|   0.0226|   0.1324|   0.1499|   0.0291|
|HUMAN(NEAN) | 29.8034| 20.1374| 20.1555| 29.8556|   0.0035|   0.0192|   0.0210|   0.0044|
|HUMAN(DENI) | 29.8032| 20.1372| 20.1553| 29.8553|   0.0036|   0.0195|   0.0215|   0.0045|
|NEAN        | 29.8006| 20.1327| 20.1505| 29.8529|   0.0046|   0.0277|   0.0255|   0.0056|
|DENI        | 29.7999| 20.1308| 20.1487| 29.8520|   0.0048|   0.0310|   0.0269|   0.0059|
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
|MACAQUE     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/MACAQUE/)     |
|AHCGO       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/AHCGO/)       |
|ORANGUTAN   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/ORANGUTAN/)   |
|AHCG        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/AHCG/)        |
|GORILLA     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/GORILLA/)     |
|AHC         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/AHC/)         |
|CHIMP       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/CHIMP/)       |
|AHN         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/AHN/)         |
|AHD         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/AHD/)         |
|HUMAN(NEAN) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/HUMAN_NEAN_/) |
|HUMAN(DENI) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/HUMAN_DENI_/) |
|NEAN        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/NEAN/)        |
|DENI        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/losslin/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/lossat/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainlin/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifPvalues/gainat/DENI/)        |

---

## Motif Cluster RData


|losslin                                                                                                                                           |lossat                                                                                                                                           |gainlin                                                                                                                                           |gainat                                                                                                                                           |
|:-------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------|
|[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifSuperResults/losslin.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifSuperResults/lossat.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifSuperResults/gainlin.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_01_22/motifSuperResults/gainat.K10.motifSuperResults.RData) |





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
