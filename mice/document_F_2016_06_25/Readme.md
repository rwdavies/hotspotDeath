#Hotspot death
##Species: mice
##Run version: 2016_06_25
---



##Changelog
Date refers to first version with changes. If there is a plus after, then all subsequent versions have the change
* **2015_04_10** Bugfix - Get the OR right in the PWM summation
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
|mouseFilter           |allPlus2GATKArrayNotFilteredAnnot295.0                |What folder used for mice                                                   |
|mrle                  |10                                                    |Maximum run length less than or equal to                                    |
|ndge                  |0                                                     |Nuclotide diversity greater than or equal to                                |
|pwmMinimum            |0.1                                                   |When a PWM has 0 entry, what to reset the 0 values to                       |
|Klist                 |10                                                    |Range of K to use                                                           |
|gcW1                  |100                                                   |Range to test for GC increase                                               |
|gcW2                  |5000                                                  |Range to plot GC increases                                                  |
|gcW3                  |10                                                    |Smoothing window for AT to GC plots                                         |
|gcW4                  |10                                                    |Range to plot recombination around loss positions                           |
|ctge                  |10                                                    |For a p-value to be generated, each cell must have at least this number     |
|rgte                  |50                                                    |For a row to count, there must be this many entries                         |
|testingNames          |losslin,lossat,gainlin,gainat                         |(Short) names of the tests we will perform                                  |
|plottingNames         |Loss Lineage,Loss AT to GC,Gain Lineage,Gain AT to GC |Plotting names of the tests we will perform                                 |
|nTests                |4                                                     |Number of tests to be performed                                             |
|globalPThresh         |9.52743902439024e-08                                  |P-value threshold (if NA use total number of tests performed)               |
|pThresh               |0.000196850393700787                                  |Initial threshold for p-value clstering                                     |
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
## Warning in readChar(con, 5L, useBytes = TRUE): cannot open compressed
## file '/data/smew1/rdavies/wildmice/motifLoss/input_A_2016_06_25/
## forFilteringPlot.RData', probable reason 'No such file or directory'
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

Next, taking care to get both the correct strand as well as the position of the motif within the cluster of motifs correct, we catalogued both the position and base composition of any changes within a neighbourhood of 5000 bases. By summing across all loss instances of motifs in the cluster, and normalizing to the local sequence context, we could plot any type of ancestral to derived base change. Smoothing was done over 21 bases, ie taking the value at the flanking bases and over the prior and aft 10 bases. 

These plots also feature a PWM for the forward and reverse forms of the motif, as well as a series of line plots which show the number of motifs and their p-values for the motifs in the cluster. There is no particular order in which test is placed where vertically. Each line shows p-values for the test under consideration, with grey lines linking the same motif (motifs with undefined p-values on the other two tests are omitted from the plots for those tests and are not linked). These are stratified into those which meet genome-wide significance to the right of the red line, those which are between the initial clustering p-value threshold and the Bonferonni threshold in the middle, and those which do not meet the initial clustering p-value threhsold on the left. Numbers of motifs falling into each category are given as well.


###AT to GC plot - curve fitting
We fitted a curve to estimate parameters relating to the increa
se in AT to GC fixation near ancient hotspot death events. More details are available in my thesis


###Ancestral map construction

First, we calculate the expectation for a cluster with no Prdm9 affinity. For rate of loss, this means we first calculate the rate of loss of all motifs in the cluster as the number of times these motifs were lost divided by their ancestral count. We then calculate the rate of loss for all motifs in a similar way. The expectation for the rate of loss for a given lineage is taken as the rate of loss over all motifs multiplied by the median of the rate of loss of these motifs divided by the rate of loss of all motifs, without including the current lineage. 


, both for what proportion of motifs we'd expect to lose if motifs were lost at neutrality, and for what AT to GC rate we'd expect to see given neutrality. 



Next, for a given window size, over all intervals in the genome, we count both the number of motif losses in a cluster and the number of ancestral motif counts. Both of these can be either filtered or unfiltered, ie distinct loss events must be a certain distance away; used here is the cluster width. 



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


|    Total|  Pass QC|   Fail QC| Pass QC Non Repeat|
|--------:|--------:|---------:|------------------:|
| 2.472342| 1.644911| 0.8274314|            1.11376|

Number of Derived Mutations down a specific lineage


|FAM        |AMS       |Spretus    |AM        |PWKPhJ    |CASTEiJ   |CASTEiJ.PWKPhJ |WSBEiJ    |WSBEiJ.PWKPhJ |WSBEiJ.CASTEiJ |
|:----------|:---------|:----------|:---------|:---------|:---------|:--------------|:---------|:-------------|:--------------|
|17,913,173 |8,758,108 |11,716,370 |4,692,855 |3,710,202 |3,560,700 |1,381,827      |3,604,469 |1,228,438     |1,226,785      |

