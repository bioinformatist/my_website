+++
title = "从PacBio转录组原始数据到？？？"
date = 2019-03-21T03:10:11+08:00
draft = false

# Tags: can be used for filtering projects.
# Example: `tags = ["machine-learning", "deep-learning"]`
tags = ['PacBio']

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

## 文件组织

将下机数据中所有的`*.subreads.bam`和`*.subreads.bam.pbi`堆在一个目录下。

## 流程主体（shell简陋版本）

先保证有一个可用的流程，都是用shell命令串起来的，虽然是串行，但是在单条命令中尽量利用并发，效率还可以，只是不能在集群上跨节点计算。

```shell
# qsub -I -l nodes=cu01:ppn=27,mem=180gb  # 你可以用这个申请27个CPU和180GB的内存去交互式计算，也可以将下述命令全部写到一个脚本中用qsub提交
source activate anaCogent5.2
for x in *.bam; do ccs $x $(basename $x .subreads.bam).ccs.bam --minPasses 1 --minPredictedAccuracy=0.8 --noPolish -j 27; done
```