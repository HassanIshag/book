--- 
title: "Computational Genomics with R"
author: "Altuna Akalin"
date: "2020-09-30"
knit: "bookdown::render_book"
documentclass: krantz
classoption: numberinsequence,krantz2
bibliography: [book.bib]
biblio-style: [apalike.bst]
csl: chicago-manual.csl
link-citations: yes
colorlinks: yes
fontsize: 12pt
monofont: "Source Code Pro"
monofontoptions: "Scale=0.7"
site: bookdown::bookdown_site
description: "A guide to computationa genomics using R. The book covers fundemental topics with practical examples for an interdisciplinery audience"
url: 'https\://compmgenomr.github.io/book/'
github-repo: compgenomr/book
cover-image: images/cover.jpg
---



# Preface {-}


<a href="" target="_blank"><img src="images/cover.jpg" style="display: block; margin: auto;" /></a>

The aim of this book is to provide the fundamentals for data analysis for genomics. We developed this book based on the computational genomics courses we are giving every year. We have had invariably an interdisciplinary audience with backgrounds from physics, biology, medicine, math, computer science or other quantitative fields. We want this book to be a starting point for computational genomics students and a guide for further data analysis in more specific topics in genomics. This is why we tried to cover a large variety of topics from programming to basic genome biology. As the field is interdisciplinary, it requires different starting points for people with different backgrounds. A biologist might skip sections on basic genome biology and start with R programming, whereas a computer scientist might want to start with genome biology. In the same manner, a more experienced person might want to refer to this book when needing to do a certain type of analysis, but having no prior experience.


![Creative Commons License](images/by-nc-sa.png)  
The online version of this book is licensed under the [Creative Commons Attribution-NonCommercial-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-nc-sa/4.0/). 

## Who is this book for? {-}
The book contains practical and theoretical aspects of computational genomics. Biology and 
medicine generate more data than ever before. Therefore, we need to educate more people with data 
analysis skills and understanding of computational genomics.
Since computational genomics is interdisciplinary, this book aims to be accessible for
biologists, medical scientists, computer scientists and people from other quantitative backgrounds. We wrote this book for the following audiences:

 - Biologists and medical scientists who generate the data and are keen on analyzing it themselves.
 - Students and researchers who are formally starting to do research on or using computational genomics do not have extensive domain-specific knowledge, but have at least a beginner-level understanding in a quantitative field, for example, math, stats.
 - Experienced researchers looking for recipes or quick how-to's to get started in specific data analysis tasks related to computational genomics. 


### What will you get out of this?  {-}
This resource describes the skills and provides how-to's that will help readers 
analyze their own genomics data.

After reading:

- If you are not familiar with R, you will get the basics of R and dive right in to specialized uses of R for computational genomics.
- You will understand genomic intervals and operations on them, such as overlap.
- You will be able to use R and its vast package library to do sequence analysis, such as calculating GC content for given segments of a genome or find transcription factor binding sites.
- You will be familiar with visualization techniques used in genomics, such as heatmaps, meta-gene plots, and genomic track visualization.
- You will be familiar with supervised and unsupervised learning techniques which are important in data modeling and exploratory analysis of high-dimensional data.
- You will be familiar with analysis of different high-throughput sequencing datasets (RNA-seq, ChIP-seq, BS-seq and multi-omics integration) mostly using R-based tools.



## Structure of the book {-}
The book is designed with the idea that practical and conceptual
understanding of data analysis methods is as important, if not more important, than the theoretical understanding, such as detailed derivation of equations in statistics or machine learning. That is why we first try to give a conceptual explanation of the concepts then we try to give essential parts of the mathematical formulas for more detailed understanding. In this spirit, we always show and 
explain the code for a particular data analysis task. We also give additional references such as books, websites, video lectures and scientific papers for readers who desire to gain deeper theoretical understanding of data analysis-related methods or concepts.

Chapter \@ref(intro): "Introduction to Genomics" introduces the basic concepts in genome biology and genomics. Understanding these concepts is important for computational genomics. 

Chapter \@ref(Rintro): "Introduction to R for Genomic Data Analysis" provides the basic R skills necessary to follow the book in addition to common data analysis paradigms we observe in genomic data analysis.  Chapter \@ref(stats): "Statistics for Genomics", Chapter \@ref(unsupervisedLearning): "Exploratory Data Analysis with Unsupervised Machine Learning" and Chapter \@ref(supervisedLearning): "Predictive Modeling with Supervised Machine Learning" introduce the necessary quantitative skills that one will need when analyzing high-dimensional genomics data.

