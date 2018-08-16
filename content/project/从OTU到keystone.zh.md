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
parallel fastspar --otu_table {} --correlation bootstrap_correlation/cor_{/} --covariance bootstrap_correlation/cov_{/} -i 500 ::: bootstrap_counts/*
nohup parallel ./fastspar --otu_table {} --correlation bootstrap_correlation/cor_{/} --covariance bootstrap_correlation/cov_{/} -i 500 ::: bootstrap_counts/* > parallel_correlation.logs &
nohup ./fastspar_pvalues -t 50 --otu_table lsa_input.txt --correlation median_correlation.tsv --prefix bootstrap_correlation/cor_lsa_input_ --permutations 1000 --outfile pvalues.tsv > pvalues.logs &
```