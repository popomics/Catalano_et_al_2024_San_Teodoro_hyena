---
title: "Catalano et al 2024: San Teodoro hyena"
author: "Axel Barlow"
date: "2024-05-21"
output:
  html_document:
    keep_md: true
---



## Github repo

If you want to try out any of this code, you should clone the repo. You can knit the R markdown script (`San_Teodoro_hyena.Rmd`) in R studio to generate the `html`

```bash
git clone https://github.com/popomics/Catalano_et_al_2024_San_Teodoro_hyena.git
```

## Samples

Here's a list of samples used in the study


```r
my_samp <- read.table("./sample_inf/cro_info", header=TRUE)
kable(my_samp, align = "l")
```



|sample        |GB_data  |locality          |colour     |
|:-------------|:--------|:-----------------|:----------|
|c2518         |8.09396  |Zambia            |red        |
|ST-Copro_Iena |0.11303  |San_Teodoro       |yellow     |
|Ccsp043       |3.48269  |LindenthalerHohle |dodgerblue |
|c801          |10.79344 |Kenya             |red        |
|NamCrocuta    |76.09127 |Namibia           |red        |
|c4035         |3.39304  |Tanzania          |red        |
|cHyena2071    |25.68063 |Zoo               |red        |
|c793          |10.48718 |Kenya             |red        |
|c1979         |11.72082 |Botswana          |red        |
|Ccsp040       |8.20711  |AufhausenerHohle  |dodgerblue |
|cHyena375     |23.18486 |Zoo               |red        |
|Ccsp042       |5.01636  |LindenthalerHohle |dodgerblue |
|c5695         |4.54362  |Tanzania          |red        |
|c815          |9.94523  |Kenya             |red        |
|Ccsp041       |7.77150  |RussiaEast        |dodgerblue |
|Ghana_crocuta |76.14338 |Ghana             |red        |
|SAMN07212965  |35.48187 |striped_hyena     |black      |

## Data processing with `BEARCAVE`

We use the publicly available `BEARCAVE` scripts for Illumina data processing. To replicate the analyses, you'll need the following programs:

- `bwa v.0.7.17`
- `samtools v.1.3.1`
- `cutadapt v.1.18`
- `FLASH v.1.2.11`

```bash
# get the BEARCAVE2
git clone https://github.com/nikolasbasler/BEARCAVE.git
cd BEARCAVE/

# put fastqs in BEARCAVE/rawdata, with a separate folder for each sample. Folder name is sample name. For example:
ls -1 ./rawdata/*/

./rawdata/ST-Copro_Iena/:
6z0+ST-Copro_Iena_S15_R1_001.fastq.gz
6z0+ST-Copro_Iena_S15_R2_001.fastq.gz

# download reference genome to BEARCAVE/refgenomes
cd ../refgenomes
mkdir ./hyaena
cd ./hyaena
wget https://www.ebi.ac.uk/ena/browser/api/fasta/GCA_003009895.1?download=true&gzip=true

# index with the BEARCAVE script, first take a look at scripts and instructions
cd ../../scripts
less index_ref.sh
# run it
bash index_ref.sh ../refgenomes/hyaena/GCA_003009895.1_ASM300989v1_genomic.fa

# adapter trimming and PE read merging with the BEARCAVE2 script, first take a look at script and instructions
less trim_merge_DS_PE_standard.sh
# we'll process one of the San Teodora datasets as an example. Here are the files:
ls -1 ../rawdata/ST-Copro_Iena/:
6z0+ST-Copro_Iena_S15_R1_001.fastq.gz
6z0+ST-Copro_Iena_S15_R2_001.fastq.gz

# the script needs: 1. the sample name, 2. the prefix (3 characters preceding +), 3. the run name (first part of filename after prefix)
# Note there is a BEARCAVE script to add the prefix, or you can do it manually
bash trim_merge_DS_PE_standard.sh ST-Copro_Iena 6z0 ST-Copro_Iena_S15

# check results
less ../trimdata/ST-Copro_Iena_processing/6z0+ST-Copro_Iena_trim_report.log
less ../trimdata/ST-Copro_Iena_processing/6z0+ST-Copro_Iena_merge_report.log
# move files into BEARCAVE/trimdata
mv ../trimdata/ST-Copro_Iena_processing/*fastq* ../trimdata
mv ../trimdata/ST-Copro_Iena_processing/*log ../trimdata/trimlogs/
rmdir ../trimdata/ST-Copro_Iena_processing/

# mapping with the BEARCAVE script, first take a look at script and instructions
# Note this script just maps the merged reads (*mappable.fastq) as is suitable for aDNA. For modern data we additionally map the unmerged reads (*mappable_R1.fastq/*mappable_R2.fastq) using the map_modern_PE.sh script. The usage is identical. 
less map_SE.sh 
# the script needs 1. prefix, 2. reference (folder name in refgenomes), 3. three character taxon identifier (use spe = cave hyena), 4. sample name
bash map_SE.sh 6z0 hyaena spe ST-Copro_Iena
# check results
less ../mappedhyaena/6z0+ST-Copro_Iena_hyaena_map_processing/6z0+ST-Copro_Iena_hyaena_mapping.log
# move bam files into BEARCAVE2/mappedhyaena/
mv ../mappedhyaena/6z0+ST-Copro_Iena_hyaena_map_processing/*bam* ../mappedhyaena/
mv ../mappedhyaena/6z0+ST-Copro_Iena_hyaena_map_processing/*log ../mappedhyaena/hyaena_logs/
rmdir ../mappedhyaena/6z0+ST-Copro_Iena_hyaena_map_processing/
```

