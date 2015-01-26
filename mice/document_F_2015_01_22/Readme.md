#Hotspot death
##Species: mice
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
|mouseFilter           |allPlus2GATKArrayNotFilteredAnnot295.0                |What folder used for mice                                                   |
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
## '/Net/dense/data/wildmice/motifLoss/input_A_2015_01_22/forFilteringPlot.RData',
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
| 2.4723|  1.6449|  0.8274|             1.1138|
Number of Derived Mutations down a specific lineage


|FAM        |AMS        |Spretus    |AM         |PWKPhJ     |CASTEiJ    |CASTEiJ.PWKPhJ |WSBEiJ     |WSBEiJ.PWKPhJ |WSBEiJ.CASTEiJ |
|:----------|:----------|:----------|:----------|:----------|:----------|:--------------|:----------|:-------------|:--------------|
|17,913,173 | 8,758,108 |11,716,370 | 4,692,855 | 3,710,202 | 3,560,700 | 1,381,827     | 3,604,469 | 1,228,438    | 1,226,785     |
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


---

## Number of significant motifs per lineage per test

Note that the following numbers are only for motifs that have been clustered

Number of motifs (clusters) per test and lineage


|        |FAM      |AMS     |Spretus  |AM       |PWKPhJ |CASTEiJ |WSBEiJ |
|:-------|:--------|:-------|:--------|:--------|:------|:-------|:------|
|gainat  |39 (15)  |13 (4)  |46 (10)  |8 (3)    |1 (1)  |3 (2)   |0 (0)  |
|gainlin |26 (9)   |69 (18) |7 (5)    |18 (3)   |0 (0)  |0 (0)   |0 (0)  |
|lossat  |181 (27) |26 (9)  |569 (17) |140 (6)  |5 (2)  |3 (2)   |16 (3) |
|losslin |63 (14)  |87 (15) |358 (21) |437 (11) |0 (0)  |5 (1)   |11 (2) |
FAM


|        |nonRepeat |B1_Mus1 |RSINE1 |Lx8   |B1_Mm |(TCTA)n |RMER6C |B2_Mm2 |(CA)n |MTC   |ORR1B1-int |B1_Mus2 |B3A   |ID_B1 |(TG)n |(GAA)n |MTD   |RLTR15 |
|:-------|:---------|:-------|:------|:-----|:-----|:-------|:------|:------|:-----|:-----|:----------|:-------|:-----|:-----|:-----|:------|:-----|:------|
|gainat  |9 (6)     |16 (2)  |0 (0)  |0 (0) |0 (0) |6 (1)   |0 (0)  |4 (3)  |2 (1) |0 (0) |0 (0)      |1 (1)   |0 (0) |1 (1) |0 (0) |0 (0)  |0 (0) |0 (0)  |
|gainlin |2 (2)     |3 (1)   |5 (1)  |0 (0) |9 (1) |0 (0)   |0 (0)  |0 (0)  |2 (1) |0 (0) |0 (0)      |1 (1)   |2 (1) |0 (0) |2 (1) |0 (0)  |0 (0) |0 (0)  |
|lossat  |159 (19)  |1 (1)   |0 (0)  |5 (1) |0 (0) |2 (1)   |6 (1)  |0 (0)  |0 (0) |3 (1) |3 (1)      |0 (0)   |0 (0) |0 (0) |0 (0) |1 (1)  |0 (0) |1 (1)  |
|losslin |50 (10)   |0 (0)   |6 (1)  |5 (1) |0 (0) |0 (0)   |0 (0)  |0 (0)  |0 (0) |0 (0) |0 (0)      |0 (0)   |0 (0) |1 (1) |0 (0) |0 (0)  |1 (1) |0 (0)  |
AMS


|        |B2_Mm2 |nonRepeat |RSINE1 |MTC   |B1_Mus1 |B1_Mus2 |B3    |B4A   |B4    |Lx6   |(TAGA)n |
|:-------|:------|:---------|:------|:-----|:-------|:-------|:-----|:-----|:-----|:-----|:-------|
|gainat  |12 (3) |1 (1)     |0 (0)  |0 (0) |0 (0)   |0 (0)   |0 (0) |0 (0) |0 (0) |0 (0) |0 (0)   |
|gainlin |48 (9) |0 (0)     |5 (1)  |1 (1) |8 (4)   |5 (2)   |0 (0) |0 (0) |0 (0) |0 (0) |2 (1)   |
|lossat  |0 (0)  |11 (4)    |3 (1)  |6 (1) |0 (0)   |0 (0)   |0 (0) |3 (1) |0 (0) |3 (2) |0 (0)   |
|losslin |0 (0)  |47 (7)    |23 (1) |6 (1) |2 (1)   |1 (1)   |4 (2) |1 (1) |3 (1) |0 (0) |0 (0)   |
Spretus


