+++
highlight = true
math = false
external_link = ""
summary = "Dr. Gao's strange requirement oriented toolbox, a tool box for bioinformatics."
date = "2017-05-16T15:53:20+08:00"
title = "Gao's SB!"
image = ""
image_preview = ""
tags = ['software']

+++

This project post is only for maintaining the process and problems, but not a document.

## Use PyQt5
Python has its internal GUI package `tkinter`, with which we can build lightweight GUI program. But to some complicated cases (e.g. program with many interactive functions and cascading widgets), PyQt's *QtDesigner* module is much easier: The uic module implements support for handling the XML files created by Qt Designer that describe the whole or part of a graphical user interface. It includes classes that load an XML file and render it directly, and classes that generate Python code from an XML file for later execution.

In order to avoid most of errors, you may use the [**latest *Anaconda Python* (with Python 3.x)**](https://www.continuum.io/downloads/), which contains PyQt5 of "most suitable" version itself. For Chinese user, there's [a mirror](https://mirrors.tuna.tsinghua.edu.cn/anaconda/archive/) here.

{{% alert warning %}}
**DO NOT** upgrade your PyQt5 yourself using such as `pip`, `conda` and etc. Or you'll get a "DLL not found" error as PyQt5 need correct `python3.dll` with interpreter. *Anaconda* always provides combination that can works well.
{{% /alert %}}

The PyQt5 wheels do not provide tools such as Qt Designer that were included in the old binary installers. Instead, there's another package on [*PyPi*](https://pypi.python.org/pypi/pyqt5-tools/), use `pip install pyqt5-tools` to install. You can use the **latest version** of pyqt5-tools, for there's always no conflict here.

## Design the UI of mainwindow
Qt's *stacked widget* object has been used for pack all tools in a single main widget in this project. I just add menu, stacked widget with items such as *text edit box*, *radio button* and so on within it through *QtDesigner*, and finally, run the command like `pyuic5 -o mainwindow.py gui.UI` to transform UI file (in *XML* format) into Python code. You can also use other similar tools.

{{% alert note %}}
It is highly recommended to **NOT** add *signal/slot* in *QtDesigner*, for `pyuic5` has parameter `-p` allows you to preview the UI without code,and you may lose flexibility when you're losing in OOP.
{{% /alert %}}

## Make one-folder or one-file bundle by *PyInstaller*


## Tool 1: A wrapped sequence aligner depended on *NCBI MagicBlast*
To be continued...

|| <!-- empty table header -->
|:--:| <!-- table header/body separator with center formatting -->
| I'm centered! | <!-- cell gets column's alignment -->