+++
date = "2017-07-13T11:53:32+08:00"
external_link = ""
highlight = true
image_preview = ""
math = false
summary = "DY Quant -- Donate Yourself Quantopian, perform your own investment algorithms"
tags = ['software']
title = "DY Quant"

[header]
  caption = ""
  image = ""

+++

My python interpreter is version 3.6, when I try install `zipline` package by `conda`:

```bash
conda install -c Quantopian zipline
```

It raises:

```pre
Fetching package metadata .............
Solving package specifications: .

UnsatisfiableError: The following specifications were found to be in conflict:
  - python 3.6*
  - zipline -> alembic >=0.7.7 -> python 3.4*
Use "conda info <package>" to see the dependencies for each package.
```

So it seems I need a virtualenv with Python 3.4:

```bash
conda create -n py34ForZipline python=3.4
```

To activate this environment and install `zipline`:

```bash
activate py34ForZipline
conda install -c Quantopian zipline
```

Package `talib` is also needed for running examples:

```bash
conda install -c Quantopian ta-lib
```