## DNA properties

Analysis of the San Teodoro ancient DNA sequence data: fragmentation, damage patterns, and metagenomic analysis.

`Mapdamage` analysis, example command

```bash
mapDamage --merge-reference-sequences --no-stats -y 0.6 -l 100 -i 6z0_dco+ST-Copro_Iena_cro_crocuta_.11303.bam -r GCA_008692635.1_BGI_CrCroc_1.0_genomic.fa
```

Metagenomic analysis in `fastqscreen`

```bash
fastq_screen 6z0+ST-Copro_Iena_S15_R1_001.fastq
```

Plotting results


```r
# read in results from mapdamage and fastqscreen
leng <- read.table("./dna_properties/lengths", header=TRUE)
ct_5p <- read.table("./dna_properties/5pCtoT_freq.txt", header=TRUE)
ga_3p <- read.table("./dna_properties/3pGtoA_freq.txt", header=TRUE)
my_fast <- read.table("./dna_properties/fastqscreen_results")

# set up plot
layout(matrix(c(1,2,3,4,4,4), nrow=2, ncol=3, byrow=TRUE))

# length distribution
par(mar=c(5,5,3,2))
plot(leng$length, leng$freq, type="h", 
     xlim=c(30,70), ylim=c(0,230000),
     xlab="Fragment length (bp)", ylab="frequency",
     axes=FALSE, frame.plot=TRUE,
     col="grey50", lwd=2
)
axis(1)
axis(2)
mtext(substitute(paste(bold("A."))), side=3, line=0.5, cex=1, adj=-0.275, cex.lab=1.5)

# C to T
par(mar=c(5,5,3,0.5))
plot(1:10, ct_5p$ct[1:10], type="l",
     col="red", lwd=2,
     xlab="5' position", ylab="Proportion C>T",
     axes=FALSE, frame.plot=TRUE
)
axis(1, at=c(1:10), cex.axis=1)
axis(2)
mtext(substitute(paste(bold("B."))), side=3, line=0.5, cex=1, adj=-0.275, cex.lab=1.5)

# G to A
par(mar=c(5,0.5,3,5))
plot(c(1:10), rev(ga_3p$ga[1:10]), type="l",
     col="blue", lwd=2,
     xlab="3' position", ylab="",
     axes=FALSE, frame.plot=TRUE
)
axis(1, at=c(1:10), labels=c(10:1), cex.axis=0.8)
axis(4)
mtext("Proportion G>A", side=4, line=3, cex=0.65)
mtext(substitute(paste(bold("C."))), side=3, line=0.5, cex=1, adj=-0.01, cex.lab=1.5)

# fastqscreen
par(mar=c(10,5,3,3))
colnames(my_fast) <- c("Apodemus sylvaticus", "Bos taurus", "Canis lupus", "Cervus elephas", "Crocuta crocuta", "Equus caballus", "Erinaceus europaeus", "Gallus gallus", "Lacerta agilis", "Loxodonta africana", "Microtus arvalis", "Myotis myotis", "Sus scrofa", "Vulpes vulpes", "Homo sapiens")
my_names <- c("One hit one genome", "Multiple hits one genome", "One hit multiple genomes", "Multiple hits multiple genomes")
my_mat <- as.matrix(my_fast)

barplot(height=my_mat, beside=TRUE, axes=FALSE, las=2, ylab="Percentage of reads",
	col=brewer.pal(4, "Set2")
)
axis(2)
legend(x=58, y=1.4, inset=0.03, legend=my_names, pch=22, pt.bg=brewer.pal(4, "Set2"), pt.cex=1.5, bty="n")
mtext(substitute(paste(bold("D."))), side=3, line=0.5, cex=1, adj=-0.06, cex.lab=1.5)
```

<img src="San_Teodoro_hyena_files/figure-html/unnamed-chunk-1-1.png" width="100%" style="display: block; margin: auto;" />