Chapter \@ref(genomicIntervals): "Operations on Genomic Intervals and Genome Arithmetic" introduces the fundamental tools for dealing with genomic intervals and their relationship to each other over the genome. In addition, the chapter introduces a variety of genomic data visualization methods. The skills introduced in this chapter are key skills that are needed to work with processed genomic data which are available through public databases such as Ensembl and the UCSC browser.

The next chapters deal with specific analysis of high-throughput sequencing data and integrating different kinds of datasets. Chapter \@ref(processingReads): "Quality Check, Processing and Alignment of High-throughput Sequencing Reads" introduces quality checks that need to be done on sequencing reads and different ways to process them further. Chapters \@ref(rnaseqanalysis), \@ref(chipseq) and \@ref(bsseq) deal with RNA-seq analysis, ChIP-seq analysis and BS-seq analysis. The last chapter, Chapter \@ref(multiomics):"Multi-omics Analysis" deals with methods for integrating multiple omics datasets.

Most chapters have exercises that reinforce some of the important points introduced in the chapters. The exercises are classified into beginner, intermediate and advanced categories. If you are well versed in a certain subject you might want to skip beginner-level exercises.

To sum it up, this book is a comprehensive guide for computational genomics. Some sections are there for the sake of the wide interdisciplinary audience and completeness, and not all sections will be equally useful to all readers of this broad audience. 

## Software information and conventions {-}

Package names and inline code and file names are formatted in a typewriter font (e.g. `methylKit`). Function names are followed by parentheses (e.g. `genomation::ScoreMatrix()`). The double-colon operator `::` means accessing an object from a package.

### Assignment operator convention {-}
Traditionally, `<-` is the preferred assignment operator. However, throughout the book we use `=` and `<-` as the assignment operator interchangeably. 

### Packages needed to run the book code {-}
This book is primarily about using R packages to analyze genomics data, therefore if you want to reproduce the analysis in this book you need to install the relevant packages in each chapter using `install.packages` or `BiocManager::install` functions. In each chapter, we load the necessary packages with the `library()` or `require()` function when we use the needed functions from respective packages. By looking at calls, you can see which packages are needed for that code chunk or chapter. If you need to install all the package dependencies for the book, you can run the following command and have a cup of tea while waiting.

```r
if (!requireNamespace("BiocManager", quietly = TRUE))
    install.packages("BiocManager")
BiocManager::install(c('qvalue','plot3D','ggplot2','pheatmap','cowplot',
                      'cluster', 'NbClust', 'fastICA', 'NMF','matrixStats',
                      'Rtsne', 'mosaic', 'knitr', 'genomation',
                      'ggbio', 'Gviz', 'DESeq2', 'RUVSeq',
                      'gProfileR', 'ggfortify', 'corrplot',
                      'gage', 'EDASeq', 'citr', 'formatR',
                      'svglite', 'Rqc', 'ShortRead', 'QuasR',
                      'methylKit','FactoMineR', 'iClusterPlus',
                      'enrichR','caret','xgboost','glmnet',
                      'DALEX','kernlab','pROC','nnet','RANN',
                      'ranger','GenomeInfoDb', 'GenomicRanges',
                      'GenomicAlignments', 'ComplexHeatmap', 'circlize', 
                      'rtracklayer', 'BSgenome.Hsapiens.UCSC.hg38',
                      'BSgenome.Hsapiens.UCSC.hg19','tidyr',
                      'AnnotationHub', 'GenomicFeatures', 'normr',
                      'MotifDb', 'TFBSTools', 'rGADEM', 'JASPAR2018'
                     ))
```