|        |nonRepeat |B3A    |Lx3C   |B4A    |(TCTA)n |RMER17C |B3    |URR1B |(TAGA)n |Lx9   |URR1A |Lx2B  |RSINE1 |(TG)n |ID_B1 |(TTC)n |
|:-------|:---------|:------|:------|:------|:-------|:-------|:-----|:-----|:-------|:-----|:-----|:-----|:------|:-----|:-----|:------|
|gainat  |34 (6)    |1 (1)  |0 (0)  |0 (0)  |8 (1)   |0 (0)   |0 (0) |0 (0) |1 (1)   |0 (0) |2 (1) |0 (0) |0 (0)  |0 (0) |0 (0) |0 (0)  |
|gainlin |4 (3)     |0 (0)  |0 (0)  |0 (0)  |2 (1)   |0 (0)   |0 (0) |0 (0) |0 (0)   |0 (0) |0 (0) |0 (0) |0 (0)  |1 (1) |0 (0) |0 (0)  |
|lossat  |503 (5)   |27 (1) |2 (1)  |11 (2) |4 (1)   |3 (1)   |5 (1) |6 (1) |4 (1)   |2 (1) |0 (0) |1 (1) |0 (0)  |0 (0) |1 (1) |0 (0)  |
|losslin |302 (8)   |20 (2) |13 (1) |3 (2)  |0 (0)   |7 (1)   |1 (1) |0 (0) |0 (0)   |2 (1) |2 (1) |2 (1) |3 (1)  |2 (1) |0 (0) |1 (1)  |
AM


|        |nonRepeat |ID_B1  |B3    |RSINE1 |B1_Mus2 |
|:-------|:---------|:------|:-----|:------|:-------|
|gainat  |7 (2)     |0 (0)  |0 (0) |0 (0)  |1 (1)   |
|gainlin |11 (1)    |4 (1)  |3 (1) |0 (0)  |0 (0)   |
|lossat  |136 (5)   |0 (0)  |4 (1) |0 (0)  |0 (0)   |
|losslin |410 (4)   |12 (4) |8 (2) |7 (1)  |0 (0)   |
PWKPhJ


|       |(TCTA)n |B2_Mm2 |nonRepeat |
|:------|:-------|:------|:---------|
|gainat |0 (0)   |1 (1)  |0 (0)     |
|lossat |4 (1)   |0 (0)  |1 (1)     |
CASTEiJ


|        |nonRepeat |(TAGA)n |
|:-------|:---------|:-------|
|gainat  |1 (1)     |2 (1)   |
|lossat  |3 (2)     |0 (0)   |
|losslin |5 (1)     |0 (0)   |
CASTEiJ.PWKPhJ
No significant results

WSBEiJ


|        |nonRepeat |
|:-------|:---------|
|lossat  |16 (3)    |
|losslin |11 (2)    |
WSBEiJ.PWKPhJ
No significant results

WSBEiJ.CASTEiJ
No significant results

---


## Features of significant motifs

Test: losslin 
AT content 


|                |   0|   1|   2|    3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|----:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.6| 3.1| 10.0| 20.0| 25.5| 21.6| 12.5| 5.0| 1.4| 0.3|
|significant     | 0.0| 0.0| 0.5|  4.2| 12.9| 32.0| 31.9| 16.2| 2.3| 0.0| 0.0|
Number of CpGs 


|                |    0|    1|   2|   3|   4|
|:---------------|----:|----:|---:|---:|---:|
|not significant | 69.9| 28.0| 2.1| 0.0| 0.0|
|significant     | 99.7|  0.3| 0.0| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 7.2| 54.5| 28.4| 7.7| 1.8| 0.3| 0.1| 0.0| 0.0| 0.0|
|significant     | 8.9| 68.8| 19.9| 2.4| 0.0| 0.0| 0.0| 0.0| 0.0| 0.0|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 2.0| 23.6| 74.4|
|significant     | 0.0| 1.2| 30.3| 68.6|
Test: lossat 
AT content 