# Relationships

Analysis of Crocuta relationships: PCA, NJ tree, topology tests

Generating covariance matrix in `angsd`

```bash
# making bamlist
cat > bamlist_croc
430+c2518_cro_hyaena_7.40266.bam
6z0_dco+ST-Copro_Iena_cro_hyaena_.11410.bam
88a+Ccsp043_cro_hyaena_3.37216.bam
8Ln+c801_cro_hyaena_9.90712.bam
977+NamCrocuta_cro_hyaena_67.07102.bam
9z1+c4035_cro_hyaena_3.04918.bam
b4b+cHyena2071_cro_hyaena_23.19858.bam
e6r+c793_cro_hyaena_9.40624.bam
f62+c1979_cro_hyaena_10.58993.bam
i01+Ccsp040_cro_hyaena_7.88971.bam
Ltv+cHyena375_cro_hyaena_20.67419.bam
mj3+Ccsp042_cro_hyaena_4.84953.bam
mre+c5695_cro_hyaena_4.09332.bam
rzc+c815_cro_hyaena_9.10468.bam
s72+Ccsp041_cro_hyaena_7.51147.bam
shq+Ghana_crocuta_cro_hyaena_67.35423.bam

# making list of scaffolds > 1 Mb for filtering
# I use my length_doct.R script
git clone https://github.com/drabarlow/length_doct.R.git

# get scaffold lengths with awk
cat GCA_003009895.1_ASM300989v1_genomic.fa | awk '{print $1}' | awk '$0 ~ ">" {print c; c=0;printf substr($0,2,100) "\t"; } $0 !~ ">" {c+=length($0);} END { print c; }' > scaffold_lengths.txt 

# run R script
Rscript length_doct.R scaffold_lengths.txt
# generates list in the file list_over_1mb.txt

# angsd
angsd -doIBS 1 -doCov 1 -makeMatrix 1 -doCounts 1 -doMajorMinor 1 -GL 1 -minFreq 0.1 -minInd 16 -rmTrans 1 -minQ 30 -minMapQ 30 -b bamlist_croc -ref GCA_003009895.1_ASM300989v1_genomic.fa -rf list_over_1mb.txt -out bamlist_croc -nThreads 10
```

Generating distance matrix in `angsd`

```bash
#making bamlist
cat > bamlist_out
430+c2518_cro_hyaena_7.40266.bam
6z0_dco+ST-Copro_Iena_cro_hyaena_.11410.bam
88a+Ccsp043_cro_hyaena_3.37216.bam
8Ln+c801_cro_hyaena_9.90712.bam
977+NamCrocuta_cro_hyaena_67.07102.bam
9z1+c4035_cro_hyaena_3.04918.bam
b4b+cHyena2071_cro_hyaena_23.19858.bam
e6r+c793_cro_hyaena_9.40624.bam
f62+c1979_cro_hyaena_10.58993.bam
i01+Ccsp040_cro_hyaena_7.88971.bam
k3e+SAMN07212965_hya_hyaena_35.48187.bam
Ltv+cHyena375_cro_hyaena_20.67419.bam
mj3+Ccsp042_cro_hyaena_4.84953.bam
mre+c5695_cro_hyaena_4.09332.bam
rzc+c815_cro_hyaena_9.10468.bam
s72+Ccsp041_cro_hyaena_7.51147.bam
shq+Ghana_crocuta_cro_hyaena_67.35423.bam

#angsd
angsd -doIBS 1 -makeMatrix 1 -doCounts 1 -doMajorMinor 1 -GL 1 -minFreq 0.1 -minInd 17 -rmTrans 1 -minQ 30 -minMapQ 30 -b bamlist_out -ref GCA_003009895.1_ASM300989v1_genomic.fa -rf list_over_1mb.txt -out bamlist_out -nThreads 10 
```

D stat analysis for generating counts of incongruent sites for topology tests

```bash
#angsd
angsd -doAbbababa 1 -rmTrans 1 -blockSize 5000000 -doCounts 1 -minQ 30 -minMapQ 30 -anc GCA_003009895.1_ASM300989v1_genomic.fa -ref GCA_003009895.1_ASM300989v1_genomic.fa -rf list_over_1mb.txt -b bamlist_croc -out dstat_bamlist_croc -nThreads 20

# generating output with R script that ships with angsd
Rscript jackknife.R file=dstat_bamlist_croc.abbababa indNames=bamlist_croc
```

Plotting


