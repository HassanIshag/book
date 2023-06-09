
# Multi-omics Analysis {#multiomics}


*Chapter Author*: **Jonathan Ronen**



\index{multi-omics}Living cells are a symphony of complex processes. Modern sequencing technology has led to many comprehensive assays being routinely available to experimenters, giving us different ways to peek at the internal doings of the cells, each experiment revealing a different part of some underlying processes. As an example, most cells have the same DNA, but sequencing the genome of a cell allows us to find mutations and structural alterations that drive tumerogenesis in cancer. If we treat the DNA with bisulfite prior to sequencing, cytosine residues are converted to uracil, but 5-methylcytosine residues are unaffected. This allows us to probe the methylation patterns of the genome, or its methylome. By sequencing the mRNA molecules in a cell, we can calculate the abundance, in different samples, of different mRNA transcripts, or uncover its transcriptome. Performing different experiments on the same samples, for instance RNA-seq, DNA-seq, and BS-seq, results in multi-dimensional omics datasets, which enable the study of relationships between different biological processes, e.g. DNA methylation, mutations, and gene expression, and the leveraging of multiple data types to draw inferences about biological systems. This chapter provides an overview of some of the available methods for such analyses, focusing on matrix factorization approaches. In the examples in this chapter we will demonstrate how these methods are applicable to cancer molecular subtyping, i.e. finding tumors which are driven by the same molecular processes.

## Use case: Multi-omics data from colorectal cancer

\index{multi-omics}\index{colorectal cancer}The examples in this chapter will use the following data: a set of 121 tumors from the TCGA [@tcga_pan_cancer] colorectal cancer cohort. The tumors have been profiled for gene expression using RNA-seq, mutations using Exome-seq, and copy number variations using genotyping arrays. Projects such as TCGA have turbocharged efforts to sub-divide cancer into subtypes. Although two tumors arise in the colon, they may have distinct molecular profiles, which is important for treatment decisions. The subset of tumors used in this chapter belong to two distinct molecular subtypes defined by the Colorectal Cancer Subtyping Consortium [@cmscc], _CMS1_ and _CMS3_. The following code snippets load this multi-omics data from the companion package, starting with gene expression data from RNA-seq (see Chapter \@ref(rnaseqanalysis)). Below we are reading the RNA-seq data from the `compGenomRData` package.

```r
# read in the csv from the companion package as a data frame
csvfile <- system.file("extdata", "multi-omics", "COREAD_CMS13_gex.csv", 
                       package="compGenomRData")
x1 <- read.csv(csvfile, row.names=1)
# Fix the gene names in the data frame
rownames(x1) <- sapply(strsplit(rownames(x1), "\\|"), function(x) x[1])
# Output a table
knitr::kable(head(t(head(x1))), caption="Example gene expression data (head)")
```