|                |   0|   1|   2|   3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|---:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.3| 2.5| 9.2| 19.3| 25.8| 22.8| 13.4| 5.1| 1.3| 0.3|
|significant     | 0.0| 0.0| 0.5| 3.1|  9.5| 26.6| 33.8| 20.7| 5.6| 0.1| 0.0|
Number of CpGs 


|                |    0|    1|   2|   3|
|:---------------|----:|----:|---:|---:|
|not significant | 68.1| 31.8| 0.1| 0.0|
|significant     | 99.8|  0.2| 0.0| 0.0|
Maximum run length 


|                |   1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|---:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant | 7.2| 55.2| 28.2| 7.4| 1.6| 0.3| 0.1| 0.0| 0.0| 0.0|
|significant     | 5.4| 62.1| 29.6| 2.7| 0.1| 0.0| 0.0| 0.0| 0.0| 0.0|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.3| 22.3| 76.4|
|significant     | 0.0| 1.4| 36.3| 62.2|
Test: gainlin 
AT content 


|                |   0|   1|   2|   3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|---:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.5| 3.0| 9.6| 19.7| 25.5| 21.9| 12.7| 5.0| 1.6| 0.4|
|significant     | 0.0| 0.9| 2.7| 7.2| 23.4| 40.5| 21.6|  3.6| 0.0| 0.0| 0.0|
Number of CpGs 


|                |    0|    1|   2|   3|
|:---------------|----:|----:|---:|---:|
|not significant | 64.7| 32.8| 2.5| 0.0|
|significant     | 84.7| 14.4| 0.9| 0.0|
Maximum run length 


|                |    1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|----:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant |  7.6| 55.2| 27.7| 7.4| 1.7| 0.4| 0.1| 0.0| 0.0| 0.0|
|significant     | 20.7| 65.8| 12.6| 0.0| 0.0| 0.0| 0.0| 0.9| 0.0| 0.0|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 2.0| 22.5| 75.6|
|significant     | 0.0| 4.5| 20.7| 74.8|
Test: gainat 
AT content 


|                |   0|   1|   2|   3|    4|    5|    6|    7|   8|   9|  10|
|:---------------|---:|---:|---:|---:|----:|----:|----:|----:|---:|---:|---:|
|not significant | 0.0| 0.4| 2.7| 9.6| 19.8| 25.9| 22.4| 12.8| 4.8| 1.2| 0.2|
|significant     | 0.0| 0.0| 0.9| 7.5| 14.2| 34.0| 34.0|  9.4| 0.0| 0.0| 0.0|
Number of CpGs 


|                |    0|    1|   2|   3|
|:---------------|----:|----:|---:|---:|
|not significant | 61.5| 37.8| 0.7| 0.0|
|significant     | 88.7| 11.3| 0.0| 0.0|
Maximum run length 


|                |    1|    2|    3|   4|   5|   6|   7|   8|   9|  10|
|:---------------|----:|----:|----:|---:|---:|---:|---:|---:|---:|---:|
|not significant |  7.3| 56.1| 27.6| 7.0| 1.5| 0.3| 0.1| 0.0| 0.0| 0.0|
|significant     | 20.8| 48.1| 29.2| 1.9| 0.0| 0.0| 0.0| 0.0| 0.0| 0.0|
Nucleotide diversity 


|                |   1|   2|    3|    4|
|:---------------|---:|---:|----:|----:|
|not significant | 0.0| 1.0| 21.1| 78.0|
|significant     | 0.0| 3.8| 29.2| 67.0|
---





## PWK results at CAST top results

Only for the mouse analysis

