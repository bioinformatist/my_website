+++
highlight = true
external_link = ""
image_preview = ""
summary = "Xiaoya Zhou's miRNA mircroarray data analysis."
tags = ['miRNA', 'microarray']
math = false
date = "2017-02-09T15:37:51+08:00"
title = "Research Project 1"
image = ""

+++

<!-- TOC START min:1 max:3 link:true update:true -->
  - [Count numbers for each type of miRNAs/probes](#count-numbers-for-each-type-of-mirnasprobes)
  - [Extract expression matrix from raw data](#extract-expression-matrix-from-raw-data)
  - [Normalize expression matrix](#normalize-expression-matrix)
  - [Principle components analysis](#principle-components-analysis)
    - [By SVD (a "false" demo :joy:)](#by-svd-a-false-demo-joy)
    - [By base R function `stats::prcomp`](#by-base-r-function-statsprcomp)
    - [By SVD (Centered data)](#by-svd-centered-data)

<!-- TOC END -->

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

Through checking the results, file `expr` has 2081 lines while the `Raw Intensity File.csv` file has 2085 probe records. What's the **four** ones remaining? Comparison applied to show difference.

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

These miRNAs had more than one one probes. Considering the requirement of identifying differentially expressed miRNAs and this rare scenario, my procedure may works well up to now.

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

What comes first?

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

### By SVD (a "false" demo :joy:)

Let's decomposite the matrix by [SVD (Singular Value Decomposition)](http://genomicsclass.github.io/book/pages/svd.html) method first.

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
source('~/github.com/bioinformatist/research_projects/project1/scripts/plot.PCA.R')
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