## Data for the book {-}
We rely on data from different R and Bioconductor packages throughout the book. For the datasets that do not ship with those packages, we created our own package [**compGenomRData**](https://github.com/compgenomr/compGenomRData). You can install this package via `devtools::install_github("compgenomr/compGenomRData")`. We use the `system.file()` function to get the path to the files. We noticed many inexperienced users are confused about this function. This function just outputs the full path to the file that is installed with the data package. 

## Exercises in the book {-}
There is a set of exercises at the end of each chapter. The exercises are 
separated in thematic sections that follow the major sections in the chapter.
In addition, each exercise is classified based on its difficulty as "Beginner",
"Intermediate" and "Advanced". Beginner-level exercises can usually be done
by refactoring the code in the chapter. Advanced-level exercises usually require
a combination of code from different sections or chapters. The intermediate level
is somewhere in between. The solutions to the exercises are available at
https://github.com/compgenomr/exercises. 

## Reproducibility statement {-}
This book is compiled with R 4.0.0 and the following packages. We only list the main packages and their versions but not their dependencies. 


```
## qvalue_2.20.0 | plot3D_1.3 | ggplot2_3.3.1 | pheatmap_1.0.12
## cowplot_1.0.0 | cluster_2.1.0 | NbClust_3.0 | fastICA_1.2.2
## NMF_0.23.0 | matrixStats_0.56.0 | Rtsne_0.15 | mosaic_1.7.0
## knitr_1.28 | genomation_1.20.0 | ggbio_1.36.0 | Gviz_1.32.0
## DESeq2_1.28.1 | RUVSeq_1.22.0 | gProfileR_0.7.0 | ggfortify_0.4.10
## corrplot_0.84 | gage_2.37.0 | EDASeq_2.22.0 | citr_0.3.2
## formatR_1.7 | svglite_1.2.3 | Rqc_1.22.0 | ShortRead_1.46.0
## QuasR_1.28.0 | methylKit_1.14.2 | FactoMineR_2.3 | iClusterPlus_1.24.0
## enrichR_2.1 | caret_6.0.86 | xgboost_1.0.0.2 | glmnet_4.0
## DALEX_1.2.1 | kernlab_0.9.29 | pROC_1.16.2 | nnet_7.3.14
## RANN_2.6.1 | ranger_0.12.1 | GenomeInfoDb_1.24.0 | GenomicRanges_1.40.0
## GenomicAlignments_1.24.0 | ComplexHeatmap_2.4.2 | circlize_0.4.9 | rtracklayer_1.48.0
## tidyr_1.1.0 | AnnotationHub_2.20.0 | GenomicFeatures_1.40.0 | normr_1.14.0
## MotifDb_1.30.0 | TFBSTools_1.26.0 | rGADEM_2.36.0 | JASPAR2018_1.1.1
## BSgenome.Hsapiens.UCSC.hg38_1.4.3 | BSgenome.Hsapiens.UCSC.hg19_1.4.3
```

## Acknowledgements {-}

I wish to thank the R and Bioconductor communities for developing and maintaining libraries for genomic data analysis. Without their constant work and dedication, writing such a book would not be possible.

I also wish to thank all my past and present mentors, colleagues and employers.
The interaction with them provided the motivation to write such a book, and organize and teach hands-on courses on computational genomics.

I wish to thank John Kimmel, the editor from Chapman & Hall/CRC, who helped me publish this book. It was a pleasure to work with him. He generously agreed to let me keep the online version of this book, so I can continue updating it after it is printed.

This has been a long journey for me. I started writing parts of this book as early as 2013. If it wasn't for Vedran Franke, Bora Uyar and Jonathan Ronen, it would have taken even longer. They kindly agreed to contribute the missing chapters and they did a great job. I am thankful for their contributions.

The following people kindly contributed fixes for typos and code, and various suggestions: Thomas Schalch, Alex Gosdschan, Rodrigo Ogava, Fei Zhao, Jonathan Kitt, Janani Ravi, Christian Schudoma, Samuel Sledzieski, Dania Hamo and Sarvesh Nikumbh.
 
\BeginKnitrBlock{flushright}<p class="flushright">Altuna Akalin  
Berlin, Germany</p>\EndKnitrBlock{flushright}


\frontmatter

## How to contribute {-}

We have been developing this book on github and your contributions are very welcome. We believe more eyes looking at this will increase the quality. There are a number of ways you can help make the
book better:

* If you don't understand something, please 
  [let Altuna know](mailto:aakalin@gmail.com). Your feedback on what is confusing
  or hard to understand is valuable.

* If you spot a typo, feel free to edit the underlying page and send a pull 
  request. If you've never done this before, the process is very easy: 
  
    * Click the edit this page on the top bar of the book website. The icon is near the search and download icons on the top of the page.
  
    * Make the changes using GitHub's in-page editor (fixing typos is
      perfectly adequate).
      
    * Include a brief description of your changes in the area below the
      editor, then propose the file change. If you make significant changes,
      include the phrase "I assign the copyright of this contribution to
      Altuna Akalin" - We need this so I can publish the printed book.
    
    * Create a pull request.




