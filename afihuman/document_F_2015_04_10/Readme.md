#Hotspot death
##Species: afihuman
##Run version: 2015_04_10
---



##Changelog
Date refers to first version with changes. If there is a plus after, then all subsequent versions have the change
* **2015_04_06** Bugfix - for the human results, fix issue where two SNPs are nearby when one was supposed to enter the immediate previous lineage
* **2015_04_06** Re-work the way expectations are calcualted for ancestral rate maps
* **2015_01_22** Bigfix - prevent motif where a motif and its converse could both enter motif cluster at the same time
* **2015_01_22** Add curve fitting to AT to GC plotting
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
|nATtoGCrepeats        |1000                                                  |How many bootstrap repeats should we do for curve fitting                   |

---


##Methods
###SNP Filtering

SNPs are filtered if they have any missingness among the lineages, do not agree with the species tree, are multi-allelic, or are heterozygous among homozygous animals. 

Subsequently, SNPs are removed if they are too close together. This is the last step in the procedure, ie after removing SNPs for reasons listed above. I removed any SNPs if there were nR1 SNPs within nD1 bases, and, following this, I removed lineage specific SNPs down any lineage if there were nR2 SNPs within nD2 bases. For example, if nR1 was 10 and nD1 was 50, and there was a cluster of 14 SNPs within 50 bases, all of the offending SNPs were removed. 

To get a sense of how many SNPs this removed for given parameter settings, I checked how many SNPs were filtered for a range of parameter settings for the smallest chromosome. I also compared this against expectations. To get an expectation, I simulated a pseudo-chromosome of results. I calculated the expected branch lengths of each lineage given the emprical data to this point, ie lineage 1 has 1 percent divergence against MRCA of the set of all lineages, lineage 2 has 0.5 percent divergence against MRCA, etc. Then, I decided whether each base was mutated according to the total branch length of the tree, and then, given a mutation, what branch it occured on with probabilities equal to each lineages share of the total tree length. 



```
## Warning in readChar(con, 5L, useBytes = TRUE): cannot open compressed file
## '/Net/dense/data/itch/rwdavies/motifLoss/afihuman/input_A_2015_04_10/forFilteringPlot.RData',
## probable reason 'No such file or directory'
```

