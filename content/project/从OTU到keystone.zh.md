+++
title = "从OTU到keystone网络图"
date = 2018-08-16T22:32:30+08:00
draft = false

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ['16S rDNA测序', '数据可视化']

# Project summary to display on homepage.
summary = ""

# Optional image to display on homepage.
image_preview = ""

# Optional external URL for project (replaces project detail page).
external_link = ""

# Does the project detail page use math formatting?
math = false

# Does the project detail page use source code highlighting?
highlight = true

# Featured image
# Place your image in the `static/img/` folder and reference its filename below, e.g. `image = "example.jpg"`.
[header]
image = ""
caption = ""

+++

先把存放OTU丰度矩阵的表格另存为制表符分隔的*lsa_input.txt* 文件（其实就是tsv），然后找到了这个[fastspar](https://github.com/scwatts/fastspar)，是SparCC的快速版（原来的是python实现的好慢啊在服务器上都跑不动啊摔），经过小数据集测试，发现结果基本是一致的，那就可用啦。

> 此处我未用conda安装，因为conda提供的并非最新版本，这种小软件版本号都不到0.1，担心是修复bug才更新，那就糗大了。

```shell
nohup ./fastspar -t 50 -s 1013 --iterations 500 --exclude_iterations 200 --otu_table lsa_input.txt --correlation median_correlation.tsv --covariance median_covariance.tsv > fastspar.logs &
mkdir bootstrap_counts
nohup ./fastspar_bootstrap --otu_table lsa_input.txt --number 1000 --prefix bootstrap_counts/lsa_input > bootstrap.logs &
mkdir bootstrap_correlation
nohup parallel ./fastspar --otu_table {} --correlation bootstrap_correlation/cor_{/} --covariance bootstrap_correlation/cov_{/} -i 500 ::: bootstrap_counts/* > parallel_correlation.logs &
nohup ./fastspar_pvalues -t 50 --otu_table lsa_input.txt --correlation median_correlation.tsv --prefix bootstrap_correlation/cor_lsa_input_ --permutations 1000 --outfile pvalues.tsv > pvalues.logs &
```

算得的correlation很低，非常惨，不适合画图，就希望在R里面做一个类似center的变换(`x / sum(x)`)：

```r
library(phyloseq)
library(gdata)
otu.table <- otu_table(read.table(file = "lsa_input.txt", sep = "\t", header = T, row.names = 1, comment.char = '', check.names = FALSE), taxa_are_rows = TRUE)
otu.table <- transform_sample_counts(otu.table, function(x) x / sum(x) )
write.table(otu.table, 'normalized.txt', sep = '\t', quote = FALSE)
```

```shell
nohup ./fastspar -t 50 -s 1013 --iterations 500 --exclude_iterations 200 --otu_table normalized.txt --correlation median_correlation.tsv --covariance median_covariance.tsv > fastspar.logs &
mkdir bootstrap_counts
nohup ./fastspar_bootstrap --otu_table normalized.txt --number 1000 --prefix bootstrap_counts/normalized > bootstrap.logs &
mkdir bootstrap_correlation
nohup parallel ./fastspar --otu_table {} --correlation bootstrap_correlation/cor_{/} --covariance bootstrap_correlation/cov_{/} -i 500 ::: bootstrap_counts/* > parallel_correlation.logs &
nohup ./fastspar_pvalues -t 50 --otu_table normalized.txt --correlation median_correlation.tsv --prefix bootstrap_correlation/cor_normalized_ --permutations 1000 --outfile pvalues.tsv > pvalues.logs &
```

结果还是很差。

直接用第一轮结果，用R画图：

```r
makeNet <- function(X){
  a<-matrix(nrow=1,ncol=4)
  a[1,1] <- rownames(sparcc$r)[X[1]]
  a[1,2] <- colnames(sparcc$r)[X[2]]
  a[1,3] <- sparcc$r[X[1], X[2]]
  a[1,4] <- sparcc$P[X[1],X[2]]
  return(a)
}

graph.transform.weights <- function (X) {
    require(igraph)
    data.tmp <- matrix(0, nrow=nrow(X), ncol=2)
    dimnames(data.tmp)[[2]] <- c("i", "j")
    data.tmp[,1] <- X[,1]
    data.tmp[,2] <- X[,2]
    data <- data.frame(data.tmp)
    graph.data <- graph.data.frame(data, directed=F)
    E(graph.data)$width <- abs(as.numeric(X[,3])) * 10 + 1
    E(graph.data)$arrow.width <- 2 - as.numeric(X[,4]) * 100
    summary(graph.data)
    cat("Average degree:",ecount(graph.data)/vcount(graph.data)*2)
    return(graph.data)
}

sparcc.cor <- read.table(file = "cor_lsa_input_999.tsv", sep = "\t", header = TRUE, row.names = 1, comment.char = '', check.names = FALSE)
dim(sparcc.cor)
sparcc.pval <- read.table(file = "pvalues.tsv", sep = "\t", header = TRUE, row.names = 1, comment.char = '', check.names = FALSE)
dim(sparcc.pval)
sparcc <- structure(list(r = sparcc.cor, P = sparcc.pval))
anno <- read.table('lsa_annotation.txt', header = TRUE, row.names = NULL)
names(anno) <- c('lineage', 'names', 'meanRA')
anno$meanRA <- as.double(anno$meanRA)
anno <- merge(anno, fuck, by = 'lineage', all.x = TRUE, all.y = FALSE)
anno$color[!anno$meanRA > 1] <- "darkgray"
anno$lineage[!anno$meanRA > 2] <- ""

pval <- 0.01
r <- 0.02
selrp <- which((abs(sparcc$r) > r & sparcc$P < pval) & lower.tri(sparcc$r == TRUE), arr.ind = TRUE)
sparcc.graph.df <- t(apply(selrp,1, makeNet))

sparcc.graph <- graph.transform.weights(sparcc.graph.df)
sparcc.graph.layout <- layout_with_fr(sparcc.graph, niter=9999)
# sparcc.graph.layout <- layout_in_circle(sparcc.graph)

sparcc.graph.names <- as.data.frame(V(sparcc.graph)$name, stringsAsFactors = F)
colnames(sparcc.graph.names) <- "names"

sparcc.graph.atr <- as.data.frame(merge(sparcc.graph.names, anno, by="names", all.x=TRUE, all.y = FALSE), stringsAsFactors = FALSE)
# sparcc.graph.atr <- data.frame(lapply(sparcc.graph.atr, as.character), stringsAsFactors=FALSE)
sparcc.graph.atr[is.na(sparcc.graph.atr)] <- 'FALSE'
sparcc.graph.atr.sorted <- sparcc.graph.atr[match(sparcc.graph.names$names,sparcc.graph.atr$names),]
V(sparcc.graph)$label <- sparcc.graph.atr.sorted$lineage
V(sparcc.graph)$size <- sparcc.graph.atr.sorted$meanRA + 1
V(sparcc.graph)$color <- sparcc.graph.atr.sorted$color

plot(sparcc.graph, vertex.label.cex = 0.8, layout=sparcc.graph.layout, asp = 0)
sparcc.graph <- delete.edges(sparcc.graph, E(sparcc.graph)[ arrow.width < 1.9 ])
plot(sparcc.graph, vertex.label.cex = 0.8, layout=sparcc.graph.layout, asp = 0)
```