Table: (\#tab:moloadMultiomicsGE)Example gene expression data (head)

                 RNF113A    S100A13      AP3D1   ATP6V1G1     UBQLN4      TPPP3
-------------  ---------  ---------  ---------  ---------  ---------  ---------
TCGA.A6.2672    21.19567   19.72600   11.53022    0.00000   15.35637   12.76747
TCGA.A6.3809    21.50866   18.65729   12.98830   14.12675   19.62208    0.00000
TCGA.A6.5661    20.08072   18.97034   10.83759   15.31325    0.00000    0.00000
TCGA.A6.5665     0.00000   11.88336   10.24248   19.79300    0.00000    0.00000
TCGA.A6.6653     0.00000   12.07753    0.00000    0.00000    0.00000    0.00000
TCGA.A6.6780     0.00000   12.99128    0.00000   19.96976   13.17618   11.58742
Table \@ref(tab:moloadMultiomicsGE) shows the head of the gene expression matrix. The rows correspond to patients, referred to by their TCGA identifier, as the first column of the table. Columns represent the genes, and the values are RPKM expression values. The column names are the names or symbols of the genes.
The details about how these expression values are calculated are in Chapter \@ref(rnaseqanalysis).


We first **read mutation data** with the following code snippet.

```r
# read in the csv from the companion package as a data frame
csvfile <- system.file("extdata", "multi-omics", "COREAD_CMS13_muts.csv", 
                       package="compGenomRData")
x2 <- read.csv(csvfile, row.names=1)
# Set mutation data to be binary (so if a gene has more than 1 mutation,
# we only count one)
x2[x2>0]=1
# output a table
knitr::kable(head(t(head(x2))), caption="Example mutation data (head)")
```



Table: (\#tab:moloadMultiomicsMUT)Example mutation data (head)

                TTN   TP53   APC   KRAS   SYNE1   MUC16
-------------  ----  -----  ----  -----  ------  ------
TCGA.A6.2672      1      0     0      0       1       1
TCGA.A6.3809      1      0     0      0       0       0
TCGA.A6.5661      1      0     0      0       1       1
TCGA.A6.5665      1      0     0      0       1       1
TCGA.A6.6653      1      0     0      1       0       0
TCGA.A6.6780      1      0     0      0       0       1
Table \@ref(tab:moloadMultiomicsMUT) shows the mutations of these tumors (mutations were introduced in Chapter \@ref(intro)). In the mutation matrix, each cell is a binary 1/0, indicating whether or not a tumor has a non-synonymous mutation in the gene indicated by the column. These types of mutations change the aminoacid sequence, therefore they are likely to change the function of the protein.

Next, we **read copy number data** with the following code snippet.

```r
# read in the csv from the companion package as a data frame
csvfile <- system.file("extdata", "multi-omics", "COREAD_CMS13_cnv.csv", 
                       package="compGenomRData")
x3 <- read.csv(csvfile, row.names=1)
# output a table
knitr::kable(head(t(head(x3))), 
             caption="Example copy number data for CRC samples")
```



Table: (\#tab:moloadMultiomicsCNV)Example copy number data for CRC samples

                8p23.2   8p23.3   8p23.1   8p21.3   8p12   8p22
-------------  -------  -------  -------  -------  -----  -----
TCGA.A6.2672         0        0        0        0      0      0
TCGA.A6.3809         0        0        0        0      0      0
TCGA.A6.5661         0        0        0        0      0      0
TCGA.A6.5665         0        0        0        0      0      0
TCGA.A6.6653         0        0        0        0      0      0
TCGA.A6.6780         0        0        0        0      0      0
Finally, table \@ref(tab:moloadMultiomicsCNV) shows GISTIC scores [@mermel2011gistic2] for copy number alterations in these tumors. During transformation from healthy cells to cancer cells, the genome sometimes undergoes large-scale instability; large segments of the genome might be replicated or lost. This will be reflected in each segment's "copy number". In this matrix, each column corresponds to a chromosome segment, and the value of the cell is a real-valued score indicating if this segment has been amplified (copied more) or lost, relative to a non-cancer control from the same patient.

Each of the data types (gene expression, mutations, copy number variation) on its own, provides some signal which allows us to somewhat separate the samples into the two different subtypes. In order to explore these relations, we must first obtain the subtypes of these tumors. The following code snippet reads these, also from the companion package:


```r
# read in the csv from the companion package as a data frame
csvfile <- system.file("extdata", "multi-omics", "COREAD_CMS13_subtypes.csv",
                       package="compGenomRData")
covariates <- read.csv(csvfile, row.names=1)
# Fix the TCGA identifiers so they match up with the omics data
rownames(covariates) <- gsub(pattern = '-', replacement = '\\.',
                             rownames(covariates))
covariates <- covariates[colnames(x1),]
# create a dataframe which will be used to annotate later graphs
anno_col <- data.frame(cms=as.factor(covariates$cms_label))
rownames(anno_col) <- rownames(covariates)
```

Before proceeding with any multi-omics integration analysis which might obscure the underlying data, it is important to take a look at each omic data type on its own, and in this case in particular, to examine their relation to the underlying condition, i.e. the cancer subtype. A great way to get an eagle-eye view of such large data is using heatmaps (see Chapter \@ref(unsupervisedLearning) for more details).

We will first check the gene expression data in relation to the subtypes. One way of doing that is plotting a heatmap and clustering the tumors, while displaying a color annotation atop the heatmap, indicating which subtype each tumor belongs to. This is shown in Figure \@ref(fig:mogeneExpressionHeatmap), which is generated by the following code snippet:

```r
pheatmap::pheatmap(x1,
                   annotation_col = anno_col,
                   show_colnames = FALSE,
                   show_rownames = FALSE,
                   main="Gene expression data")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/mogeneExpressionHeatmap-1.png" alt="Heatmap of gene expression data for colorectal cancers." width="60%" />
<p class="caption">(\#fig:mogeneExpressionHeatmap)Heatmap of gene expression data for colorectal cancers.</p>
</div>

In Figure \@ref(fig:mogeneExpressionHeatmap), each column is a tumor, and each row is a gene. The values in the cells are FPKM values. There is another band above the heatmap annotating each column (tumor) with its corresponding subtype. The tumors are clustered using hierarchical clustering denoted by the dendrogram above the heatmap, according to which the columns (tumors) are ordered. While this ordering corresponds somewhat to the subtypes, it would not be possible to cut this dendrogram in a way which achieves perfect separation between the subtypes.

Next we repeat the same exercise using the mutation data. The following snippet generates Figure \@ref(fig:momutationsHeatmap):

```r
pheatmap::pheatmap(x2,
                   annotation_col = anno_col,
                   show_colnames = FALSE,
                   show_rownames = FALSE,
                   main="Mutation data")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/momutationsHeatmap-1.png" alt="Heatmap of mutation data for colorectal cancers." width="50%" />
<p class="caption">(\#fig:momutationsHeatmap)Heatmap of mutation data for colorectal cancers.</p>
</div>

An examination of Figure \@ref(fig:momutationsHeatmap) shows that tumors clustered and ordered by mutation data correspond very closely to their CMS subtypes. However, one should be careful in drawing conclusions about this result. Upon closer examination, you might notice that the separating factor seems to be that CMS1 tumors have significantly more mutations than do CMS3 tumors. This, rather than mutations in a specific genes, seems to be driving this clustering result. Nevertheless, this hyper-mutated status is an important indicator for this subtype.

Finally, we look into copy number variation data and try to see if clustered samples are in concordance with subtypes. The following code snippet generates Figure \@ref(fig:moCNVHeatmap):

```r
pheatmap::pheatmap(x3,
                   annotation_col = anno_col,
                   show_colnames = FALSE,
                   show_rownames = FALSE,
                   main="Copy number data")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/moCNVHeatmap-1.png" alt="Heatmap of copy number variation data, colorectal cancers." width="50%" />
<p class="caption">(\#fig:moCNVHeatmap)Heatmap of copy number variation data, colorectal cancers.</p>
</div>

The interpretation of Figure \@ref(fig:moCNVHeatmap) is left as an exercise for the reader.

It is clear that while there is some "signal" in each of these omics types, as is evident from these heatmaps, it is equally clear that none of these omics types completely and on its own explains the subtypes. Each omics type provides but a glimpse into what makes each of these tumors different from a healthy cell. Through the rest of this chapter, we will demonstrate how analyzing the gene expression, mutations, and copy number variations, in tandem, we will be able to get a better picture of what separates these cancer subtypes.

The next section will describe latent variable models for multi-omics integrations. Latent variable models are a form of dimensionality reduction (see Chapter \@ref(unsupervisedLearning)). Each omics data type is "big data" in its own right; a typical RNA-seq experiment profiles upwards of 50 thousand different transcripts. The difficulties in handling large data matrices are only exacerbated by the introduction of more omics types into the analysis, as we are suggesting here. In order to overcome these challenges, latent variable models are a powerful way to reduce the dimensionality of the data down to a manageable size.

## Latent variable models for multi-omics integration

\index{unsupervised learning}Unsupervised multi-omics integration methods are methods that look for patterns within and across data types, in a label-agnostic fashion, i.e. without knowledge of the identity or label of the analyzed samples (e.g. cell type, tumor/normal). This chapter focuses on latent variable models, a form of dimensionality reduction technique (see Chapter \@ref(unsupervisedLearning)). Latent variable models make an assumption that the high-dimensional data we observe (e.g. counts of tens of thousands of mRNA molecules) arise from a lower dimension description. The variables in that lower dimensional description are termed _latent variables_, as they are believed to be latent in the data, but not directly observable through experimentation. Therefore, there is a need for methods to infer the latent variables from the data. For instance, (see Chapter \@ref(rnaseqanalysis) for details of RNA-seq analysis) the relative abundance of different mRNA molecules in a cell is largely determined by the cell type. There are other experiments which may be used to discern the cell type of cells (e.g. looking at them under a microscope), but an RNA-seq experiment does not, directly, reveal whether the analyzed sample was taken from one organ or another. A latent variable model would set the cell type as a latent variable, and the observable abundance of mRNA molecules to be dependent on the value of the latent variable (e.g. if the latent variable is "Regulatory T-cell", we would expect to find high expression of CD4, FOXP3, and CD25).
  
## Matrix factorization methods for unsupervised multi-omics data integration

\index{dimensionality reduction}\index{matrix factorization}Matrix factorization techniques attempt to infer a set of latent variables from the data by finding factors of a data matrix. Principal Component Analysis (introduced in Chapter \@ref(unsupervisedLearning)) is a form of matrix factorization which finds factors based on the covariance structure of the data. Generally, matrix factorization methods may be formulated as


$$
X = WH,
$$
where $X$ is the _data matrix_, $[M \times N]$ where $M$ is the number of features (typically genes), and $N$ is the number of samples. $W$ is an $[M \times K]$ _factors_ matrix, and $H$ is the $[K \times N]$ _latent variable coefficient matrix_. Tying this back to PCA, where $X = U \Sigma V^T$, we may formulate the factorization in the same terms by setting $W=U\Sigma$ and $H=V^T$. If $K=rank(X)$, this factorization is lossless, i.e. $X=WH$. However if we choose $K<rank(X)$, the factorization is lossy, i.e. $X \approx WH$. In that case, matrix factorization methods normally opt to minimize the error

$$
min~\|X-WH\|.
$$
As we normally seek a latent variable model with a considerably lower dimensionality than $X$, this is the more common case.

The loss function we choose to minimize may be further subject to some constraints or regularization terms\index{regularization}\index{loss function}\index{optimization}. Regularization has been introduced in Chapter \@ref(supervisedLearning). In the current context of latent factor models, a regularization term might be added to the loss function, i.e. we might choose to minimize $min~\|X-WH\| + \lambda \|W\|^2$ (this is called $L_2$-regularization) instead of merely the reconstruction error. Adding such a term to our loss function here will push the $W$ matrix entries towards 0, in effect balancing between better reconstruction of the data and a more parsimonious model. A more parsimonious latent factor model is one with more sparsity in the latent factors. This sparsity is desirable for model interpretation, as will become evident in later sections.

<div class="figure" style="text-align: center">
<img src="images/matrix_factorization.png" alt="General matrix factorization framework. The data matrix on the left-hand side is decomposed into factors on the right-hand side. The equality may be an approximation as some matrix factorization methods are lossless (exact), while others are an approximation." width="75%" />
<p class="caption">(\#fig:momatrixFactorization)General matrix factorization framework. The data matrix on the left-hand side is decomposed into factors on the right-hand side. The equality may be an approximation as some matrix factorization methods are lossless (exact), while others are an approximation.</p>
</div>

In Figure \@ref(fig:momatrixFactorization), the $5 \times 4$ data matrix $X$ is decomposed to a 2-dimensional latent variable model.

### Multiple factor analysis

\index{multiple factor analysis}Multiple factor analysis is a natural starting point for a discussion about matrix factorization methods for integrating multiple data types. It is a straightforward extension of PCA into the domain of multiple data types [^mfamca].

Figure \@ref(fig:moMFA) sketches a naive extension of PCA to a multi-omics context.

<div class="figure" style="text-align: center">
<img src="images/mfa.png" alt="A naive extension of PCA to multi-omics; data matrices from different platforms are stacked, before applying PCA." width="50%" />
<p class="caption">(\#fig:moMFA)A naive extension of PCA to multi-omics; data matrices from different platforms are stacked, before applying PCA.</p>
</div>



Formally, we have
$$
X = \begin{bmatrix}
           X_{1} \\
           X_{2} \\
           \vdots \\
           X_{L}
         \end{bmatrix} = WH,
$$
a joint decomposition of the different data matrices ($X_i$) into the factor matrix $W$ and the latent variable matrix $H$. This way, we can leverage the ability of PCA to find the highest variance decomposition of the data, when the data consists of different omics types. As a reminder, PCA finds the linear combinations of the features which, when the data is projected onto them, preserve the most variance of any $K$-dimensional space. But because measurements from different experiments have different scales, they will also have variance (and co-variance) at different scales. 

Multiple Factor Analysis addresses this issue and achieves balance among the data types by normalizing each of the data types, before stacking them and passing them on to PCA. Formally, MFA is given by

$$
X_n = \begin{bmatrix}
           X_{1} / \lambda^{(1)}_1 \\
           X_{2} / \lambda^{(2)}_1 \\
           \vdots \\
           X_{L} / \lambda^{(L)}_1
         \end{bmatrix} = WH,
$$
where $\lambda^{(i)}_1$ is the first eigenvalue of the principal component decomposition of $X_i$.

Following this normalization step, we apply PCA to $X_n$. From there on, MFA analysis is the same as PCA analysis, and we refer the reader to Chapter \@ref(unsupervisedLearning) for more details.

#### MFA in R

MFA is available through the CRAN package `FactoMineR`. The code snippet below shows how to run it:

```r
# run the MFA function from the FactoMineR package
r.mfa <- FactoMineR::MFA(
  t(rbind(x1,x2,x3)), # binding the omics types together
  c(dim(x1)[1], dim(x2)[1], dim(x3)[1]), # specifying the dimensions of each
  graph=FALSE)
```

Since this generates a two-dimensional factorization of the multi-omics data, we can now plot each tumor as a dot in a 2D scatter plot to see how well the MFA factors separate the cancer subtypes. The following code snippet generates Figure \@ref(fig:momfascatterplot):

```r
# first, extract the H and W matrices from the MFA run result
mfa.h <- r.mfa$global.pca$ind$coord
mfa.w <- r.mfa$quanti.var$coord

# create a dataframe with the H matrix and the CMS label
mfa_df <- as.data.frame(mfa.h)
mfa_df$subtype <- factor(covariates[rownames(mfa_df),]$cms_label)

# create the plot
ggplot2::ggplot(mfa_df, ggplot2::aes(x=Dim.1, y=Dim.2, color=subtype)) +
ggplot2::geom_point() + ggplot2::ggtitle("Scatter plot of MFA")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/momfascatterplot-1.png" alt="Scatter plot of 2-dimensional MFA for multi-omics data shows separation between the subtypes." width="50%" />
<p class="caption">(\#fig:momfascatterplot)Scatter plot of 2-dimensional MFA for multi-omics data shows separation between the subtypes.</p>
</div>

Figure \@ref(fig:momfascatterplot) shows remarkable separation between the cancer subtypes; it is easy enough to draw a line separating the tumors to CMS subtypes with good accuracy.

Another way to examine the MFA factors, which is also useful for factor models with more than two components, is a heatmap, as shown in Figure \@ref(fig:momfaheatmap), generated by the following code snippet:

```r
pheatmap::pheatmap(t(mfa.h)[1:2,], annotation_col = anno_col,
                  show_colnames = FALSE,
                  main="MFA for multi-omics integration")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/momfaheatmap-1.png" alt="A heatmap of the two MFA components shows separation between the cancer subtypes." width="50%" />
<p class="caption">(\#fig:momfaheatmap)A heatmap of the two MFA components shows separation between the cancer subtypes.</p>
</div>

Figure \@ref(fig:momfaheatmap) shows that indeed, when tumors are clustered and ordered using the two MFA factors we learned above, their separation into CMS clusters is nearly trivial.

\BeginKnitrBlock{rmdtip}<div class="rmdtip">
__Want to know more ?__

- Learn more about FactoMineR on the website: http://factominer.free.fr/
- Learn more about MFA on the Wikipedia page https://en.wikipedia.org/wiki/Multiple_factor_analysis
</div>\EndKnitrBlock{rmdtip}



### Joint non-negative matrix factorization

\index{non-negative matrix factorization (NMF)}As introduced in Chapter \@ref(unsupervisedLearning), NMF (Non-negative Matrix Factorization) is an algorithm from 2000 that seeks to find a non-negative additive decomposition for a non-negative data matrix. It takes the familiar form $X \approx WH$, with $X \ge 0$, $W \ge 0$, and $H \ge 0$. The non-negative constraints make a lossless decomposition (i.e. $X=WH$) generally impossible. Hence, NMF attempts to find a solution which minimizes the Frobenius norm of the reconstruction:

$$
min~\|X-WH\|_F \\
W \ge 0, \\
H \ge 0,
$$

where the Frobenius norm $\|\cdot\|_F$ is the matrix equivalent of the Euclidean distance:

$$
\|X\|_F = \sqrt{\sum_i\sum_jx_{ij}^2}.
$$

This is typically solved for $W$ and $H$ using random initializations followed by iterations of a multiplicative update rule:


\begin{align}
    W_{t+1} &= W_t^T \frac{XH_t^T}{XH_tH_t^T} \\
    H_{t+1} &= H_t \frac{W_t^TX}{W^T_tW_tX}.
\end{align}


Since this algorithm is guaranteed only to converge to a local minimum, it is typically run several times with random initializations, and the best result is kept.

In the multi-omics context, we will, as in the MFA case, wish to find a decomposition for an integrated data matrix of the form

$$
X = \begin{bmatrix}
    X_{1} \\
    X_{2} \\
    \vdots \\
    X_{L}
\end{bmatrix},
$$

with $X_i$s denoting data from different omics platforms.

As NMF seeks to minimize the reconstruction error $\|X-WH\|_F$, some care needs to be taken with regards to data normalization. Different omics platforms may produce data with different scales (i.e. real-valued gene expression quantification, binary mutation data, etc.), and so will have different baseline Frobenius norms. To address this, when doing Joint NMF, we first feature-normalize each data matrix, and then normalize by the Frobenius norm of the data matrix. Formally, we run NMF on

$$
X = \begin{bmatrix}
    X_{1}^N / \alpha_1 \\
    X_{2}^N / \alpha_2 \\
    \vdots \\
    X_{L}^N / \alpha_L
\end{bmatrix},
$$

where $X_i^N$ is the feature-normalized data matrix $X_i^N = \frac{x^{ij}}{\sum_jx^{ij}}$, and $\alpha_i = \|X_{i}^N\|_F$.

Another consideration with NMF is the non-negativity constraint. Different omics data types may have negative values, for instance, copy-number variations (CNVs) may be positive, indicating gains, or negative, indicating losses, as in Table \@ref(tab:mocnvsplitcolshow1). In order to turn such data into a non-negative form, we will split each feature into two features, one new feature holding all the non-negative values of the original feature, and another feature holding the absolute value of the negative ones, as in Table \@ref(tab:mocnvsplitcolshow2).




Table: (\#tab:mocnvsplitcolshow1)Example copy number data. Data can be both positive (amplified regions) or negative (deleted regions).

         seg1   seg2
------  -----  -----
samp1       1      0
samp2       2      1
samp3       1     -2


Table: (\#tab:mocnvsplitcolshow2)Example copy number data after splitting each column into a column representing copy number gains (+) and a column representing deletions (-). This data matrix is non-negative, and thus suitable for NMF algorithms.

         seg1+   seg1-   seg2+   seg2-
------  ------  ------  ------  ------
samp1        1       0       0       0
samp2        2       0       1       0
samp3        1       0       0       2


#### NMF in R

Many NMF algorithms are available through the CRAN package `NMF`. The following code chunk demonstrates how it may be run:





```r
# Feature-normalize the data
x1.featnorm <- x1 / rowSums(x1)
x2.featnorm <- x2 / rowSums(x2)
x3.featnorm <- x3 / rowSums(x3)

# Normalize by each omics type's frobenius norm
x1.featnorm.frobnorm <- x1.featnorm / norm(as.matrix(x1.featnorm), type="F")
x2.featnorm.frobnorm <- x2.featnorm / norm(as.matrix(x2.featnorm), type="F")
x3.featnorm.frobnorm <- x3.featnorm / norm(as.matrix(x3.featnorm), type="F")

# Split the features of the CNV matrix into two non-negative features each
x3.featnorm.frobnorm.nonneg <- t(split_neg_columns(t(x3.featnorm.frobnorm)))

# run the nmf function from the NMF package
require(NMF)
```

```
## Warning: package 'NMF' was built under R version 4.0.2
```

```r
r.nmf <- nmf(t(rbind(x1.featnorm.frobnorm,
                     x2.featnorm.frobnorm,
                     x3.featnorm.frobnorm.nonneg)),
             2,
             method='Frobenius')

# exctract the H and W matrices from the nmf run result
nmf.h <- NMF::basis(r.nmf)
nmf.w <- NMF::coef(r.nmf)
nmfw <- t(nmf.w)
```

As with MFA, we can examine how well 2-factor NMF splits tumors into subtypes by looking at the scatter plot in Figure \@ref(fig:monmfscatterplot), generated by the following code chunk:

```r
# create a dataframe with the H matrix and the CMS label (subtype)
nmf_df <- as.data.frame(nmf.h)
colnames(nmf_df) <- c("dim1", "dim2")
nmf_df$subtype <- factor(covariates[rownames(nmf_df),]$cms_label)

# create the scatter plot
ggplot2::ggplot(nmf_df, ggplot2::aes(x=dim1, y=dim2, color=subtype)) +
ggplot2::geom_point() +
ggplot2::ggtitle("Scatter plot of 2-component NMF for multi-omics integration")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/monmfscatterplot-1.png" alt="NMF creates a disentangled representation of the data using two components which allow for separation between tumor sub-types CMS1 and CMS3 based on NMF factors learned from multi-omics data." width="50%" />
<p class="caption">(\#fig:monmfscatterplot)NMF creates a disentangled representation of the data using two components which allow for separation between tumor sub-types CMS1 and CMS3 based on NMF factors learned from multi-omics data.</p>
</div>

Figure \@ref(fig:monmfscatterplot) shows an important difference between NMF and MFA (PCA). It shows the tendency of samples to lie close to the X or Y axes, that is, the tendency of each sample to be high in only one of the factors. This will be discussed more in the later section on disentangledness.\index{disentangled representations}

Again, should we choose to run NMF with more than two factors, a more useful plot might be the heatmap shown in Figure \@ref(fig:monmfheatmap), generated by the following code snippet:

```r
pheatmap::pheatmap(t(nmf_df[,1:2]),
                   annotation_col = anno_col,
                   show_colnames=FALSE,
                   main="Heatmap of 2-component NMF")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/monmfheatmap-1.png" alt="A heatmap of NMF factors shows separability of tumors into subtype clusters. This plot is more useful than a scatter plot when there are more than two factors." width="50%" />
<p class="caption">(\#fig:monmfheatmap)A heatmap of NMF factors shows separability of tumors into subtype clusters. This plot is more useful than a scatter plot when there are more than two factors.</p>
</div>


\BeginKnitrBlock{rmdtip}<div class="rmdtip">
__Want to know more ?__

- Joint NMF to uncover gene regulatory networks: Zhang S., Li Q., Liu J., Zhou X. J. (2011). A novel computational framework for simultaneous integration of multiple types of genomic data to identify microRNA-gene regulatory modules. _Bioinformatics_ 27, i401–i409. 10.1093/bioinformatics/btr206 https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3117336/
- Joint NMF for cancer research: Zhang S., Liu C.-C., Li W., Shen H., Laird P. W., Zhou X. J. (2012). Discovery of multi-dimensional modules by integrative analysis of cancer genomic data. _Nucleic Acids Res._ 40, 9379–9391. 10.1093/nar/gks725 https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3479191/
</div>\EndKnitrBlock{rmdtip}

### iCluster

\index{iCluster}iCluster takes a Bayesian approach to the latent variable model. In Bayesian statistics, we infer distributions over model parameters, rather than finding a single maximum-likelihood parameter estimate. In iCluster, we model the data as 

$$
X_{(i)} = W_{(i)}Z + \epsilon_i,
$$

where $X_{(i)}$ is a data matrix from a single omics platform, $W_{(i)}$ are model parameters, $Z$ is a latent variable matrix, and is shared among the different omics platforms, and $\epsilon_i$ is a "noise" random variable, $\epsilon \sim N(0,\Psi)$, with $\Psi = diag(\psi_1,\dots \psi_M)$ is a diagonal covariance matrix.

<div class="figure" style="text-align: center">
<img src="images/icluster.png" alt="Sketch of iCluster model. Each omics datatype is decomposed to a coefficient matrix and a shared latent variable matrix, plus noise." width="75%" />
<p class="caption">(\#fig:moiCluster)Sketch of iCluster model. Each omics datatype is decomposed to a coefficient matrix and a shared latent variable matrix, plus noise.</p>
</div>



Note that with this construction, the omics measurements $X$ are expected to be the same for samples with the same latent variable representation, up to Gaussian noise. Further, we assume a Gaussian prior distribution on the latent variables $Z \sim N(0,I)$, which means we assume $X_{(i)} \sim N \big( 0,W_{(i)} W_{(i)}^T + \Psi_{(i)} \big)$. In order to find suitable values for $W$, $Z$, and $\Psi$, we can write down the multivariate normal log-likelihood function and optimize it. For a multivariate normal distribution with mean $0$ and covariance $\Sigma$, the log-likelihood function is given by

$$
\ell = -\frac{1}{2} \bigg( \ln (|\Sigma|) + X^T \Sigma^{-1} X + k\ln (2 \pi) \bigg)
$$

(this is simply the log of the Probability Density Function of a multivariate Gaussian). For the multi-omics iCluster case, we have $X=\big( X_{(1)}, \dots, X_{(L)} \big)^T$, $W = \big( W_{(1)}, \dots, W_{(L)} \big)^T$, where $X$ is a multivariate normal with $0$-mean and $\Sigma = W W^T + \Psi$ covariance. Hence, the log-likelihood function for the iCluster model is given by:

$$
\ell_{iC}(W,\Sigma) = -\frac{1}{2} \bigg( \sum_{i=1}^L \ln (|\Sigma|) + X^T\Sigma^{-1}X + p_i \ln (2 \pi) \bigg)
$$

where $p_i$ is the number of features in omics data type $i$. Because this model has more parameters than we typically have samples, we need to push the model to use fewer parameters than it has at its disposal, by using regularization. iCluster uses Lasso regularization, which is a direct penalty on the absolute value of the parameters. I.e., instead of optimizing $\ell_{iC}(W,\Sigma)$, we will optimize the regularized log-likelihood:\index{loss function}

$$
\ell = \ell_{iC}(W,\Sigma) - \lambda\|W\|_1.
$$

The parameter $\lambda$ acts as a dial to weigh the trade-off between better model fits (higher log-likelihood) and a sparser model, with more $w_{ij}$s set to $0$, which gives models which generalize better and are more interpretable.

In order to solve this problem, iCluster employs the Expectation Maximization (EM) algorithm. The full details are beyond the scope of this textbook. We will introduce a short sketch instead. The intuition behind the EM algorithm is a more general case of the k-means clustering algorithm (Chapter 4). The basic **EM algorithm** is as follows. 

* Initialize $W$ and $\Psi$.
* **Until convergence of $W$, $\Psi$**
    - E-step: Calculate the expected value of $Z$ given the current estimates of $W$ and $\Psi$ and the data $X$.
    - M-step: Calculate maximum likelihood estimates for the parameters $W$ and $\Psi$ based on the current estimate of $Z$ and the data $X$.

#### iCluster+: Extending iCluster

iCluster+ is an extension of the iCluster framework, which allows for omics types to arise from distributions other than a Gaussian. While normal distributions are a good assumption for log-transformed, centered gene expression data, it is a poor model for binary mutations data, or for copy number variation data, which can typically take the values $(-2, 1, 0, 1, 2)$ for heterozygous / monozygous deletions or amplifications. iCluster+ allows the different $X$s to have different distributions:

* for binary mutations, $X$ is drawn from a multivariate binomial
* for normal, continuous data, $X$ is drawn from a multivariate Gaussian
* for copy number variations, $X$ is drawn from a multinomial
* for count data, $X$ is drawn from a Poisson.

In that way, iCluster+ allows us to explicitly model our assumptions about the distributions of our different omics data types, and leverage the strengths of Bayesian inference.

Both iCluster and iCluster+ make use of sophisticated Bayesian inference algorithms (EM for iCluster, Metropolis-Hastings MCMC for iCluster+), which means they do not scale up trivially. Therefore, it is recommended to filter down the features to a manageable size before inputting data to the algorithm. The exact size of "manageable" data depends on your hardware, but a rule of thumb is that dimensions in the thousands are ok, but in the tens of thousands might be too slow.

#### Running iCluster+

iCluster+ is available through the Bioconductor package `iClusterPlus`. The following code snippet demonstrates how it can be run with two components:

```r
# run the iClusterPlus function
r.icluster <- iClusterPlus::iClusterPlus(
  t(x1), # Providing each omics type
  t(x2),
  t(x3),
  type=c("gaussian", "binomial", "multinomial"), # Providing the distributions
  K=2, # provide the number of factors to learn
  alpha=c(1,1,1), # as well as other model parameters
  lambda=c(.03,.03,.03))

# extract the H and W matrices from the run result
# here, we refer to H as z, to keep with iCluster terminology
icluster.z <- r.icluster$meanZ
rownames(icluster.z) <- rownames(covariates) # fix the row names
icluster.ws <- r.icluster$beta

# construct a dataframe with the H matrix (z) and the cancer subtypes
# for later plotting
icp_df <- as.data.frame(icluster.z)
colnames(icp_df) <- c("dim1", "dim2")
rownames(icp_df) <- colnames(x1)
icp_df$subtype <- factor(covariates[rownames(icp_df),]$cms_label)
```

As with other methods, we examine the iCluster results by looking at the scatter plot in Figure \@ref(fig:moiclusterplusscatter) and the heatmap in Figure \@ref(fig:moiclusterplusheatmap). Both figures show that iCluster learns two factors which nearly perfectly discriminate between tumors of the two subtypes.

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/moiclusterplusscatter-1.png" alt="iCluster+ learns factors which allow tumor sub-types CMS1 and CMS3 to be discriminated." width="50%" />
<p class="caption">(\#fig:moiclusterplusscatter)iCluster+ learns factors which allow tumor sub-types CMS1 and CMS3 to be discriminated.</p>
</div>

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/moiclusterplusheatmap-1.png" alt="iCluster+ factors, shown in a heatmap, separate tumors into their subtypes well." width="50%" />
<p class="caption">(\#fig:moiclusterplusheatmap)iCluster+ factors, shown in a heatmap, separate tumors into their subtypes well.</p>
</div>


\BeginKnitrBlock{rmdtip}<div class="rmdtip">
__Want to know more ?__

- Read the original iCluster paper: Shen R., Olshen A. B., Ladanyi M. (2009). Integrative clustering of multiple genomic data types using a joint latent variable model with application to breast and lung cancer subtype analysis. _Bioinformatics_ 25, 2906–2912. 10.1093/bioinformatics/btp543 https://www.ncbi.nlm.nih.gov/pmc/articles/PMC2800366/
- Read the original iClusterPlus paper: an extension of iCluster: Shen R., Mo Q., Schultz N., Seshan V. E., Olshen A. B., Huse J., et al. (2012). Integrative subtype discovery in glioblastoma using iCluster. _PLoS ONE_ 7:e35236. 10.1371/journal.pone.0035236 https://www.ncbi.nlm.nih.gov/pmc/articles/PMC3335101/
- Learn more about the LASSO for model regularization: Tibshirani, R. (1996). Regression shrinkage and selection via the lasso. _J. Royal. Statist. Soc B._, Vol. 58, No. 1, pages 267-288: http://www-stat.stanford.edu/%7Etibs/lasso/lasso.pdf
- Learn more about the EM algorithm: Dempster, A. P., et al. Maximum likelihood from incomplete data via the EM algorithm. _Journal of the Royal Statistical Society. Series B (Methodological)_, vol. 39, no. 1, 1977, pp. 1–38. JSTOR, JSTOR: http://www.jstor.org/stable/2984875
- Read about MCMC algorithms: Hastings, W.K. (1970). Monte Carlo sampling methods using Markov chains and their applications. _Biometrika._ 57 (1): 97–109. doi:10.1093/biomet/57.1.97: https://www.jstor.org/stable/2334940
</div>\EndKnitrBlock{rmdtip}


## Clustering using latent factors

\index{clustering}\index{unsupervised learning}A common analysis in biological investigations is clustering. This is often interesting in cancer studies as one hopes to find groups of tumors (clusters) which behave similarly, i.e. have similar risks and/or respond to the same drugs. PCA is a common step in clustering analyses, and so it is easy to see how the latent variable models above may all be a useful pre-processing step before clustering. In the examples below, we will use the latent variables inferred by the algorithms in the previous section on the set of colorectal cancer tumors from the TCGA. For a more complete introduction to clustering, see Chapter \@ref(unsupervisedLearning).

### One-hot clustering

A specific clustering method for NMF data is to assume each sample is driven by one component, i.e. that the number of clusters $K$ is the same as the number of latent variables in the model and that each sample may be associated to one of those components. We assign each sample a cluster label based on the latent variable which affects it the most. Figure \@ref(fig:monmfheatmap) above (heatmap of 2-component NMF) shows the latent variable values for the two latent variables, for the 72 tumors, obtained by Joint NMF.

The two rows are the two latent variables, and the columns are the 72 tumors. We can observe that most tumors are indeed driven mainly by one of the factors, and not a combination of the two. We can use this to assign each tumor a cluster label based on its dominant factor, shown in the following code snippet, which also produces the heatmap in Figure \@ref(fig:moNMFClustering).


```r
# one-hot clustering in one line of code:
# assign each sample the cluster according to its dominant NMF factor
# easily accessible using the max.col function
nmf.clusters <- max.col(nmf.h)
names(nmf.clusters) <- rownames(nmf.h)

# create an annotation data frame indicating the NMF one-hot clusters
# as well as the cancer subtypes, for the heatmap plot below
anno_nmf_cl <- data.frame(
  nmf.cluster=factor(nmf.clusters),
  cms.subtype=factor(covariates[rownames(nmf.h),]$cms_label)
)

# generate the plot
pheatmap::pheatmap(t(nmf.h[order(nmf.clusters),]),
  cluster_cols=FALSE, cluster_rows=FALSE,
  annotation_col = anno_nmf_cl,
  show_colnames = FALSE,border_color=NA,
  main="Joint NMF factors\nwith clusters and molecular subtypes")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/moNMFClustering-1.png" alt="Joint NMF factors with clusters, and molecular sub-types. One-hot clustering assigns one cluster per dimension, where each sample is assigned a cluster based on its dominant component. The clusters largely recapitulate the CMS sub-types." width="50%" />
<p class="caption">(\#fig:moNMFClustering)Joint NMF factors with clusters, and molecular sub-types. One-hot clustering assigns one cluster per dimension, where each sample is assigned a cluster based on its dominant component. The clusters largely recapitulate the CMS sub-types.</p>
</div>

We see that using one-hot clustering with Joint NMF, we were able to find two clusters in the data which correspond fairly well with the molecular subtype of the tumors.

The one-hot clustering method does not lend itself very well to the other methods discussed above, i.e. iCluster and MFA. The latent variables produced by those other methods may be negative, and further, in the case of iCluster, are going to assume a multivariate Gaussian shape. As such, it is not trivial to pick one "dominant factor" for them. For NMF variants, this is a very common way to assign clusters.

### K-means clustering

K-means \index{clustering!k-means} clustering was introduced in Chapter \@ref(unsupervisedLearning). Briefly, k-means is a special case of the EM algorithm, and indeed iCluster was originally conceived as an extension of K-means from binary cluster assignments to real-valued latent variables. The iCluster algorithm, as it is so named, calls for application of K-means clustering on its latent variables, after the inference step. The following code snippet shows how to pull K-means clusters out of the iCluster results, and produces the heatmap in Figure \@ref(fig:moiClusterHeatmap), which shows how well these clusters correspond to cancer subtypes.


```r
# use the kmeans function to cluster the iCluster H matrix (here, z)
# using 2 as the number of clusters.
icluster.clusters <- kmeans(icluster.z, 2)$cluster
names(icluster.clusters) <- rownames(icluster.z)

# create an annotation dataframe for the heatmap plot
# containing the kmeans cluster assignments and the cancer subtypes
anno_icluster_cl <- data.frame(
  iCluster=factor(icluster.clusters),
  cms.subtype=factor(covariates$cms_label))

# generate the figure
pheatmap::pheatmap(
  t(icluster.z[order(icluster.clusters),]), # order z by the kmeans clusters
  cluster_cols=FALSE, # use cluster_cols and cluster_rows=FALSE
  cluster_rows=FALSE, # as we want the ordering by k-means clusters to hold
  show_colnames = FALSE,border_color=NA,
  annotation_col = anno_icluster_cl,
  main="iCluster factors\nwith clusters and molecular subtypes")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/moiClusterHeatmap-1.png" alt="K-means clustering on iCluster+ factors largely recapitulates the CMS sub-types." width="50%" />
<p class="caption">(\#fig:moiClusterHeatmap)K-means clustering on iCluster+ factors largely recapitulates the CMS sub-types.</p>
</div>

This demonstrates the ability of iClusterPlus to find clusters which correspond to molecular subtypes, based on multi-omics data.


## Biological interpretation of latent factors

### Inspection of feature weights in loading vectors

The most straightforward way to go about interpreting the latent factors in a biological context, is to look at the coefficients which are associated with them. The latent variable models introduced above all take the linear form $X \approx WH$, where $W$ is a factor matrix, with coefficients tying each latent variable with each of the features in the $L$ original multi-omics data matrices. By inspecting these coefficients, we can get a sense of which multi-omics features are co-regulated. The code snippet below generates Figure \@ref(fig:moNMFHeatmap), which shows the coefficients of the Joint NMF analysis above:

```r
# create an annotation dataframe for the heatmap
# for each feature, indicating its omics-type
data_anno <- data.frame(
  omics=c(rep('expression',dim(x1)[1]),
          rep('mut',dim(x2)[1]),
          rep('cnv',dim(x3.featnorm.frobnorm.nonneg)[1])))
rownames(data_anno) <- c(rownames(x1),
                         paste0("mut:", rownames(x2)),
                         rownames(x3.featnorm.frobnorm.nonneg))
rownames(nmfw) <- rownames(data_anno)

# generate the heat map
pheatmap::pheatmap(nmfw,
                   cluster_cols = FALSE,
                   annotation_row = data_anno,
                   main="NMF coefficients",
                   clustering_distance_rows = "manhattan",
                   fontsize_row = 1)
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/moNMFHeatmap-1.png" alt="Heatmap showing the association of input features from multi-omics data (gene expression, copy number variation, and mutations), with JNMF factors. Gene expression features dominate both factors, but copy numbers and mutations mostly affect only one factor each." width="50%" />
<p class="caption">(\#fig:moNMFHeatmap)Heatmap showing the association of input features from multi-omics data (gene expression, copy number variation, and mutations), with JNMF factors. Gene expression features dominate both factors, but copy numbers and mutations mostly affect only one factor each.</p>
</div>

Inspection of the factor coefficients in the heatmap above reveals that Joint NMF has found two nearly orthogonal non-negative factors. One is associated with high expression of the HOXC11, ZIC5, and XIRP1 genes, frequent mutations in the BRAF, PCDHGA6, and DNAH5 genes, as well as losses in the 18q12.2 and gains in 8p21.1 cytobands. The other factor is associated with high expression of the SOX1 gene, more frequent mutations in the APC, KRAS, and TP53 genes, and a weak association with some CNVs.

#### Disentangled representations

\index{disentangled representations}The property displayed above, where each feature is predominantly associated with only a single factor, is termed _disentangledness_, i.e. it leads to _disentangled_ latent variable representations, as changing one input feature only affects a single latent variable. This property is very desirable as it greatly simplifies the biological interpretation of modules. Here, we have two modules with a set of co-occurring molecular signatures which merit deeper investigation into the mechanisms by which these different omics features are related. For this reason, NMF is widely used in computational biology today.

### Making sense of factors using enrichment analysis

\index{enrichment analysis}In order to investigate the oncogenic processes that drive the differences between tumors, we may draw upon biological prior knowledge by looking for overlaps between genes that drive certain tumors, and genes involved in familiar biological processes.

#### Enrichment analysis

The recent decades of genomics have uncovered many of the ways in which genes cooperate to perform biological functions in concert. This work has resulted in rich annotations of genes, groups of genes, and the different functions they carry out. Examples of such annotations include the Gene Ontology Consortium's _GO terms_ [@go_first_paper, @go_latest_paper], the _Reactome pathways database_ [@reactome_latent_paper], and the _Kyoto Encyclopaedia of Genes and Genomes_ [@kegg_latest_paper]. These resources, as well as others, publish lists of so-called _gene sets_, or _pathways_, which are sets of genes which are known to operate together in some biological function, e.g. protein synthesis, DNA mismatch repair, cellular adhesion, and many other functions. Gene set enrichment analysis is a method which looks for overlaps between genes which we have found to be of interest, e.g. by them being implicated in a certain tumor type, and the a-priori gene sets discussed above.

In the context of making sense of latent factors, the question we will be asking is whether the genes which drive the value of a latent factor (the genes with the highest factor coefficients) also belong to any interesting annotated gene sets, and whether the overlap is greater than we would expect by chance. If there are $N$ genes in total, $K$ of which belong to a gene set, the probability that $k$ out of the $n$ genes associated with a latent factor are also associated with a gene set is given by the hypergeometric distribution:

$$
P(k) = \frac{{\binom{K}{k}} - \binom{N-K}{n-k}}{\binom{N}{n}}.
$$

The **hypergeometric test** \index{statistical test} uses the hypergeometric distribution to assess the statistical significance of the presence of genes belonging to a gene set in the latent factor. The null hypothesis is that there is no relationship between genes in a gene set, and genes in a latent factor. When testing for over-representation of gene set genes in a latent factor, the P value from the hypergeometric test is the probability of getting $k$ or more genes from a gene set in a latent factor

$$
p = \sum_{i=k}^K P(k=i).
$$

The hypergeometric enrichment test is also referred to as _Fisher's one-sided exact test_. This way, we can determine if the genes associated with a factor significantly overlap (beyond chance) the genes involved in a biological process. Because we will typically be testing many gene sets, we will also need to apply multiple testing correction, such as Benjamini-Hochberg correction (see Chapter 3, multiple testing correction).

#### Example in R

In R, we can do this analysis using the `enrichR` package, which gives us access to many gene set libraries. In the example below, we will find the genes associated with preferentially NMF factor 1 or NMF factor 2, by the contribution of those genes' expression values to the factor. Then, we'll use `enrichR` to query the Gene Ontology terms which might be overlapping:


```r
require(enrichR)

# select genes associated preferentially with each factor
# by their relative loading in the W matrix
genes.factor.1 <- names(which(nmfw[1:dim(x1)[1],1] > nmfw[1:dim(x1)[1],2]))
genes.factor.2 <- names(which(nmfw[1:dim(x1)[1],1] < nmfw[1:dim(x1)[1],2]))

# call the enrichr function to find gene sets enriched
# in each latent factor in the GO Biological Processes 2018 library
go.factor.1 <- enrichR::enrichr(genes.factor.1,
                                databases = c("GO_Biological_Process_2018")
                                )$GO_Biological_Process_2018
go.factor.2 <- enrichR::enrichr(genes.factor.2,
                                databases = c("GO_Biological_Process_2018")
                                )$GO_Biological_Process_2018
```

The top GO terms associated with NMF factor 2 are shown in Table \@ref(tab:moNMFGOTerms):
\begin{table}

\caption{(\#tab:moNMFGOTerms)GO-terms associated with NMF factor 2}
\centering
\resizebox{\linewidth}{!}{
\begin{tabular}[t]{l|r|r}
\hline
Term & Adjusted.P.value & Combined.Score\\
\hline
nuclear-transcribed mRNA catabolic process (GO:0000956) & 0 & 207.7403\\
\hline
rRNA metabolic process (GO:0016072) & 0 & 161.4781\\
\hline
nuclear-transcribed mRNA catabolic process, nonsense-mediated decay (GO:0000184) & 0 & 220.4298\\
\hline
\end{tabular}}
\end{table}



### Interpretation using additional covariates

Another way to ascribe biological significance to the latent variables is by correlating them with additional covariates we might have about the samples. In our example, the colorectal cancer tumors have also been characterized for microsatellite instability (MSI) status, using an external test (typically PCR-based). By examining the latent variable values as they relate to a tumor's MSI status, we might discover that we've learned latent factors that are related to it. The following code snippet demonstrates how this might be looked into, by generating Figures \@ref(fig:moNMFClinicalCovariates) and \@ref(fig:moNMFClinicalCovariates2):


```r
# create a data frame holding covariates (age, gender, MSI status)
a <- data.frame(age=covariates$age,
                gender=as.numeric(covariates$gender),
                msi=covariates$msi)
```

```
## Warning in data.frame(age = covariates$age, gender =
## as.numeric(covariates$gender), : NAs introduced by coercion
```

```r
b <- nmf.h
colnames(b) <- c('factor1', 'factor2')

# concatenate the covariate dataframe with the H matrix
cov_factor <- cbind(a,b)

# generate the figure
ggplot2::ggplot(cov_factor, ggplot2::aes(x=msi, y=factor1, group=msi)) +
  ggplot2::geom_boxplot() +
  ggplot2::ggtitle("NMF factor 1 microsatellite instability")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/moNMFClinicalCovariates-1.png" alt="Box plot showing MSI/MSS status distribution and NMF factor 1 values." width="50%" />
<p class="caption">(\#fig:moNMFClinicalCovariates)Box plot showing MSI/MSS status distribution and NMF factor 1 values.</p>
</div>

```r
ggplot2::ggplot(cov_factor, ggplot2::aes(x=msi, y=factor2, group=msi)) +
  ggplot2::geom_boxplot() +
  ggplot2::ggtitle("NMF factor 2 and microsatellite instability")
```

<div class="figure" style="text-align: center">
<img src="11-multiomics-analysis_files/figure-html/moNMFClinicalCovariates2-1.png" alt="Box plot showing MSI/MSS status distribution and NMF factor 2 values." width="50%" />
<p class="caption">(\#fig:moNMFClinicalCovariates2)Box plot showing MSI/MSS status distribution and NMF factor 2 values.</p>
</div>

Figures \@ref(fig:moNMFClinicalCovariates) and \@ref(fig:moNMFClinicalCovariates2) show that NMF factor 1 and NMF factor 2 are separated by the MSI or MSS (microsatellite stability) status of the tumors.

## Exercises

### Matrix factorization methods

1. Find features associated with iCluster and MFA factors, and visualize the feature weights. [Difficulty: **Beginner**]

2. Normalizing the data matrices by their $\lambda_1$'s as in MFA supposes we wish to assign each data type the same importance in the down-stream analysis. This leads to a natural generalization whereby the different data types may be differently weighted. Provide an implementation of weighed-MFA where the different data types may be assigned individual weights. [Difficulty: **Intermediate**]

3. In order to use NMF algorithms on data which can be negative, we need to split each feature into two new features, one positive and one negative. Implement the following function, and see that the included test does not fail: [Difficulty: **Intermediate/Advanced**]


```r
# Implement this function
split_neg_columns <- function(x) {
    # your code here
}

# a test that shows the function above works
test_split_neg_columns <- function() {
    input <- as.data.frame(cbind(c(1,2,1),c(0,1,-2)))
    output <- as.data.frame(cbind(c(1,2,1), c(0,0,0), c(0,1,0), c(0,0,2)))
    stopifnot(all(output == split_neg_columns(input)))
}

# run the test to verify your solution
test_split_neg_columns()
```

4. The iCluster+ algorithm has some parameters which may be tuned for maximum performance. The `iClusterPlus` package has a method, `iClusterPlus::tune.iClusterPlus`, which does this automatically based on the Bayesian Information Criterion (BIC). Run this method on the data from the examples above and find the optimal lambda and alpha values. [Difficulty: **Beginner/Intermediate**]

### Clustering using latent factors

1. Why is one-hot clustering more suitable for NMF than iCluster? [Difficulty: **Intermediate**]

2. Which clustering algorithm produces better results when combined with NMF, K-means, or one-hot clustering? Why do you think that is? [Difficulty: **Intermediate/Advanced**]

### Biological interpretation of latent factors

1. Another covariate in the metadata of these tumors is their _CpG island methylator Phenotype_ (CIMP). This is a phenotype carried by a group of colorectal cancers that display hypermethylation of promoter CpG island sites, resulting in the inactivation of some tumor suppressors. This is also assayed using an external test. Do any of the multi-omics methods surveyed find a latent variable that is associated with the tumor's CIMP phenotype? [Difficulty: **Beginner/Intermediate**]



2. Does MFA give a disentangled representation? Does `iCluster` give disentangled representations? Why do you think that is? [Difficulty: **Advanced**]

3. Figures \@ref(fig:moNMFClinicalCovariates) and \@ref(fig:moNMFClinicalCovariates2) show that MSI/MSS tumors have different values for NMF factors 1 and 2. Which NMF factor is associated with microsatellite instability? [Difficulty: **Beginner**]

4. Microsatellite instability (MSI) is associated with hyper-mutated tumors. As seen in Figure \@ref(fig:momutationsHeatmap), one of the subtypes has tumors with significantly more mutations than the other. Which subtype is that? Which NMF factor is associated with that subtype? And which NMF factor is associated with MSI? [Difficulty: **Advanced**]


[^mfamca]: When dealing with categorical variables, MFA uses MCA (Multiple Correspondence Analysis). This is less relevant to biological data analysis and will not be discussed here.