Branch length as percent of alignable genome


|   FAM|   AMS| Spretus|    AM| PWKPhJ| CASTEiJ| CASTEiJ.PWKPhJ| WSBEiJ| WSBEiJ.PWKPhJ| WSBEiJ.CASTEiJ|
|-----:|-----:|-------:|-----:|------:|-------:|--------------:|------:|-------------:|--------------:|
| 1.089| 0.532|   0.712| 0.285|  0.226|   0.216|          0.084|  0.219|         0.075|          0.075|

Branch length compared to ancestral of all lineages in SNPs


|WSBEiJ.CASTEiJ |WSBEiJ.PWKPhJ |WSBEiJ     |CASTEiJ.PWKPhJ |CASTEiJ    |PWKPhJ     |Spretus    |FAM        |
|:--------------|:-------------|:----------|:--------------|:----------|:----------|:----------|:----------|
|14,677,748     |14,679,401    |17,055,432 |14,832,790     |17,011,663 |17,161,165 |20,474,478 |17,913,173 |

Branch length compared to ancestral as percent of alignable genome


| WSBEiJ.CASTEiJ| WSBEiJ.PWKPhJ| WSBEiJ| CASTEiJ.PWKPhJ| CASTEiJ| PWKPhJ| Spretus|   FAM|
|--------------:|-------------:|------:|--------------:|-------:|------:|-------:|-----:|
|          0.892|         0.892|  1.037|          0.902|   1.034|  1.043|   1.245| 1.089|

Aligned Genome (Gbp)
Number of Derived Mutations down a specific lineage

```
## Error in (ncol(clusterList2) - 1):ncol(clusterList2): argument of length 0
```


---

## Number of significant motifs per lineage per test

Note that the following numbers are only for motifs that have been clustered

Number of motifs (clusters) per test and lineage
FAM
No significant results

AMS
No significant results

Spretus
No significant results

AM
No significant results

PWKPhJ
No significant results

CASTEiJ
No significant results

CASTEiJ.PWKPhJ
No significant results

WSBEiJ
No significant results

WSBEiJ.PWKPhJ
No significant results

WSBEiJ.CASTEiJ
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

Other studies have found that M. m. musculus has a PRDM9 allele similar to the CAST motif, but while we found a denovo CAST motif similar to published work, here we found no de novo PWK motifs. Here, we look at the p-values for the motifs in the top CAST cluster in the other strains
Test: losslin
Test: lossat
Test: gainlin
Test: gainat
---




## AT to GC changes along lineages

For each lineage, stratify into percents of the bases that are the same from its ancestor, and those which changed from the ancestor into this lineage, what they are towards


|               |    sameA|    sameC|    sameG|    sameT|  A/T->A/T|  C/G->A/T|  A/T->C/G|  C/G->C/G|
|:--------------|--------:|--------:|--------:|--------:|---------:|---------:|---------:|---------:|
|FAM            | 28.81810| 20.63734| 20.63797| 28.83311| 0.0962757| 0.4882479| 0.4256704| 0.0632856|
|AMS            | 28.93921| 20.78703| 20.78809| 28.95425| 0.0452984| 0.2184430| 0.2344067| 0.0332787|
|Spretus        | 28.92919| 20.71307| 20.71416| 28.94436| 0.0635006| 0.3719097| 0.2201518| 0.0436727|
|AM             | 28.99745| 20.85292| 20.85398| 29.01254| 0.0239615| 0.1177884| 0.1232413| 0.0181216|
|PWKPhJ         | 29.02730| 20.85377| 20.85506| 29.04222| 0.0186558| 0.1262921| 0.0635632| 0.0131339|
|CASTEiJ        | 29.02653| 20.85952| 20.86060| 29.04159| 0.0178029| 0.1156161| 0.0658222| 0.0125245|
|CASTEiJ.PWKPhJ | 29.05014| 20.90075| 20.90188| 29.06523| 0.0068932| 0.0406926| 0.0294775| 0.0049310|
|WSBEiJ         | 29.02705| 20.85643| 20.85746| 29.04222| 0.0180785| 0.1217935| 0.0643947| 0.0125749|
|WSBEiJ.PWKPhJ  | 29.05274| 20.90258| 20.90359| 29.06778| 0.0060070| 0.0376963| 0.0252163| 0.0043973|
|WSBEiJ.CASTEiJ | 29.05195| 20.90336| 20.90453| 29.06691| 0.0061466| 0.0359333| 0.0267268| 0.0044362|
---




## Giant AT to GC plot

All clustered AT to GC plots collapsed together for the loss lineage test





---

## Correlation between some top motifs