```r
# read in data
my_cov <- read.table("./relationships/bamlist_croc.covMat")
my_dmat <- read.table("./relationships/bamlist_out.ibsMat")
my_matrix <- as.dist(as(my_dmat, "matrix"))
my_dat <- read.table("./relationships/topo") # I pulled out the ABBA/BABA counts from the angsd output

# set up plot
par(mfrow=c(1,3))

# pca
my_pca <- eigen(my_cov)
pc1_var <- (my_pca$values[1]/sum(my_pca$values)) * 100
pc2_var <- (my_pca$values[2]/sum(my_pca$values)) * 100

par(mar=c(5,5,3,2))

plot(my_pca$vectors[,1], my_pca$vectors[,2],
    xlab=paste("PC1:", signif(pc1_var,3), "% variation"),
    ylab=paste("PC2:", signif(pc2_var,3), "% variation"),
    xlim=c(-0.45, 0.25), ylim=c(-0.45, 0.5),
	pch=21, bg=my_samp$colour, cex=1.2
)

leglab=c("Sicily", "cave","spotted")
legend("topleft", inset=0.03, legend=leglab, pch=21, pt.bg=c("yellow", "dodgerblue", "red"), pt.cex=1.2) 

mtext(substitute(paste(bold("A."))), side=3, line=0.5, cex=1, adj=-0.275, cex.lab=1.5)

# phylo
my_tree <- nj(my_matrix)
my_root <- root(my_tree, outgroup=11)

my_labels=c("Zambia",
"Sicily",
"Lindenthaler",
"Kenya",
"Namibia",
"Tanzania",
"Zoo",
"Kenya",
"Botswana",
"Aufhausener",
"striped hyena",
"Zoo",
"Lindenthaler",
"Tanzania",
"Kenya",
"Geo. Soc.",
"Ghana")

my_col=c("red", "yellow", "dodgerblue", "red", "red", "red", "red", "red", "red", "dodgerblue", "black", "red", "dodgerblue", "red", "red", "dodgerblue", "red")

par(mar=c(4,2,3,5), xpd=TRUE)

plot(my_root, show.tip.label=F, edge.width=3)
tiplabels(my_labels, frame="none", adj=-0.2)
tiplabels(pch=21, bg=my_col, cex=1.2)

mtext(substitute(paste(bold("B."))), side=3, line=0.5, cex=1, adj=0, cex.lab=1.5)

# topology tests
colnames(my_dat) <- c("tr1", "tr2", "tr3", "tr4", "tr5", "tr6")
rownames(my_dat) <- c("top1", "top2", "top3")
my_mat <- as.matrix(my_dat)

par(mar=c(4,5,4,2))

barplot(height=my_mat, beside=TRUE, axes=FALSE, ylab="Incongruent sites (thousands)",
	col=c("yellow", "dodgerblue", "dodgerblue")
)
axis(2, at=c(0,2e4,4e4,6e4,8e4,10e4,12e4,14e4), labels=c(0,20,40,60,80,100,120,140))

leglab=c("Sicily basal","Other cave basal")
legend(x=12.5, y=165000, inset=0.03, legend=leglab, pch=22, pt.bg=c("yellow", "dodgerblue"), pt.cex=1.2, bty="n")

mtext(substitute(paste(bold("C."))), side=3, line=1.45, cex=1, adj=-0.275, cex.lab=1.5)
```

<img src="San_Teodoro_hyena_files/figure-html/unnamed-chunk-2-1.png" width="100%" style="display: block; margin: auto;" />

## Admixture

We use the `angsd` results generated above, they just need plotting


```r
# I just extracted the comparisons I want from the angsd output, and saved in ./admixture/dstat_tidy
# There's an extra column "comp" which is the y axis coordinate
my_d <- read.table("./admixture/dstat_tidy", header=TRUE)

# set up plot
par(mar=c(4,6,1,1))

# plot
plot(abs(my_d$Dstat), jitter(my_d$comp),
	pch=21, cex=1, bg=ifelse(abs(my_d$Z) > 3, "red", "white"),
	axes=FALSE, xlab="", ylab="", xlim=c(0,0.2)
)

my_sig <- c("sig. N = 9", "sig. N = 28", "sig. N = 30", "sig. N = 28", "sig. N = 25")
text(rep(0.12, 5), 5:9, labels=my_sig, cex=0.8)

names <- c(
"Lindenthaler", 
"Aufhausener", 
"Lindenthaler", 
"Geo. Soc.", 
"Sicily", 
"Lindenthaler", 
"Aufhausener", 
"Lindenthaler", 
"Geo. Soc."
)

axis(2, at=1:9, lwd=-1, las=1, labels=names, cex.axis=0.8)

axis(1, cex.axis=0.8)
mtext("D statistic", side=1, line=2.5, cex=1)
```

<img src="San_Teodoro_hyena_files/figure-html/unnamed-chunk-3-1.png" width="50%" style="display: block; margin: auto;" />