Other studies have found that M. m. musculus has a PRDM9 allele similar to the CAST motif, but while we found a denovo CAST motif similar to published work, here we found no de novo PWK motifs. Here, we look at the p-values for the motifs in the top CAST cluster in the other strains
Test: losslin


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |p_WSBEiJ.PWKPhJ |or_WSBEiJ.PWKPhJ |p_CASTEiJ.PWKPhJ |or_CASTEiJ.PWKPhJ |p_WSBEiJ.CASTEiJ |or_WSBEiJ.CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|:---------------|:----------------|:----------------|:-----------------|:----------------|:-----------------|
|AGACTTGGCT |0.081    |1.2       |0.073    |0.8       |1.9e-07   |1.67       |0.64            |1.09             |0.32             |0.81              |0.39             |1.16              |
|AGACTTTGCT |0.87     |1.01      |0.91     |1.01      |1.7e-10   |1.83       |0.64            |1.08             |0.051            |1.35              |1                |0.99              |
|AATCCAAGTC |0.73     |0.94      |0.94     |1         |1.6e-14   |2.35       |0.72            |0.87             |0.58             |1.1               |0.2              |1.29              |
|AAGCCAAGTC |0.073    |1.21      |0.86     |0.97      |1.4e-10   |1.87       |0.12            |0.69             |0.71             |1.06              |0.0062           |1.57              |
|AATCCCAGTC |0.84     |1.02      |0.12     |0.79      |5e-07     |1.79       |0.064           |0.56             |0.44             |1.17              |0.2              |1.31              |
|AAGCAAAGTC |0.62     |1.05      |0.55     |0.93      |6.5e-12   |1.88       |0.52            |0.86             |0.046            |1.36              |0.31             |1.18              |
|AAGCCCAGTC |0.099    |1.22      |0.95     |1.01      |5.4e-09   |1.87       |0.91            |0.93             |0.61             |0.86              |3e-04            |1.88              |
|GTCCAAGTCA |0.81     |1.03      |1        |0.99      |0.47      |1.11       |0.79            |1.06             |0.31             |1.28              |0.34             |0.7               |
|GGCCAAGTCA |0.31     |1.15      |0.095    |1.26      |0.77      |1.05       |NA              |NA               |0.24             |0.7               |0.17             |1.35              |
|GCCCAAGTCA |0.11     |1.22      |0.45     |1.1       |0.49      |1.1        |0.64            |1.11             |0.58             |1.12              |0.41             |1.2               |
|GACCAAGTCA |0.71     |1.05      |0.41     |1.12      |0.039     |1.34       |0.7             |1.07             |NA               |NA                |0.37             |1.21              |
|AGTCCAAGTC |0.21     |0.79      |0.82     |0.95      |0.3       |1.16       |0.36            |1.22             |0.32             |0.72              |0.79             |0.88              |
|AGGCCAAGTC |0.26     |1.16      |0.53     |1.08      |0.52      |1.09       |0.81            |0.92             |0.43             |0.79              |1                |0.95              |
|AGCCCAAGTC |0.068    |1.24      |0.38     |1.11      |0.016     |1.32       |0.6             |1.09             |0.76             |1.05              |0.28             |1.22              |
|AGACCAAGTC |0.23     |1.17      |0.67     |1.05      |0.43      |1.12       |0.23            |1.29             |1                |0.95              |0.63             |0.84              |
Test: lossat


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |p_WSBEiJ.PWKPhJ |or_WSBEiJ.PWKPhJ |p_CASTEiJ.PWKPhJ |or_CASTEiJ.PWKPhJ |p_WSBEiJ.CASTEiJ |or_WSBEiJ.CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|:---------------|:----------------|:----------------|:-----------------|:----------------|:-----------------|
|AGACTTGGCT |0.15     |1.33      |0.89     |0.9       |2.1e-05   |1.9        |NA              |NA               |NA               |NA                |NA               |NA                |
|AGACTTTGCT |0.82     |1.05      |0.91     |0.93      |3.9e-08   |2.11       |NA              |NA               |NA               |NA                |NA               |NA                |
|AATCCAAGTC |0.0018   |2.07      |0.89     |0.9       |0.028     |1.46       |NA              |NA               |NA               |NA                |NA               |NA                |
|AAGCCAAGTC |0.025    |1.54      |0.8      |1.05      |4.3e-09   |2.26       |NA              |NA               |0.11             |1.65              |NA               |NA                |
|AATCCCAGTC |0.022    |1.69      |NA       |NA        |0.00031   |1.89       |NA              |NA               |NA               |NA                |NA               |NA                |
|AAGCAAAGTC |0.062    |1.43      |0.16     |0.66      |1.5e-07   |2.02       |NA              |NA               |NA               |NA                |NA               |NA                |
|AAGCCCAGTC |0.8      |1.04      |0.78     |1.04      |0.034     |1.47       |NA              |NA               |NA               |NA                |0.36             |1.33              |
|GTCCAAGTCA |0.095    |1.56      |0.22     |1.45      |0.25      |1.37       |NA              |NA               |NA               |NA                |NA               |NA                |
|GGCCAAGTCA |0.76     |1.07      |1        |0.99      |NA        |NA         |NA              |NA               |NA               |NA                |NA               |NA                |
|GCCCAAGTCA |0.58     |1.13      |0.67     |0.83      |0.46      |1.21       |NA              |NA               |NA               |NA                |0.00022          |3.34              |
|GACCAAGTCA |0.00085  |2.21      |NA       |NA        |0.58      |1.16       |NA              |NA               |NA               |NA                |NA               |NA                |
|AGTCCAAGTC |NA       |NA        |0.39     |1.3       |0.88      |1.04       |NA              |NA               |NA               |NA                |NA               |NA                |
|AGGCCAAGTC |0.00042  |2.15      |NA       |NA        |1         |0.97       |NA              |NA               |NA               |NA                |NA               |NA                |
|AGCCCAAGTC |0.9      |1         |0.35     |1.25      |NA        |NA         |0.0075          |2.41             |NA               |NA                |0.00079          |2.77              |
|AGACCAAGTC |0.026    |1.67      |NA       |NA        |0.15      |1.4        |NA              |NA               |NA               |NA                |NA               |NA                |
Test: gainlin


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |p_WSBEiJ.PWKPhJ |or_WSBEiJ.PWKPhJ |p_CASTEiJ.PWKPhJ |or_CASTEiJ.PWKPhJ |p_WSBEiJ.CASTEiJ |or_WSBEiJ.CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|:---------------|:----------------|:----------------|:-----------------|:----------------|:-----------------|
|AGACTTGGCT |0.72     |0.95      |0.31     |0.88      |0.12      |1.2        |0.62            |1.1              |0.57             |1.11              |0.69             |1.08              |
|AGACTTTGCT |0.92     |1.01      |0.37     |0.9       |0.96      |1          |0.37            |0.82             |0.73             |1.06              |0.12             |1.28              |
|AATCCAAGTC |0.78     |1.04      |0.83     |0.96      |0.12      |0.77       |0.34            |1.24             |0.91             |1                 |0.28             |0.73              |
|AAGCCAAGTC |0.52     |0.91      |0.16     |1.18      |0.75      |0.95       |0.33            |0.76             |0.68             |1.07              |0.13             |1.34              |
|AATCCCAGTC |0.83     |0.95      |0.017    |1.36      |1         |1          |0.71            |0.87             |0.13             |0.63              |0.39             |0.76              |
|AAGCAAAGTC |0.83     |1.02      |0.83     |1.02      |0.32      |0.89       |0.58            |0.87             |0.66             |0.9               |0.71             |0.9               |
|AAGCCCAGTC |0.83     |0.95      |0.72     |0.94      |0.099     |0.76       |0.81            |0.92             |0.5              |1.14              |0.47             |1.16              |
|GTCCAAGTCA |0.13     |0.76      |0.88     |1.02      |0.69      |1.06       |0.43            |0.75             |0.9              |0.94              |0.35             |1.26              |
|GGCCAAGTCA |0.68     |1.07      |0.46     |0.87      |0.41      |1.14       |0.5             |0.78             |NA               |NA                |0.22             |1.33              |
|GCCCAAGTCA |0.18     |1.23      |0.39     |0.85      |0.26      |1.19       |1               |0.94             |0.38             |1.23              |NA               |NA                |
|GACCAAGTCA |0.16     |0.78      |0.76     |1.05      |0.39      |0.85       |0.6             |1.12             |0.11             |0.6               |0.43             |1.21              |
|AGTCCAAGTC |0.39     |1.13      |0.087    |0.73      |0.87      |0.96       |0.36            |0.73             |0.8              |1.04              |NA               |NA                |
|AGGCCAAGTC |0.45     |1.11      |1        |0.98      |0.65      |0.92       |1               |0.96             |0.55             |0.82              |0.38             |1.23              |
|AGCCCAAGTC |0.56     |0.9       |0.24     |0.83      |0.94      |0.98       |0.81            |0.91             |0.049            |1.48              |0.71             |0.87              |
|AGACCAAGTC |0.31     |0.84      |0.19     |0.81      |0.24      |0.82       |0.47            |1.16             |0.42             |0.79              |0.22             |1.3               |
Test: gainat