This plot shows correlation of some pre-selected top motifs on different lineages and backgrounds with each other and with 3 linkage-disequilibrium based maps, at fine and broad scales.

```
## Error in eval(expr, envir, enclos): could not find function "getCorrelationsForPlotting"
```

```
## Error in unlist(corrMat): object 'corrMat' not found
```

Correlation at 5 megabase scale between LD rate maps and hotspot death estimates

```
## Error in inherits(x, "list"): object 'corrMat' not found
```



```
## The following plot shows the breakdown chromosome wide
```

```
## Warning in readChar(con, 5L, useBytes = TRUE): cannot open compressed
## file '/data/smew1/rdavies/wildmice/motifLoss/cluster_E_2016_06_25/
## allCorrelations.names.RData', probable reason 'No such file or directory'
```

```
## Error in readChar(con, 5L, useBytes = TRUE): cannot open the connection
```






```
## Warning in readChar(con, 5L, useBytes = TRUE): cannot open compressed
## file '/data/smew1/rdavies/wildmice/motifLoss/cluster_E_2016_06_25/
## allCorrelations.RData', probable reason 'No such file or directory'
```

```
## Error in readChar(con, 5L, useBytes = TRUE): cannot open the connection
```


---

## Links to pages of significant motifs


|               |losslin                                                                                                                      |lossat                                                                                                                      |gainlin                                                                                                                      |gainat                                                                                                                      |
|:--------------|:----------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|
|FAM            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/FAM/)            |
|AMS            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/AMS/)            |
|Spretus        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/Spretus/)        |
|AM             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/AM/)             |
|PWKPhJ         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/PWKPhJ/)         |
|CASTEiJ        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/CASTEiJ/)        |
|CASTEiJ.PWKPhJ |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/CASTEiJ.PWKPhJ/) |
|WSBEiJ         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/WSBEiJ/)         |
|WSBEiJ.PWKPhJ  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/WSBEiJ.PWKPhJ/)  |
|WSBEiJ.CASTEiJ |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/losslin/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/lossat/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainlin/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifPvalues/gainat/WSBEiJ.CASTEiJ/) |

---

## Motif Cluster RData


|losslin                                                                                                                                       |lossat                                                                                                                                       |gainlin                                                                                                                                       |gainat                                                                                                                                       |
|:---------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|
|[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifSuperResults/losslin.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifSuperResults/lossat.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifSuperResults/gainlin.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2016_06_25/motifSuperResults/gainat.K10.motifSuperResults.RData) |



---

## AT to GC curve fitting

Results from curve fitting bootstrapping procedure

Number of bootstrap repeats:  1000 

```
## Warning in readChar(con, 5L, useBytes = TRUE): cannot open compressed
## file '/data/smew1/rdavies/wildmice/motifLoss/cluster_E_2016_06_25/
## atToGCfit.losslin.K10.meanAndCI.RData', probable reason 'No such file or
## directory'
```

```
## Error in readChar(con, 5L, useBytes = TRUE): cannot open the connection
```



|               |     sameA|     sameC|     sameG|     sameT| A/T->A/T| C/G->A/T| A/T->C/G| C/G->C/G|
|:--------------|---------:|---------:|---------:|---------:|--------:|--------:|--------:|--------:|
|FAM            | 320964352| 229850309| 229857375| 321131480|  1072280|  5437907|  4740945|   704849|
|AMS            | 322313152| 231517547| 231529318| 322480663|   504515|  2432930|  2610727|   370645|
|Spretus        | 322201549| 230693752| 230705896| 322370505|   707244|  4142180|  2451962|   486409|
|AM             | 322961867| 232251335| 232263192| 323129907|   266874|  1311879|  1372612|   201831|
|PWKPhJ         | 323294333| 232260891| 232275209| 323460472|   207781|  1406590|   707941|   146280|
|CASTEiJ        | 323285703| 232324925| 232336867| 323453442|   198281|  1287685|   733101|   139493|
|CASTEiJ.PWKPhJ | 323548700| 232784116| 232796716| 323716744|    76774|   453218|   328309|    54920|
|WSBEiJ         | 323291475| 232290510| 232301919| 323460499|   201351|  1356487|   717202|   140054|
|WSBEiJ.PWKPhJ  | 323577657| 232804453| 232815696| 323745118|    66903|   419846|   280849|    48975|
|WSBEiJ.CASTEiJ | 323568901| 232813186| 232826165| 323735496|    68458|   400210|   297672|    49409|

---





Comparison of various models for fitting


```
## [1] "/data/smew1/rdavies/wildmice/motifLoss/cluster_E_2016_06_25/atToGCfit.losslin.K10.mice.modelComparison.pdf"
```

![plot of chunk atToGCcomparison](figure/atToGCcomparison-1.png)

```
## [1] Inf
```



