```
## Error in readChar(con, 5L, useBytes = TRUE): cannot open the connection
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

These plots also feature a PWM for the forward and reverse forms of the motif, as well as a series of line plots which show the number of motifs and their p-values for the motifs in the cluster. There is no particular order in which test is placed where vertically. Each line shows p-values for the test under consideration, with grey lines linking the same motif (motifs with undefined p-values on the other two tests are omitted from the plots for those tests and are not linked). These are stratified into those which meet genome-wide significance to the right of the red line, those which are between the initial clustering p-value threshold and the Bonferonni threshold in the middle, and those which do not meet the initial clustering p-value threhsold on the left. Numbers of motifs falling into each category are given as well.


###AT to GC plot - curve fitting
We fitted a curve to estimate parameters relating to the increa
se in AT to GC fixation near ancient hotspot death events.



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


|    Total|  Pass QC|  Fail QC| Pass QC Non Repeat|
|--------:|--------:|--------:|------------------:|
| 2.881033| 1.107748| 1.773285|          0.6828016|

Number of Derived Mutations down a specific lineage


|MACAQUE    |AHCGO      |ORANGUTAN  |AHCG       |GORILLA    |AHC        |CHIMP      |AHN        |AHD        |HUMAN(NEAN) |HUMAN(DENI) |NEAN       |DENI       |
|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:----------|:-----------|:-----------|:----------|:----------|
|26,621,100 | 9,748,431 |12,008,463 | 5,789,106 | 5,754,222 | 1,306,032 | 4,296,644 | 3,620,908 | 3,610,379 |   520,460  |   531,019  |   748,324 |   811,296 |

Branch length as percent of alignable genome


| MACAQUE| AHCGO| ORANGUTAN|  AHCG| GORILLA|   AHC| CHIMP|   AHN|   AHD| HUMAN(NEAN)| HUMAN(DENI)|  NEAN|  DENI|
|-------:|-----:|---------:|-----:|-------:|-----:|-----:|-----:|-----:|-----------:|-----------:|-----:|-----:|
|   2.403|  0.88|     1.084| 0.523|   0.519| 0.118| 0.388| 0.327| 0.326|       0.047|       0.048| 0.068| 0.073|

Branch length compared to ancestral of all lineages in SNPs


|DENI       |NEAN       |HUMAN(DENI) |HUMAN(NEAN) |CHIMP      |GORILLA    |ORANGUTAN  |MACAQUE    |
|:----------|:----------|:-----------|:-----------|:----------|:----------|:----------|:----------|
|24,886,152 |24,823,180 |24,605,875  |24,595,316  |21,140,213 |21,291,759 |21,756,894 |26,621,100 |

Branch length compared to ancestral as percent of alignable genome


|  DENI|  NEAN| HUMAN(DENI)| HUMAN(NEAN)| CHIMP| GORILLA| ORANGUTAN| MACAQUE|
|-----:|-----:|-----------:|-----------:|-----:|-------:|---------:|-------:|
| 2.247| 2.241|       2.221|        2.22| 1.908|   1.922|     1.964|   2.403|


---

## Number of significant motifs per lineage per test

Note that the following numbers are only for motifs that have been clustered

Number of motifs (clusters) per test and lineage


|        |MACAQUE |AHCGO |ORANGUTAN |AHCG   |GORILLA |AHC   |CHIMP   |AHN      |AHD      |NEAN  |
|:-------|:-------|:-----|:---------|:------|:-------|:-----|:-------|:--------|:--------|:-----|
|gainat  |0 (0)   |0 (0) |1 (1)     |0 (0)  |0 (0)   |0 (0) |28 (22) |0 (0)    |0 (0)    |1 (1) |
|gainlin |1 (1)   |7 (6) |5 (4)     |0 (0)  |0 (0)   |0 (0) |15 (5)  |8 (3)    |6 (2)    |0 (0) |
|lossat  |21 (10) |2 (1) |10 (5)    |16 (3) |6 (4)   |0 (0) |43 (30) |66 (6)   |61 (5)   |0 (0) |
|losslin |41 (20) |4 (1) |136 (23)  |56 (7) |7 (4)   |2 (1) |54 (10) |112 (10) |110 (10) |0 (0) |

MACAQUE
AHCGO
ORANGUTAN
AHCG
GORILLA
AHC
CHIMP
AHN
AHD
HUMAN(NEAN)
No significant results

HUMAN(DENI)
No significant results

NEAN
DENI
No significant results

---


## Features of significant motifs

Test: losslin 
AT content 
Number of CpGs 
Maximum run length 
Nucleotide diversity 
Test: lossat 
AT content 
Number of CpGs 
Maximum run length 
Nucleotide diversity 
Test: gainlin 
AT content 
Number of CpGs 
Maximum run length 
Nucleotide diversity 
Test: gainat 
AT content 
Number of CpGs 
Maximum run length 
Nucleotide diversity 
---





## PWK results at CAST top results

Only for the mouse analysis


---




## AT to GC changes along lineages

For each lineage, stratify into percents of the bases that are the same from its ancestor, and those which changed from the ancestor into this lineage, what they are towards


|            |    sameA|    sameC|    sameG|    sameT|  A/T->A/T|  C/G->A/T|  A/T->C/G|  C/G->C/G|
|:-----------|--------:|--------:|--------:|--------:|---------:|---------:|---------:|---------:|
|MACAQUE     | 29.21216| 19.55884| 19.57625| 29.26574| 0.1674355| 0.9998602| 1.0382078| 0.1815095|
|AHCGO       | 29.60904| 19.92377| 19.94209| 29.66162| 0.0560564| 0.3933122| 0.3568227| 0.0572931|
|ORANGUTAN   | 29.57329| 19.84609| 19.86499| 29.62781| 0.0700564| 0.4759290| 0.4488664| 0.0929604|
|AHCG        | 29.70513| 19.99679| 20.01610| 29.75944| 0.0337619| 0.2242101| 0.2216985| 0.0428709|
|GORILLA     | 29.69879| 20.00176| 20.02205| 29.75498| 0.0330742| 0.2052810| 0.2356937| 0.0483757|
|AHC         | 29.80188| 20.10081| 20.12095| 29.85766| 0.0075449| 0.0454995| 0.0554616| 0.0102062|
|CHIMP       | 29.72720| 20.03764| 20.05855| 29.78350| 0.0282302| 0.1556489| 0.1736525| 0.0355875|
|AHN         | 29.74162| 20.05253| 20.07302| 29.79791| 0.0226262| 0.1327213| 0.1504254| 0.0291561|
|AHD         | 29.74190| 20.05275| 20.07323| 29.79818| 0.0225650| 0.1323715| 0.1499406| 0.0290742|
|HUMAN(NEAN) | 29.80677| 20.13019| 20.15136| 29.86359| 0.0035234| 0.0191928| 0.0209931| 0.0043843|
|HUMAN(DENI) | 29.80655| 20.12990| 20.15107| 29.86340| 0.0035848| 0.0195490| 0.0214767| 0.0044656|
|NEAN        | 29.80369| 20.12477| 20.14571| 29.86056| 0.0045975| 0.0290297| 0.0260244| 0.0056246|
|DENI        | 29.80295| 20.12273| 20.14372| 29.85973| 0.0048617| 0.0326550| 0.0274639| 0.0058923|
---




## Giant AT to GC plot

All clustered AT to GC plots collapsed together for the loss lineage test

![plot of chunk atToGCAll](figure/atToGCAll-1.png) 



---

## Correlation between some top motifs


This plot shows correlation of some pre-selected top motifs on different lineages and backgrounds with each other and with 3 linkage-disequilibrium based maps, at fine and broad scales.

```
## Warning in mclapply(chrlist, function(chr, iLin) {: all scheduled cores
## encountered errors in user code
```

```
## Error in x$rateComp: $ operator is invalid for atomic vectors
```

```
## Error in unlist(corrMat): object 'corrMat' not found
```

Correlation at 5 megabase scale between LD rate maps and hotspot death estimates

```
## Error in is.data.frame(x): object 'corrMat' not found
```











---

## Links to pages of significant motifs


|            |losslin                                                                                                                       |lossat                                                                                                                       |gainlin                                                                                                                       |gainat                                                                                                                       |
|:-----------|:-----------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------|
|MACAQUE     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/MACAQUE/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/MACAQUE/)     |
|AHCGO       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/AHCGO/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/AHCGO/)       |
|ORANGUTAN   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/ORANGUTAN/)   |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/ORANGUTAN/)   |
|AHCG        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/AHCG/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/AHCG/)        |
|GORILLA     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/GORILLA/)     |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/GORILLA/)     |
|AHC         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/AHC/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/AHC/)         |
|CHIMP       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/CHIMP/)       |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/CHIMP/)       |
|AHN         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/AHN/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/AHN/)         |
|AHD         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/AHD/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/AHD/)         |
|HUMAN(NEAN) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/HUMAN_NEAN_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/HUMAN_NEAN_/) |
|HUMAN(DENI) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/HUMAN_DENI_/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/HUMAN_DENI_/) |
|NEAN        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/NEAN/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/NEAN/)        |
|DENI        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/losslin/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/lossat/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainlin/DENI/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifPvalues/gainat/DENI/)        |

---

## Motif Cluster RData


|losslin                                                                                                                                           |lossat                                                                                                                                           |gainlin                                                                                                                                           |gainat                                                                                                                                           |
|:-------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------|
|[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifSuperResults/losslin.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifSuperResults/lossat.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifSuperResults/gainlin.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/afihuman/document_F_2015_04_10/motifSuperResults/gainat.K10.motifSuperResults.RData) |



---

## AT to GC curve fitting

Results from curve fitting bootstrapping procedure


|      |       Mean| CI Lower Bound| CI Upper Bound|
|:-----|----------:|--------------:|--------------:|
|Ratio |   5.461815|       4.352662|       6.769241|
|Exp 1 |  64.101355|      60.064026|      67.132198|
|Exp 2 | 618.429387|     465.811872|     937.036276|
|Exp 3 |  64.101455|      60.064069|      67.132440|

Comparison of various models for fitting

![plot of chunk atToGCcomparison](figure/atToGCcomparison-1.png) 



































<!--

## all significant motifs




---


## QQ plots - seperate

![plot of chunk pValuesQQPlots](figure/pValuesQQPlots-1.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots-2.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots-3.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots-4.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots-5.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots-6.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots-7.png) ![plot of chunk pValuesQQPlots](figure/pValuesQQPlots-8.png) 
---




## Compare p-values between methods

![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods-1.png) ![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods-2.png) ![plot of chunk pvaluesBetweenMethods](figure/pvaluesBetweenMethods-3.png) 
---


## Compare p-values between lineages within a method

![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-1.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-2.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-3.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-4.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-5.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-6.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-7.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-8.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-9.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-10.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-11.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-12.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-13.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-14.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-15.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-16.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-17.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-18.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-19.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-20.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-21.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-22.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-23.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-24.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-25.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-26.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-27.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-28.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-29.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-30.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-31.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-32.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-33.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-34.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-35.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-36.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-37.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-38.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-39.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-40.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-41.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-42.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-43.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-44.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-45.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-46.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-47.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-48.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-49.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-50.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-51.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage-52.png) 
---





-->