|           |p_WSBEiJ |or_WSBEiJ |p_PWKPhJ |or_PWKPhJ |p_CASTEiJ |or_CASTEiJ |p_WSBEiJ.PWKPhJ |or_WSBEiJ.PWKPhJ |p_CASTEiJ.PWKPhJ |or_CASTEiJ.PWKPhJ |p_WSBEiJ.CASTEiJ |or_WSBEiJ.CASTEiJ |
|:----------|:--------|:---------|:--------|:---------|:---------|:----------|:---------------|:----------------|:----------------|:-----------------|:----------------|:-----------------|
|AGACTTGGCT |0.08     |1.49      |0.89     |0.89      |0.12      |1.37       |NA              |NA               |NA               |NA                |NA               |NA                |
|AGACTTTGCT |1        |0.95      |0.82     |1.03      |1         |0.99       |NA              |NA               |NA               |NA                |0.23             |1.39              |
|AATCCAAGTC |NA       |NA        |0.17     |1.43      |0.39      |1.26       |NA              |NA               |NA               |NA                |NA               |NA                |
|AAGCCAAGTC |0.24     |1.33      |NA       |NA        |0.48      |1.18       |NA              |NA               |NA               |NA                |NA               |NA                |
|AATCCCAGTC |NA       |NA        |1        |0.94      |NA        |NA         |NA              |NA               |NA               |NA                |NA               |NA                |
|AAGCAAAGTC |0.42     |0.78      |1        |0.96      |0.62      |1.11       |NA              |NA               |NA               |NA                |NA               |NA                |
|AAGCCCAGTC |0.028    |1.73      |NA       |NA        |NA        |NA         |NA              |NA               |NA               |NA                |NA               |NA                |
|GTCCAAGTCA |0.0094   |2.12      |0.18     |1.45      |NA        |NA         |NA              |NA               |NA               |NA                |NA               |NA                |
|GGCCAAGTCA |NA       |NA        |NA       |NA        |0.74      |1.09       |NA              |NA               |NA               |NA                |NA               |NA                |
|GCCCAAGTCA |NA       |NA        |NA       |NA        |1         |0.96       |NA              |NA               |NA               |NA                |NA               |NA                |
|GACCAAGTCA |NA       |NA        |NA       |NA        |NA        |NA         |NA              |NA               |NA               |NA                |NA               |NA                |
|AGTCCAAGTC |0.1      |1.55      |NA       |NA        |NA        |NA         |NA              |NA               |NA               |NA                |NA               |NA                |
|AGGCCAAGTC |NA       |NA        |0.63     |1.13      |0.13      |1.52       |NA              |NA               |NA               |NA                |NA               |NA                |
|AGCCCAAGTC |0.5      |1.21      |NA       |NA        |NA        |NA         |NA              |NA               |NA               |NA                |NA               |NA                |
|AGACCAAGTC |NA       |NA        |0.11     |1.57      |0.39      |1.27       |NA              |NA               |NA               |NA                |NA               |NA                |
---




