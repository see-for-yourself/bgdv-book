---
layout: post
title:  "Appendix A"
date:   2024-10-15
categories: book
TOC_order: 11
copyright: "Copyright © 2024-2025, P. L. Chiu. All Rights Reserved."
---
## Installing Python

There are various ways to install Python, and there is an abundance of information on the Web on how to install it and how to install additional libraries and packages.

(1) The simple option is to install it directly from the Python website [<https://www.python.org>]. For example, on a Windows computer, you download the installer from the Python website and run it on your computer. In this case, you will need to separately install the extra libraries and packages used in this book (Matplotlib, Pandas, etc.)

(2) Another option is to install the Anaconda Python distribution [<https://www.anaconda.com>]. This includes many libraries used in this book. It includes Anaconda Navigator (a graphical user interface) for launching applications, managing environments and packages, and other tasks. It also includes the popular Visual Studio Code application for editing and running programs.

## Installing Libraries or Packages

(1) If you had installed Python from its website, you will need to install Matplotlib and other libraries used in this book.

For example, to install Matplotlib, first open a terminal. One way to do the installation is to enter the command on the Matplotlib website [<https://matplotlib.org/stable>]:

`pip install matplotlib`

Alternatively, use the command with the following options:

`python -m pip install -U matplotlib`

or in the Windows operating system:

`py -m pip install -U matplotlib`

With these more explicit commands, the `-m` option will execute the pip module, and the `-U` option will install the latest version if it has not already been installed.

(2) If you had installed the Anaconda Python distribution, it will include most of the libraries used in this book. Additional libraries can be install via the Anaconda Navigator GUI application. See:
[<https://www.anaconda.com/docs/tools/anaconda-org/user-guide/packages/installing-packages>]

## Installing Pandas and Numpy

Pandas is an open source data analysis and manipulation tool. It has a dependency on Numpy, which is a widely used library for scientific computing with Python. Installing Pandas automatically installs Numpy.

For more details about these libraries, see:

- Pandas: <https://pandas.pydata.org>
- Nummpy: <https://numpy.org>

## Installing GeoPandas and Geodatasets

GeoPandas and Geodatasets can be installed directly using `pip`, or using `conda install` if you have an Anaconda setup. For more details, see:

- GeoPandas: <https://geopandas.org/en/stable/>
- Geodatasets: <https://github.com/geopandas/geodatasets>
