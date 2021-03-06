# ___**PEA**: An integrated R toolkit for plant epitranscriptome analysis___ </br>
![](https://halobi.com/wp-content/uploads/2016/08/r_logo.png "R logo")
![](https://encrypted-tbn2.gstatic.com/images?q=tbn:ANd9GcSvCvZWbl922EJkjahQ5gmTpcvsYr3ujQBpMdyX-YG99vGWfTAmfw "linux logo")
![](https://tctechcrunch2011.files.wordpress.com/2014/06/apple_topic.png?w=220) </br>
**Brief introduction:**
PEA is a docker-based integrated R toolkit that aims to facilitate the plant epitranscriptome analysis. This toolkit contains a comprehensive collection of functions required for read mapping, CMR calling, motif scanning and discovery, and gene functional enrichment analysis. PEA also takes advantage of machine learning technologies for transcriptome-scale CMR prediction, with high prediction accuracy, using the Positive Samples Only Learning algorithm, which addresses the two-class classification problem by using only positive samples (CMRs), in the absence of negative samples (non-CMRs). Hence PEA is a versatile epitranscriptome analysis pipeline covering CMR calling, prediction, and annotation.</br>
## PEA installation ##
### Docker installation and start ###
#### For Windows (Test on Windows 10 Enterprise version): ####
* Download [Docker](<https://download.docker.com/win/stable/Docker%20for%20Windows%20Installer.exe>) for windows </br>
* Double click the EXE file to open it;
* Follow the wizard instruction and complete installation;
* Search docker, select ___Docker for Windows___ in the search results and clickit.
#### For Mac OS X (Test on macOS Sierra version 10.12.6 and macOS High Sierra version 10.13.3): ####
* Download [Docker](<https://download.docker.com/mac/stable/Docker.dmg>) for Mac os <br>
* Double click the DMG file to open it;
* Drag the docker into Applications and complete installation;
* Start docker from Launchpad by click it.
#### For Ubuntu (Test on Ubuntu 14.04 LTS and Ubuntu 16.04 LTS): ####
* Go to [Docker](<https://download.docker.com/linux/ubuntu/dists/>), choose your Ubuntuversion, browse to ___pool/stable___ and choose ___amd64, armhf, ppc64el or s390x.____ Download the ___DEB___ file for the Docker version you want to install;
* Install Docker, supposing that the DEB file is download into following path:___"/home/docker-ce<version-XXX>~ubuntu_amd64.deb"___ </br>
```bash
$ sudo dpkg -i /home/docker-ce<version-XXX>~ubuntu_amd64.deb      
$ sudo apt-get install -f
```
 ### Verify if Docker is installed correctly ### 
----------------------------------------
   Once Docker installation is completed, we can run ____hello-world____ image to verify if Docker is installed correctly. Open terminal in Mac OS X and Linux operating system and open CMD for Windows operating system, then type the following command:
```bash
$ docker run hello-world
```
   **<font color =red>Note</font>:** root permission is required for Linux operating system.
   **<font color =red>Note</font>:** considering that differences between different computers may exist, please refer to [official installation manual](https://docs.docker.com/install) if instructions above don’t work.

### PEA installation from Docker Hub ###
--------------------------------
  For Mac OS X and Linux operating systems, open the terminal, for Windows operating system, open CMD. Typing the following command:
```bash
# Pull PEA from Docker PEA  
$ docker pull malab/pea 
```
## Quickly start ##
-------------
Once PEA is installed successfully, type the following command to start PEA:  
```bash
$ docker run -it -v /host directory of dataset:/home/data malab/pea R  
```
**Note:** Supposing that users’ private dataset is located in directory ___`/home/test`____, then change the words above (____`/host directory of dataset`____) to host directory (____`/home/test`____)  
```R
library(PEA)  
setwd("/home/data/")  
```
**Important:** the directory (____`/home/data/`____) is a virtual directory in PEA Docker image. In order to use private dataset more easily, the parameter “-v” is strongly recommended to mount host directory of dataset to PEA image.  
### CMR calling ### 
----------------------------------------------------------
#### Reads mapping using tophat ####
```R
# Loading sample data for reads mapping  
fq <-  system.file("extdata/test.fq", package = "PEA")  
referenceGenome <- system.file("extdata/chromosome1.fa", package = "PEA")

# reads mapping using tophat with default parameter, the alignment results will be saved to the working directory (/host directory of dataset)  
test.bam <- readMapping(alignment = "tophat", fq = fq, refGenome = referenceGenome, paired = F)  

# reads mapping using tophat with 2 threads  
test.bam <- readMapping(alignment = "tophat", fq = fq,refGenome = referenceGenome, paired = F, ... = "-p 2")  
```
   **Note:** other alignment toolkits such as Bowtie, Bowtie 2, TopHat 2, Hisat
   and Hisat can be easily invoked by specifying “alignment” parameter in
   “readMapping” function.

#### Peak calling methods implemented in PEA ####
---------------------------------------
   PEA integrated five peak calling methods including “SlidingWindow”,
   “exomePeak”, “MetPeak”, “MACS2” and “BayesPeak”. each peak calling method
   can be easily invoked by specifying parameter “method” in “CMRCalling”
   function.
   **How to choose:**
1.  “SlidingWindow” is used to examine the significant enriched regions (peaks)
    using Fisher’s exact test-based sliding window method between RIP and input
    samples.  
2.  “exomePeak” is an exome-based peak calling method designed specifically for
    analysis of MeRIP-seq (m6A-seq).  
3.  “MetPeak” is a peak calling method used for transcriptome-wide m6A detection
    designed for MeRIP-seq data which is developed based on graphical model.  
4.  “MACS2” is firstly designed to identify transcription binding sites in
    ChIP-seq dataset and can also be used for peak calling in MeRIP-seq because
    of the similar principle.  
5.  “BayesPeak” is specially designed to detect genomic enriched regions in
    ChIP-seq dataset using Bayesian hidden Markov model and Markov chain Monte
    Carlo algorithms.     
   **Summary:** Both “SlidingWindow” and “exomePeak” are sliding window-based method to detect significant enriched regions and perform well on CMR  prediction, while “MACS2” and “BayesPeak” are peak calling methods designed for ChIP-seq dataset, but “MACS2” can strictly control false positives, in contrast, BayePeak is weaker in controlling false positives. In addition, method “exomePeak”, “MeTPeak” and “MACS2” can accept biological replicates using statistical methods.
* Peak calling using SlindingWindow
```R
# Loading sample data for peak calling  
input.bam <- system.file("extdata/chr1_input_test.bam", package = "PEA")     
RIP.bam <- system.file("extdata/chr1_RIP_test.bam", package = "PEA")      
refGenome <- system.file("extdata/chromosome1.fa", package = "PEA")      
GTF <- system.file("extdata/chromosome1.gtf", package = "PEA")      

# Peak calling using sliding window-based method  
cmrMat <- CMRCalling(CMR = "m6A", IPBAM = RIP.bam, inputBAM = input.bam, method = "SlidingWindow", mappedInput = 17472,   mappedRIP = 20072, refGenome = refGenome)
# Note: one can also extract chromosome sizes to replace FASTA format genome sequences, see following command
system(paste0("samtools faidx ", refGenome))
system(paste0("cut -f1,2 ", refGenome, ".fai > genome.size"))
cmrMat <- CMRCalling(CMR = "m6A", IPBAM = RIP.bam, inputBAM = input.bam, method = "SlidingWindow", mappedInput = 17472, mappedRIP = 20072, refGenome = "genome.size")
# Save the results into working directory  
write.table(cmrMat, file = "SlidingWindow_peaks.txt", sep = "\t", quote = F, row.names = F, col.names = F)  
```
**Note:** parameters "mappedInput" and "mappedRIP" represent the number of reads aligned to reference genome in input and RIP samples, respectively.
* Peak calling using exomePeak
```R
cmrMat <- CMRCalling(CMR = "m6A", method = "exomePeak", IPBAM = RIP.bam, inputBAM = input.bam, GTF = GTF)  
write.table(cmrMat, file = "exomePeak_peaks.txt", sep = "\t", quote = F, row.names = F, col.names = F)  
```
* Peak calling using MeTPeak
```R
cmrMat <- CMRCalling(CMR = "m6A", method = "MetPeak", IPBAM = RIP.bam,  inputBAM = input.bam, GTF = GTF)  
write.table(cmrMat, file = "MetPeak_peaks.txt", sep = "\t",quote = F, row.names = F, col.names = F)  
```
* Peak calling using MACS2
```R
cmrMat <- CMRCalling(CMR = "m6A", method = "MACS2", IPBAM = RIP.bam, inputBAM = input.bam, GTF = GTF, ...="--nomodel")  
```
**Note:** futher parameters recognized by MACS2 can be specified in "..."  
```R
write.table(cmrMat, file = "MACS2_peaks.txt", sep = "\t", quote = F, row.names = F, col.names = F)  
```
* Peak calling using BayesPeak  
```R
cmrMat <- CMRCalling(CMR = "m6A", method = "BayesPeak", IPBAM = RIP.bam, inputBAM = input.bam, GTF = GTF)  
write.table(cmrMat, file = "BayesPeak_peaks.txt", sep = "\t", quote = F, row.names = F, col.names = F)  
```
### CMR prediction ###
-------------------------------
#### Samples organization ####
   <br> In order to be recognized by the ML-based classification models, PEA can transform each sample (L-nt flanking sequence centered on m6A or non-m6A modifications) into a (L*4+20+22)-dimensional vector which including L*4 binary-based features, 20 k-mer based features (k = 1 and k = 2) and 22 PseDNC-based features.</br>
```R
# Convert genomic position to cDNA position  
GTF <- system.file("extdata/chromosome1.gtf", package = "PEA")  
peaks <- G2T(bedPos = cmrMat, GTF = GTF)  

# Searching RRACH motif from cDNA sequence  
cDNA <- system.file("extdata/chr1_cdna.fa", package = "PEA")    
motifPos <- searchMotifPos(sequence = cDNA)  

# Find confident positive samples and unlabel samples  
posSamples <- findConfidentPosSamples(peaks = peaks, motifPos = motifPos)  
unlabelSamples <- findUnlabelSamples(cDNAID = posSamples$cDNAID, motifPos = motifPos, posSamples = posSamples$positives)  
```    
#### Sample vectorization with three feature encoding schemes ####
--------------------------------------------------------
   In order to be recognized by the ML-based classification models, PEA can
   transform each sample (*L*-nt flanking sequence centered on m6A or non-m6A
   modifications) into a (*L*\*4+20+22)-dimensional vector which including
   *L*\*4 *binary*-based features, 20 *k-mer* based features (*k* = 1 and *k* =
   2) and 22 PseDNC-based features.
``` R
# Extracting flanking sequence of 101-nt centered on m6A or non-m6A  
positives <- posSamples\$positives  
posSeq <- extractSeqs(RNAseq = cDNA, samples = positives, seqLen = 101)  
unlabelSeq <- extractSeqs(RNAseq = cDNA, samples = unlabelSamples, seqLen = 101)  

# Feature encoding using *binary*, *k-mer* and *PseDNC* encoding schemes  
posFeatureMat <- featureEncoding(RNAseq = posSeq)  
unlabelFeatureMat <- featureEncoding(RNAseq = unlabelSeq)  
featureMat <- rbind(posFeatureMat, unlabelFeatureMat)  
```
#### Construction of a CRM predictor using random forest (RF) and PSOL algorithm
---------------------------------------------------------------------------
   <br>The RF is a powerful ML algorithm that has been widely used for the binary classification problems in bioinformatics and computational biology (Cui, et al., 2015; Ma, et al., 2014; Touw, et al., 2013). To build a RF-based classifier (predictor), positive samples and negative samples are required to be specified by the user. Existing m6A prediction approach typically use the known m6A modifications as the positive sample set and the unlabeled samples as the negative sample set to build predictors to identify new m6A modifications from unlabeled samples (Chen, et al., 2016; Zhou, et al., 2016). Such kind of m6A predictors is actually built from a noisy negative sample set. As a result, the predictors do not perform well as they could in identifying new m6A modifications. To address this challenge, PSOL algorithm was introduced into the PEA package, which can be used to build an ML-based binary classification system with high prediction accuracy with only positive samples and without pre-specified negative samples. This algorithm has been previously applied to predict abiotic stress-response genes and genomic loci encoding functional noncoding RNAs (Ma, et al., 2014; Wang, et al., 2006). </br>
    <br>To implement the PSOL algorithm, an initial negative sample set with the same size as the positive sample set was firstly constructed, which contained samples that were selected from the unlabeled sample set based on the maximal Euclidean distances to positive samples. The negative sample set was then expanded iteratively using the RF-based classifier until the designated iteration number was reached. In each iteration, a ten-fold cross-validation method and the receiver operating characteristic (ROC) curve analysis were performed to evaluate the predictive performance of current classifier in distinguishing between positive samples and negative samples. In order to get the optimal RF classifier, the RF classifier with max AUC value was selected from the ten-fold cross-validation and was subsequently used for scoring the unlabeled samples with a user-adjustable threshold (TPR = 0.995) to ensure a large fraction of positive samples can be correctly identified.</br>
    <br>In each iteration, an m6A predictor was built using the RF algorithm with positive samples and selected negative samples. PSOL algorithm can be easily implemented using the function “PSOL” in the PEA package.</br>
```R
# Creating a directory to save psol results  
dir.create("./psol")  
psolResDic <- "./psol/"  

# Starting running PSOL  
psolRes <- PSOL(featureMatrix = featureMat, positives = positives,  
                  unlabels = unlabelSamples, PSOLResDic = psolResDic, cpus = 1)

# Extracting negative samples generated by PSOL    
negatives <- psolRes$finalNegatives  

# Performing 5-fold cross-validation experiment to evaluate the performance of predictor  
cvRes <- cross_validation(featureMat = featureMat, positives = positives, negatives = negatives, cross = 5)  

# Plotting ROC curves for 5-fold cross-validation   
pdf("cross_validation.pdf", height = 5, width = 5)  
plotROC(cvRes = cvRes)  
dev.off()  
```
#### CMR prediction using predictor generated by PSOL ####
------------------------------------------------
  After performing PSOL, an m6A predictor would be generated in the “psolRes”
   object, users can easily predict novel candidate CMRs by following command.
```R
# Predicting novel candidate m6A modifications  
predSeq <- system.file("extdata/test_pred.fa", package = "PEA")  
predMat <- predCMR(predSeq, model = psolRes$model)  
```
### CMR annotation ###
  PEA also provide CMR annotation to provide insights into spatial and functional associations of CMRs through function “CMRAnnotation”. Using this function, the manner of distribution of CMRs in the transcriptome is statistically analyzed, including the spatial distribution of CMRs, and the regions of enrichment of CMRs within transcripts. In addition, motif scanning and de novo motif discovery are also provided to investigate the potential regulatory mechanisms leading by CMRs. Moreover, gene functional (Gene Ontology) enrichment analysis is also performed to characterize the enriched functions of CMR-corresponding transcripts using R package “topGO”

#### CMR location distribution ####
```R
# Loading sample data  
GTF <- system.file("extdata/chromosome1.gtf", package = "PEA")  
input.bam <- system.file("extdata/chr1_input_test.bam", package = "PEA")   
RIP.bam <- system.file("extdata/chr1_RIP_test.bam", package = "PEA")    
refGenome <- system.file("extdata/chromosome1.fa", package = "PEA")    
cDNA <- system.file("extdata/chr1_cdna.fa", package = "PEA")  

# Extract the UTR position from GTF file  
UTRMat <- getUTR(GTF = GTF)    
cmrMat <- CMRCalling(CMR = "m6A", IPBAM = RIP.bam, inputBAM = input.bam, method = "SlidingWindow", mappedInput = 17472,  mappedRIP = 20072, refGenome = refGenome)  

#  Perform CMR location distribution analysis  
pdf("CMR_location.pdf", height = 10, width = 10)  
results <- CMRAnnotation(cmrMat = cmrMat, SNR = F, UTRMat = UTRMat,  genomic = T, annotation = "location", GTF = GTF, RNAseq = cDNA)   
dev.off()  
```
#### Motif scanning and discovery ####
```R
# Search motif  
testSeq <- system.file("extdata/test.fa", package = "PEA")    
pdf("motifScan.pdf", height = 5, width = 5)    
results.scan <- CMRAnnotation(cmrSeq = testSeq, annotation = "motifScan")    
dev.off()    

# De-novo motif detection  
pdf("motifDetect.pdf", height = 5, width = 5)    
results.detect <- CMRAnnotation(cmrSeq = testSeq, annotation = "motifDetect")    
dev.off()    
```
#### Functional enrichment analysis of CMR corresponded genes ####
```R
#  GO enrichment analysis  
library(topGO)    
peaks <- G2T(bedPos = cmrMat, GTF = GTF)    
enrichment <- CMRAnnotation(cmrMat = peaks, GTF = GTF, annotation = "GO", topNodes = 20, dataset = "athaliana_eg_gene")
```
## Source codes availability ##
   The source codes of PEA are freely available at [PEA](<https://github.com/cma2015/PEA>)
## How to access help ##
* If users encounter any bugs or issues, feel free to leave a message at Github [issues](<https://github.com/cma2015/PEA/issues>). We will try our best to deal with all issues as soon as possible.
* In addition, if any suggestions are available, feel free to contact: __Zhai Jingjing__ <zhaijingjing603@gmail.com> or ___Ma Chuang___ <chuangma2006@gmail.com>
## References ##
  * Chen, W.*, et al.* Identifying N6-methyladenosine sites in the Arabidopsis
   thaliana transcriptome. *Mol Genet Genomics* 2016;291(6):2225-2229.  
  * Cui, H., Zhai, J. and Ma, C. miRLocator: machine learning-lased prediction
   of mature microRNAs within plant pre-miRNA sequences. *PLoS One*
   2015;10(11):e0142753.  
  * Ma, C.*, et al.* Machine learning-based differential network analysis: a
   study of stress-responsive transcriptomes in Arabidopsis. *Plant Cell*  2014;26(2):520-537.  
   Touw, W.G.*, et al.* Data mining in the life sciences with random forest: a
   walk in the park or lost in the jungle? *Brief Bioinform* 2013;14(3):315-326.  
  * Wang, C.L.*, et al.* PSoL: a positive sample only learning algorithm for
   finding non-coding RNA genes. *Bioinformatics* 2006;22(21):2590-2596.  
  * Zhou, Y.*, et al.* SRAMP: prediction of mammalian N6-methyladenosine (m6A)
   sites based on sequence-derived features. *Nucleic Acids Res*
   2016;44(10):e91.  