## AT to GC changes along lineages

For each lineage, stratify into percents of the bases that are the same from its ancestor, and those which changed from the ancestor into this lineage, what they are towards


|               |   sameA|   sameC|   sameG|   sameT| A/T->A/T| C/G->A/T| A/T->C/G| C/G->C/G|
|:--------------|-------:|-------:|-------:|-------:|--------:|--------:|--------:|--------:|
|FAM            | 28.8181| 20.6373| 20.6380| 28.8331|   0.0963|   0.4882|   0.4257|   0.0633|
|AMS            | 28.9392| 20.7870| 20.7881| 28.9542|   0.0453|   0.2184|   0.2344|   0.0333|
|Spretus        | 28.9292| 20.7131| 20.7142| 28.9444|   0.0635|   0.3719|   0.2202|   0.0437|
|AM             | 28.9975| 20.8529| 20.8540| 29.0125|   0.0240|   0.1178|   0.1232|   0.0181|
|PWKPhJ         | 29.0273| 20.8538| 20.8551| 29.0422|   0.0187|   0.1263|   0.0636|   0.0131|
|CASTEiJ        | 29.0265| 20.8595| 20.8606| 29.0416|   0.0178|   0.1156|   0.0658|   0.0125|
|CASTEiJ.PWKPhJ | 29.0501| 20.9008| 20.9019| 29.0652|   0.0069|   0.0407|   0.0295|   0.0049|
|WSBEiJ         | 29.0270| 20.8564| 20.8575| 29.0422|   0.0181|   0.1218|   0.0644|   0.0126|
|WSBEiJ.PWKPhJ  | 29.0527| 20.9026| 20.9036| 29.0678|   0.0060|   0.0377|   0.0252|   0.0044|
|WSBEiJ.CASTEiJ | 29.0520| 20.9034| 20.9045| 29.0669|   0.0061|   0.0359|   0.0267|   0.0044|
---




