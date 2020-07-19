--- 
title: "Extending _iSEE_"
author: Kevin Rue-Albrecht, Federico Marini, Charlotte Soneson, and Aaron Lun
date: "2020-07-19"
site: bookdown::bookdown_site
documentclass: book
bibliography: [book.bib, packages.bib]
biblio-style: apalike
link-citations: yes
description: "This book describes how to use the Bioconductor iSEE package to create web-applications for exploring data stored in SummarizedExperiment objects."
favicon: "favicon_package_v0.16/favicon.ico"
---



# Preface {-}

<a href="https://bioconductor.org/packages/iSEE"><img src="https://github.com/Bioconductor/BiocStickers/raw/master/iSEE/iSEE.png" width="200" alt="iSEE Sticker" align="right" style="margin: 0 1em 0 1em" /></a> 

The [Bioconductor](https://bioconductor.org/) package *[iSEE](https://bioconductor.org/packages/3.11/iSEE)* provides functions to create an interactive graphical user interface (GUI) using the [RStudio](https://rstudio.com/) *[Shiny](https://CRAN.R-project.org/package=Shiny)* package for exploring data stored in *[SummarizedExperiment](https://bioconductor.org/packages/3.11/SummarizedExperiment)* objects [@rue2018isee].
This book describes how to use *[iSEE](https://bioconductor.org/packages/3.11/iSEE)*'s application programming interface (API) to develop new panel types for custom visualizations.
We also present case studies to illustrate the development process for a variety of custom panels.

The contents of this book are intended for developers of custom panel classes, usually inside a dedicated package like *[iSEEu](https://bioconductor.org/packages/3.11/iSEEu)*.
Potential end-users of *[iSEE](https://bioconductor.org/packages/3.11/iSEE)* should refer to the vignettes provided on the package landing page.


