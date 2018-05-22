+++
# Date this page was created.
date = 2017-02-09T15:37:51+08:00

# Project title.
title = "A research project"

# Project summary to display on homepage.
summary = "Somebody's miRNA mircroarray data analysis."

# Optional image to display on homepage (relative to `static/img/` folder).
image_preview = "project_img_preview/heatmap.dd.OP vs H.png"

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ['miRNA', 'microarray']

# Optional external URL for project (replaces project detail page).
external_link = ""

# Does the project detail page use math formatting?
math = false

# Optional featured image (relative to `static/img/` folder).
[header]
image = ""
caption = ""
+++

<!-- TOC START min:1 max:3 link:true update:true -->
  - [Count numbers for each type of miRNAs/probes](#count-numbers-for-each-type-of-mirnasprobes)
  - [Extract expression matrix from raw data](#extract-expression-matrix-from-raw-data)
  - [Normalize expression matrix](#normalize-expression-matrix)
  - [Principle components analysis](#principle-components-analysis)
    - [By SVD (a "false" demo)](#by-svd-a-false-demo)
    - [By base R function `stats::prcomp`](#by-base-r-function-statsprcomp)
    - [By SVD (Centered data)](#by-svd-centered-data)
  - [miRNAs filtering](#mirnas-filtering)
  - [Get DE (differentially expressed) miRNAs](#get-de-differentially-expressed-mirnas)
  - [Heatmap with dendrogram](#heatmap-with-dendrogram)
  - [Retrieve all validated target genes of given miRNAs](#retrieve-all-validated-target-genes-of-given-mirnas)
  - [GO and KEGG enrichment analysis](#go-and-kegg-enrichment-analysis)
  - [SpidermiR](#spidermir)

<!-- TOC END -->

All contents of this project post can be found in my [GitHub repo](https://github.com/bioinformatist/research_projects/tree/master/project1).

I've checked the reports and raw data, and there're only probes for *Hy3*, with no *Hy5* labled. Hence, it must be a single channel microarray.

## Count numbers for each type of miRNAs/probes
```perl
perl -F, -lane'!($.>= 12 and $F[3] =~ /miR|let/) and next; $count{(split/[-_]/, $F[3])[1]}++ }{ print qq{$_ | $count{$_}} for sort {$count{$b} <=> $count{$a}} keys %count' 'Raw Intensity File.csv'
```

STDOUT:

Type | Counts
---- | ------
miR | 8200
miRPlus | 72
let | 72

## Extract expression matrix from raw data
Perl-oneliner I used below removed headers (comment lines at the start of the file) and abundant columns.

{{% alert note %}}
Known that each kind of probe has four replication on the microarray, I chose the probe set with **the highest variance** for each miRNA as representation. Replicated miRNAs were merged by **median** of each sample by *KangChen Bio-tech*, which seems like a self-evidenct method, actually, has fatal limitation. See [**Question: Handling Duplicate Probe Expression Values In Spotted Cdna Microarray** on **Biostars**](https://www.biostars.org/p/51756/#51875) for details.
{{% /alert %}}

```perl
perl -MStatistics::Descriptive -F, -lane'!($.>= 12 and $F[3] =~ /miR|let/) and next; @tmp = split/,/, $_; $stat->add_data(@tmp[23..$#tmp]); $tvar = $stat->variance(); if ($tvar > $var{$F[3]}) {$var{$F[3]} = $tvar; $line{$F[3]} = join(qq{\t}, @tmp[3,23..$#tmp])} $stat->clear()}{ BEGIN{$stat = Statistics::Descriptive::Sparse->new()} print $line{$_} for keys %line' 'Raw Intensity File.csv' > expr
```

By checking results, file `expr` has 2081 lines while the `Raw Intensity File.csv` file has 2085 probe records. What's the **four** ones remaining? Comparison was applied to show difference.

```bash
# To get a sorted miRNA name list
cut -f1 expr | sort > 1
cut -d, -f4 'Raw Intensity File.csv' | sort > 2
# Then I remove headers manually (actually it should be deleted by `sed`)
diff 1 2 > 3
cat 3
```

The result of `diff`:

```pre
0a1
>
256a258
> hsa-miR-134-3p
685a688
> hsa-miR-328-5p
860a864
> hsa-miR-381-5p
1931a1936
> hsa-miR-874-5p
```

These miRNAs had more than one probes. Considering the requirement of identifying differentially expressed miRNAs and this rare scenario, my procedure may works well up to now.

{{% alert note %}}
As microRNA probes bind more miRNAs than what you are interested in, for each miRNA, there may be more than one probe designed. See the answer [here](https://support.bioconductor.org/p/88210/#88231).
{{% /alert %}}

## Normalize expression matrix
To illustrate raw data distribution, a density plot and a boxplot were generated.

```R
setwd('~/github.com/bioinformatist/research_projects/project1/')
library(data.table)
library(cowplot)

DT.expr1 <- fread('expr')
sample.names <- c('miRNAs', '37', '40', '58', '59', '63', '69', '49', '50', '64', '67', '68', '79', '70', '72', '73', '75', '90', '91')
setnames(DT.expr1, sample.names)
DT.expr1.raw.m <- melt(DT.expr1, id = 1, variable.name = 'sample.name')
```

{{% alert note %}}
Here, both package `reshape` and `reshape2` have function `melt`, and `data.table` has its own `melt.data.table` function alias `melt`. How can we deal with them?
{{% /alert %}}

```R
# To specify the package that you want to use, the syntax is: packagename::functionname()
data.table::melt
reshape2::melt
# If you always want to use function in one, you can define your own function as follows:
melt <- data.table::melt
```

{{% alert note %}}
And keep in mind that the order of loading the packages makes a difference, i.e. the package that gets loaded last will mask the functions in packages loaded earlier.
{{% /alert %}}

Let's move on. Make graphics now:

```R
density.raw <- ggdraw(ggplot(data = DT.expr1.raw.m, aes(value, colour = sample.name)) + geom_density() + scale_x_continuous(trans = 'log2')) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/density.raw.png', density.raw, base_height = 8.5, base_width = 11)
boxplot.raw <- ggdraw(ggplot(data = DT.expr1.raw.m, aes(sample.name, value)) + geom_boxplot(notch = TRUE) + scale_y_continuous(trans = 'log2')) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/boxplot.raw.png', boxplot.raw, base_height = 8.5, base_width = 11)
```

The density plot of raw data:
![density.raw.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/density.raw.png)

The boxplot of raw data:
![boxplot.raw.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/boxplot.raw.png)

Why use such size for figures?

> About figure size: Each figure should be able to fit on a single 8.5 x 11 inch page. Please do not send figure panels as individual files. We use three standard widths for figures: 1 column, 85 mm; 1.5 column, 114 mm; and 2 column, 174 mm (the full width of the page). Although your figure size may be reduced in the print journal, please keep these widths in mind. For Previews and other three-column formats, these widths are also applicable, though the width of a single column will be 55 mm. --From [Cell Press Digital Image Guidelines (click to see details)](http://www.cell.com/figureguidelines).

Width and height, which comes first?

As a rule of thumb, the Graphics’ industry standard is width by height (width x height). Meaning that when you write your measurements, you write them from your point of view, beginning with the width.

Layout orientation-wise using a letter-sized paper,
8.5×11 = portrait
11×8.5 = landscape

Width x Height
Width = top margin
Height = left margin

For online review in-house, I use `png` format instead of `tiff`, which will be reproduced later to remove watermark and meet conditions for publishing as many of them just for a passing glance :smile:

Let's perform constant normalization first.

```R
source('~/github.com/bioinformatist/research_projects/project1/scripts/normalization.R')

SD.pos <- 2:dim(DT.expr1)[2]
# The parentheses make the result to be assigned to the column specified in SD.pos, instead of some new variable named SD.pos
# Function lapply is better for .SD in data.table
DT.expr1.normalized.constant <- DT.expr1[, (SD.pos) := lapply(.SD, NormalizeconstantAsCol), .SDcols = SD.pos]
```

{{% alert warning %}}
Generally, index like `-1` can be used as the value of `.SDcols`, but when used as LHS of `:=`, it will raise an error:
*LHS of := appears to be column positions but are outside [1,ncol] range. New columns can only be added by name.*
{{% /alert %}}

```R
DT.expr1.normalized.constant.m <- melt(DT.expr1.normalized.constant, id = 1, variable.name = 'sample.name')
density.normalized.constant <- ggdraw(ggplot(data = DT.expr1.normalized.constant.m, aes(value, colour = sample.name)) + geom_density() + scale_x_continuous(trans = 'log2')) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/density.normalized.constant.png', density.normalized.constant, base_height = 8.5, base_width = 11)
boxplot.normalized.constant <- ggdraw(ggplot(data = DT.expr1.normalized.constant.m, aes(sample.name, value)) + geom_boxplot(notch = TRUE) + scale_y_continuous(trans = 'log2')) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/boxplot.normalized.constant.png', boxplot.normalized.constant, base_height = 8.5, base_width = 11)
```

The density plot of data processed by constant normalization:
![density.normalized.constant.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/density.normalized.constant.png)

The boxplot of data by constant normalization:
![boxplot.normalized.constant.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/boxplot.normalized.constant.png)

Then try quantile normalization:

```R
library(preprocessCore)
DT.expr1.normalized.quantile <- DT.expr1[,c(list(miRNAs, normalize.quantiles(as.matrix(.SD)))), .SDcol = SD.pos]
setnames(DT.expr1.normalized.quantile, sample.names)

DT.expr1.normalized.quantile.m <- melt(DT.expr1.normalized.quantile, id = 1, variable.name = 'sample.name')
density.normalized.quantile <- ggdraw(ggplot(data = DT.expr1.normalized.quantile.m, aes(value, colour = sample.name)) + geom_density() + scale_x_continuous(trans = 'log2')) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/density.normalized.quantile.png', density.normalized.quantile, base_height = 8.5, base_width = 11)
boxplot.normalized.quantile <- ggdraw(ggplot(data = DT.expr1.normalized.quantile.m, aes(sample.name, value)) + geom_boxplot(notch = TRUE) + scale_y_continuous(trans = 'log2')) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/boxplot.normalized.quantile.png', boxplot.normalized.quantile, base_height = 8.5, base_width = 11)
```

The density plot of data processed by quantile normalization:
![density.normalized.quantile.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/density.normalized.quantile.png)

The boxplot of data by quantile normalization:
![boxplot.normalized.quantile.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/boxplot.normalized.quantile.png)

Finally, let's output a summary:

```R
summary.plot <- plot_grid(density.raw, boxplot.raw, density.normalized.constant, boxplot.normalized.constant, density.normalized.quantile, boxplot.normalized.quantile, labels = c('R', 'R', 'C', 'C', 'Q', 'Q'), nrow = 3, align = 'v')
save_plot('figures/summary.png', summary.plot, base_height = 10 * 1.3, base_width = 10 * 1.3)
```

Summary plot:
![summary.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/summary.png)

According to this summary, the expression matrix processed by quantile normalization, i.e. `DT.expr1.normalized.quantile` was chosen as the input for the downstream analysis.

```R
# Backup normalized matrix
save(DT.expr1.normalized.quantile, file = 'expr.normalized.RData', compress = 'xz', compression_level = 9)
```

## Principle components analysis
### By SVD (a "false" demo)
First, try decomposite the matrix by [SVD (Singular Value Decomposition)](http://genomicsclass.github.io/book/pages/svd.html) method.

```R
setwd('~/github.com/bioinformatist/research_projects/project1/')
library(data.table)
library(cowplot)

load('expr.normalized.RData')
expr.T <- as.data.frame(t(DT.expr1.normalized.quantile[,-1]))
colnames(expr.T) <- DT.expr1.normalized.quantile$miRNAs
sv <- svd(expr.T)
U = sv$u
V = sv$v
D = sv$d
```

First check if we can in fact reconstruct `expr.T`:

```R
expr.T.hat <- U %*% diag(sv$d) %*% t(V)
expr.T.resid <- expr.T - expr.T.hat
max(abs(expr.T.resid))
```

The largest residual is small enough to be ignored:

```pre
[1] 2.869029e-10
```

With scree plot:

```R
variance.explained <- D^2/sum(D^2)
variance.explained <- data.table(Index = 1:length(variance.explained), var = variance.explained)
ggdraw(ggplot(data = variance.explained, aes(x = Index, y = var)) + geom_point(shape = 1, size = 3) + geom_path() + labs(y = 'Percent variability explained')) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/screeplot.SVD.png', plot = last_plot(), base_height = 8.5, base_width = 11)
```

The screeplot of data PCAed by SVD:
![screeplot.SVD.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/screeplot.SVD.png)

We can see that cumulative variance explained by first two PCs is over ~80% of total variance. Therefore, we don't need observe more than the first two PCs.

```R
# U are un-scaled PCs. Use Z as scaled PC:
Z = as.matrix(expr.T) %*% V
pc.DT <- data.table(sample = rownames(Z), PC1 = Z[,1], PC2 = Z[,2])
ggdraw(ggplot(pc.DT, aes(x = PC1, y = PC2, col = sample)) + geom_point() + geom_text(aes(label = sample), hjust=0, vjust=0)) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/biplot.SVD.png', plot = last_plot(), base_height = 8.5, base_width = 11)
```

The biplot of data PCAed by SVD:
![biplot.SVD.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/biplot.SVD.png)

 :anger:What The Fuck? :anger: Considering I'm not familiar with SVD, maybe there's some mistakes. Again, I'll do it with R's `stats::prcomp` function.

### By base R function `stats::prcomp`
```R
# Run unless ggbiplot has been installed
# library(devtools)
# install_github("ggbiplot", "vqv")
library(ggbiplot)
expr.T <- as.data.frame(t(DT.expr1.normalized.quantile[,-1]))
colnames(expr.T) <- DT.expr1.normalized.quantile$miRNAs
# Center, but not scale
expr.pca <- prcomp(expr.T, scale. = FALSE)
# Make various forms of scree plot
# source('~/github.com/bioinformatist/research_projects/project1/scripts/plot.PCA.R')
# pcaCharts(expr.pca)
# Define group, H stand for healthy group, OP for osteoporosis group and OPC for osteoporosis with cataclasis group
expr.class <- c(rep('H', 6), rep('OP', 6), rep('OPC', 6))
ggdraw(ggbiplot(expr.pca, groups = expr.class, ellipse = TRUE, circle = TRUE, labels = row.names(expr.T), var.axes = FALSE)) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/biplot.prcomp.png', plot = last_plot(), base_height = 8.5, base_width = 11)
```

The biplot of data PCAed by `stats::prcomp`:
![biplot.prcomp.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/biplot.prcomp.png)

From the R documents of `stats::prcomp`, the function use **SVD** as well:

> The calculation is done by a singular value decomposition of the (centered and possibly scaled) data matrix, not by using eigen on the covariance matrix.

(Silence... :speech_balloon:)

After careful comparison, the only difference is that I use centered matrix when performing PCA with`stats::prcomp`.

### By SVD (Centered data)
```R
expr.centered <- t(scale(t(DT.expr1.normalized.quantile[,-1]),center=TRUE,scale=FALSE))
expr.T <- t(expr.centered)

sv <- svd(expr.T)
U = sv$u
V = sv$v
D = sv$d

Z = as.matrix(expr.T) %*% V
pc.DT <- data.table(sample = rownames(Z), PC1 = Z[,1], PC2 = Z[,2])
ggdraw(ggplot(pc.DT, aes(x = PC1, y = PC2, col = sample)) + geom_point() + geom_text(aes(label = sample), hjust=0, vjust=0)) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/biplot.SVD.ver2.png', plot = last_plot(), base_height = 8.5, base_width = 11)
```

The biplot of data PCAed by SVD (with centered data):
![biplot.SVD.ver2.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/biplot.SVD.ver2.png)

Quite similar with one by `stats::prcomp`, isn't it? :trollface:

## miRNAs filtering
Use log2 method to scale the data:

```R
library(genefilter)
source('~/github.com/bioinformatist/research_projects/project1/scripts/log2.scale.R')
expr.log2 <- log2.scale(DT.expr1.normalized.quantile[,-1])
```

According to the `limma`'s user guide:

> Note that filtering methods involving variances should not be used. The limma algorithm analyses the spread of the genewise variances. Any filtering method based on genewise variances will change the distribution of variances, will interfere with the limma algorithm and hence will give poor results.

And:

> There are a number of ways that filtering can be done. One way is to keep probes that are expressed above background on at least n arrays, where n is the smallest number of replicates assigned to any of the treatment combinations.

There's a package named `genefilter` can do it. I chose **six** as the smallest number of replicates, and to set a proper threshold:

```R
ggdraw(ggplot(data = data.table(intensity = as.vector(t(expr.log2))), aes(x = intensity)) + geom_density()) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/density.log2.all.png', plot = last_plot(), base_height = 8.5, base_width = 11)
```

The density plot through all samples in log2-scaled matrix:
![density.log2.all.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/density.log2.all.png)

Zoom in and add more ticks with labels to see the details:

```R
ggdraw(ggplot(data = data.table(intensity = as.vector(t(expr.log2))), aes(x = intensity)) + geom_density() + geom_vline(xintercept = 3.5, color = 'red', size = 3)+ scale_x_continuous(limits = c(2, 8), breaks = c(seq(2, 5, 0.5), 8), labels = c(seq(2, 5, 0.5), 8))) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/density.log2.local.png', plot = last_plot(), base_height = 8.5, base_width = 11)
```

The density plot zoomed in:
![density.log2.local.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/density.log2.local.png)

That's why I determined to use **3.5** as threshold of function `genefilter::kOverA`.

```R
# Perform filtering
f1 <- kOverA(6, 3.5)
flist <- filterfun(f1)
expr.flr <- genefilter(expr.log2, flist)
# Show the remaining number of features (miRNAs)
sum(expr.flr)
```

```pre
[1] 2077
```

Actually, **none** of features has been removed...

```R
# Backup log2-scaled and filtered (no effect indeed)
save(expr.log2, file = 'expr.flr.RData', compress = 'xz', compression_level = 9)
```

## Get DE (differentially expressed) miRNAs
```R
setwd('~/github.com/bioinformatist/research_projects/project1/')
library(data.table)
library(cowplot)
library(limma)

load('expr.flr.RData')
load('expr.normalized.RData')

design <- model.matrix(~ 0+factor(c(rep(1, 6), rep(2, 6), rep(3, 6))))
colnames(design) <- c("H", "OP", "OPC")
fit <- lmFit(expr.log2, design)
contrast.matrix <- makeContrasts(OP-H, OPC-OP, OPC-H, levels=design)
fit2 <- contrasts.fit(fit, contrast.matrix)
fit2 <- eBayes(fit2)

DE.OP.H <- topTable(fit2, coef = 1, genelist = DT.expr1.normalized.quantile[,1], number = nrow(fit2), sort.by = 'p', resort.by = 'M', lfc = log2(2), p.value = 0.01)
DE.OPC.OP <- topTable(fit2, coef = 2, genelist = DT.expr1.normalized.quantile[,1], number = nrow(fit2), sort.by = 'p', resort.by = 'M', lfc = log2(2), p.value = 0.01)
DE.OPC.H <- topTable(fit2, coef = 3, genelist = DT.expr1.normalized.quantile[,1], number = nrow(fit2), sort.by = 'p', resort.by = 'M', lfc = log2(2), p.value = 0.01)

fwrite(DE.OP.H, file = 'OP vs H.csv')
fwrite(DE.OPC.OP, file = 'OPC vs OP.csv')
fwrite(DE.OPC.H, file = 'OPC vs H.csv')
```

To draw a volcano plot:

```R
volcano.OP.H <- topTable(fit2, coef = 1, genelist = DT.expr1.normalized.quantile[,1], number = nrow(fit2), sort.by = 'p', resort.by = 'M')
volcano.OP.H$threshold = as.factor(abs(volcano.OP.H$logFC) > 2 & volcano.OP.H$P.Value < 0.05 / nrow(fit2))
ggdraw(ggplot(data = volcano.OP.H, aes(x = logFC, y = -log10(P.Value), colour = threshold)) + geom_point(alpha=0.4, size=1.75) + theme(legend.position = "none") + xlim(c(-10, 10)) + ylim(c(0, 15)) + xlab(expression("log"[2]*"(Fold Change)")) + ylab(expression("-log"[10]*"(p-value)"))) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/volcano.OP vs H.png', plot = last_plot(), base_height = 8.5, base_width = 11)
```

The volcano plot of *OP vs H*:
![volcano.OP vs H.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/volcano.OP vs H.png)

## Heatmap with dendrogram

To make a heatmap with dendrogram as a `ggplot` object, I use `cowplot` and `ggdendro` in a [R script](https://github.com/bioinformatist/research_projects/blob/master/project1/scripts/ggheatmap.R).

```R
# install.packages('ggdendro')
setwd('~/github.com/bioinformatist/research_projects/project1/')
load('expr.normalized.RData')
load('DE.miRNAs.target.genes.RData')
source('~/github.com/bioinformatist/research_projects/project1/scripts/ggheatmap.R')
library(data.table)
DEGs.OP.H <- DT.expr1.normalized.quantile[DE.OP.H, on = "miRNAs"][,1:dim(DT.expr1.normalized.quantile)[2]]
heatmap.dendrogram <- ggheatmap(DEGs.OP.H)
ggdraw() + draw_plot(heatmap.dendrogram[[1]], 0.0, 0.0, .8, .8) + draw_plot(heatmap.dendrogram[[2]], 0.21, .8, .52, .1) + draw_plot(heatmap.dendrogram[[3]], .8, -.031, .2, .8673) + draw_label("Draft for \n Peng's Lab!", angle = 45, size = 80, alpha = .2)
save_plot('figures/heatmap.dd.OP vs H.png', plot = last_plot(), base_height = 30, base_width = 10)
```

The heatmap with dendrogram of *OP vs H*:
![heatmap.dd.OP vs H.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/heatmap.dd.OP vs H.png)

## Retrieve all validated target genes of given miRNAs
```R
# Install package multiMiR with dependence
# install.packages("XML")
# install.packages("RCurl")
# install.packages('http://multimir.ucdenver.edu/multiMiR_2.1.1.tar.gz', repos = NULL, type = 'source')

setwd('~/github.com/bioinformatist/research_projects/project1/')
library(multiMiR)
library(data.table)
library(cowplot)

DE.OP.H <- fread('OP vs H.csv')
DE.OPC.OP <- fread('OPC vs OP.csv')
DE.OPC.H <- fread('OPC vs H.csv')

OP.H.target.up <- get.multimir(mirna = DE.OP.H[logFC > 0, miRNAs], summary = TRUE)
OP.H.target.down <- get.multimir(mirna = DE.OP.H[logFC < 0, miRNAs], summary = TRUE)
OPC.OP.target.up <- get.multimir(mirna = DE.OPC.OP[logFC > 0, miRNAs], summary = TRUE)
OPC.OP.target.down <- get.multimir(mirna = DE.OPC.OP[logFC < 0, miRNAs], summary = TRUE)
OPC.H.target.up <- get.multimir(mirna = DE.OPC.H[logFC > 0, miRNAs], summary = TRUE)
OPC.H.target.down <- get.multimir(mirna = DE.OPC.H[logFC < 0, miRNAs], summary = TRUE)

save.image(file = "DE.miRNAs.target.genes.RData")

```

## GO and KEGG enrichment analysis
DEGs between OP and H groups were used in this step.

```R
setwd('~/github.com/bioinformatist/research_projects/project1/')
load('DE.miRNAs.target.genes.RData')
# Install R devtools
# sudo dnf install openssl-devel
# sudo dnf install libcurl-devel.x86_64
# install.packages('devtools')
# library(devtools)
# install.packages('png')
# devtools::install_github("shenwei356/swr")
# Install package clusterProfiler
# source("https://bioconductor.org/biocLite.R")
# biocLite("clusterProfiler")
# library(swr)
# library(clusterProfiler)
# library(cowplot)
# library(stringr)

source('~/github.com/bioinformatist/research_projects/project1/scripts/annotating.R')
Entrez2GOResults(as.character(OP.H.target.down$summary[,4]), 'OP vs H', watermark.content = "Draft for \n Peng's Lab!")
```

As you can see, to avoid repeating some statement for many times, I use a [R script](https://github.com/bioinformatist/research_projects/blob/master/project1/scripts/annotating.R) for annotating and enrichment, getting result tables and drawing figures (even adding watermark).

The GO classification barplot:
![barplot.ggo.OP vs H.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/barplot.ggo.OP vs H.png)

The GO enrichment dotplot:
![barplot.ego.OP vs H.png](https://github.com/bioinformatist/research_projects/raw/master/project1/figures/barplot.ego.OP vs H.png)

**KEGG over-representation test:**

```R
library(data.table)
kk.OP.H <- enrichKEGG(as.character(OP.H.target.down$summary[,4]))
mkk.OP.H <- enrichMKEGG(as.character(OP.H.target.down$summary[,4]))  # No matches
fwrite(kk.OP.H@result, file = "KEGG.OP vs H.csv")
```

**Results in tables:**

GO classification: [BP](https://github.com/bioinformatist/research_projects/blob/master/project1/ggo.OP vs H.BP.csv), [CC](https://github.com/bioinformatist/research_projects/blob/master/project1/ggo.OP vs H.CC.csv) and [MF](https://github.com/bioinformatist/research_projects/blob/master/project1/ggo.OP vs H.MF.csv);

GO enrichment: [BP](https://github.com/bioinformatist/research_projects/blob/master/project1/ego.OP vs H.BP.csv), [CC](https://github.com/bioinformatist/research_projects/blob/master/project1/ego.OP vs H.CC.csv) and [MF](https://github.com/bioinformatist/research_projects/blob/master/project1/ego.OP vs H.MF.csv).

Also [KEGG enrichment results](https://github.com/bioinformatist/research_projects/blob/master/project1/KEGG.OP vs H.csv).

## SpidermiR
```R
setwd('~/github.com/bioinformatist/research_projects/project1/')
library(SpidermiR)
# Check species supported by GeneMania
org <- SpidermiRquery_species(species)
```

Supported species:

```pre
tabOrgd
1     Arabidopsis_thaliana
2   Caenorhabditis_elegans
3              Danio_rerio
4  Drosophila_melanogaster
5         Escherichia_coli
6             Homo_sapiens
7             Mus_musculus
8        Rattus_norvegicus
9 Saccharomyces_cerevisiae
```

```R
net_type <- SpidermiRquery_networks_type(organismID=org[6,])
```

Supported network types for *Homo Sapiens*:

```pre
[1] "Genetic interactions"   "Co-expression"          "Pathway"                "Predicted"             
[5] "Co-localization"        "Shared protein domains" "Physical interactions"
```