## Giant AT to GC plot

All clustered AT to GC plots collapsed together for the loss lineage test

![plot of chunk atToGCAll](figure/atToGCAll.png) 



---

## Correlation between some top motifs

Currently only for mice


This plot shows correlation of some pre-selected top motifs on different lineages and backgrounds with each other and with 3 linkage-disequilibrium based maps, at fine and broad scales.

```
## Warning: all scheduled cores encountered errors in user code
```

```
## Error: $ operator is invalid for atomic vectors
```



```
## The following plot shows the breakdown chromosome wide
```

```
## Warning: cannot open compressed file
## '/Net/dense/data/wildmice/motifLoss/cluster_E_2015_01_22/allCorrelations.RData',
## probable reason 'No such file or directory'
```

```
## Error: cannot open the connection
```






```
## Warning: cannot open compressed file
## '/Net/dense/data/wildmice/motifLoss/cluster_E_2015_01_22/allCorrelations.RData',
## probable reason 'No such file or directory'
```

```
## Error: cannot open the connection
```


---

## Links to pages of significant motifs


|               |losslin                                                                                                                      |lossat                                                                                                                      |gainlin                                                                                                                      |gainat                                                                                                                      |
|:--------------|:----------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------|
|FAM            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/FAM/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/FAM/)            |
|AMS            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/AMS/)            |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/AMS/)            |
|Spretus        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/Spretus/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/Spretus/)        |
|AM             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/AM/)             |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/AM/)             |
|PWKPhJ         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/PWKPhJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/PWKPhJ/)         |
|CASTEiJ        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/CASTEiJ/)        |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/CASTEiJ/)        |
|CASTEiJ.PWKPhJ |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/CASTEiJ.PWKPhJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/CASTEiJ.PWKPhJ/) |
|WSBEiJ         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/WSBEiJ/)         |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/WSBEiJ/)         |
|WSBEiJ.PWKPhJ  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/WSBEiJ.PWKPhJ/)  |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/WSBEiJ.PWKPhJ/)  |
|WSBEiJ.CASTEiJ |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/losslin/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/lossat/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainlin/WSBEiJ.CASTEiJ/) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifPvalues/gainat/WSBEiJ.CASTEiJ/) |

---

## Motif Cluster RData


|losslin                                                                                                                                       |lossat                                                                                                                                       |gainlin                                                                                                                                       |gainat                                                                                                                                       |
|:---------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------|
|[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifSuperResults/losslin.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifSuperResults/lossat.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifSuperResults/gainlin.K10.motifSuperResults.RData) |[Link](https://github.com/rwdavies/hotspotDeath/blob/master/mice/document_F_2015_01_22/motifSuperResults/gainat.K10.motifSuperResults.RData) |





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

![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage1.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage2.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage3.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage4.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage5.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage6.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage7.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage8.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage9.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage10.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage11.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage12.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage13.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage14.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage15.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage16.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage17.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage18.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage19.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage20.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage21.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage22.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage23.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage24.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage25.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage26.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage27.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage28.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage29.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage30.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage31.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage32.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage33.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage34.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage35.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage36.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage37.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage38.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage39.png) ![plot of chunk pValuesWithinMethodsBetweenLineage](figure/pValuesWithinMethodsBetweenLineage40.png) 
---





-->
