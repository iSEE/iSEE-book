# (PART) API overview {-}

# Panel classes {#panels}



## Overview

This chapter provides a list of all of the classes that are implemented by the core *[iSEE](https://bioconductor.org/packages/3.12/iSEE)* package.
Each class comes with its specialized implementations of methods for various generics described in Chapter \@ref(api).
Thus, it is often possible for developers to inherit from one of these classes to get most of the relevant methods implemented "for free".
The classes themselves are either virtual or concrete; the latter can be created and used directly in an `iSEE()` application, 
while the former can only be used as a parent of a concrete subclass.
Here, we will provide a brief summary of each class along with a listing of its available slots.
Readers should refer to the documentation for each class (links below) for more details.

## Virtual classes

The [`Panel`](https://isee.github.io/iSEE/reference/Panel-class.html) class is the base class for all *[iSEE](https://bioconductor.org/packages/3.12/iSEE)* panels.
It provides functionality to control general panel parameters such as the panel width and height.
It also controls the transmission of multiple row/column selections across panels.

The [`DotPlot`](https://isee.github.io/iSEE/reference/DotPlot-class.html) class inherits from the `Panel` class and is the base class for dot-based plots.
This refers to all plots where each row or column is represented by no more than one dot (i.e., point) on the plot.
It provides functionality to create the plot, control the aesthetics of the points and to manage the brush/lasso selection.

The [`ColumnDotPlot`](https://isee.github.io/iSEE/reference/ColumnDotPlot-class.html) class inherits from the `DotPlot` class and represents all per-column dot plots.
This refers to all plots where each column is represented by no more than one dot on the plot.
It provides functionality to modify the plot aesthetics based on per-column values in the `colData` or assays.
It is also restricted to receiving and transmitting column identities in single and multiple selections.

The [`RowDotPlot`](https://isee.github.io/iSEE/reference/RowDotPlot-class.html) class inherits from the `DotPlot` class and represents all per-row dot plots.
This refers to all plots where each row is represented by no more than one dot on the plot.
It provides functionality to modify the plot aesthetics based on per-row values in the `rowData` or assays.
It is also restricted to receiving and transmitting row identities in single and multiple selections.

The [`Table`](https://isee.github.io/iSEE/reference/Table-class.html) class inherits from the `Panel` class and represents all tables rendered using *[DT](https://CRAN.R-project.org/package=DT)*.
Each row of the table is expected to correspond to a row or column of the `SummarizedExperiment`.
This class provides functionality to render the `DT::datatable` widget, monitor single/multiple selections and apply search filters.

The [`ColumnTable`](https://isee.github.io/iSEE/reference/ColumnTable-class.html) class inherits from the `Table` class and represents all tables where the rows have a one-to-zero-or-one mapping to columns of the `SummarizedExperiment`.
Instances of this class can only transmit single and multiple selections on columns.

The [`RowTable`](https://isee.github.io/iSEE/reference/RowTable-class.html) class inherits from the `Table` class and represents all tables where the rows have a one-to-zero-or-one mapping to rows of the `SummarizedExperiment`.
Instances of this class can only transmit single and multiple selections on rows.

## Concrete classes

The [`ReducedDimensionPlot`](https://isee.github.io/iSEE/reference/ReducedDimensionPlot-class.html) class inherits from the `ColumnDotPlot` class and plots reduced dimension coordinates from an entry of the `reducedDims` in a `SingleCellExperiment`.
It provides functionality to choose the result and extract the relevant dimensions in preparation for plotting.

The [`FeatureAssayPlot`](https://isee.github.io/iSEE/reference/FeatureAssayPlot-class.html) class inherits from the `ColumnDotPlot` class and plots the assay values for a feature across all samples, using an entry of the `assays()` from any `SummarizedExperiment` object.
It provides functionality to choose the feature of interest and any associated variable to plot on the x-axis, where the feature of interest can be chosen by a single selection from other row-transmitting panels.

The [`ColumnDataPlot`](https://isee.github.io/iSEE/reference/ColumnDataPlot-class.html) class inherits from the `ColumnDotPlot` class and plots `colData` variables by themselves or against each other.
It provides functionality to choose the variables to plot.

The [`SampleAssayPlot`](https://isee.github.io/iSEE/reference/SampleAssayPlot-class.html) class inherits from the `RowDotPlot` class and plots the assay values for a sample across all features, using an entry of the `assays()` from any `SummarizedExperiment` object.
It provides functionality to choose the sample of interest and any associated variable to plot on the x-axis, where the sample of interest can be chosen by a single selection from other column-transmitting panels.

The [`RowDataPlot`](https://isee.github.io/iSEE/reference/RowDataPlot-class.html) class inherits from the `RowDotPlot` class and plots `rowData` variables by themselves or against each other.
It provides functionality to choose and extract the variables to plot.

The [`ColumnDataTable`](https://isee.github.io/iSEE/reference/ColumnDataTable-class.html) class inherits from the `ColumnTable` class and shows the contents of the `colData` in a table.
It provides functionality to extract the `colData` in preparation for rendering.

The [`RowDataTable`](https://isee.github.io/iSEE/reference/RowDataTable-class.html) class inherits from the `RowTable` class and shows the contents of the `rowData` in a table.
It provides functionality to extract the `rowData` in preparation for rendering.

The [`ComplexHeatmapPlot`](https://isee.github.io/iSEE/reference/ComplexHeatmapPlot-class.html) class inherits from the `Panel` class and creates a heatmap from assay values using the *[ComplexHeatmap](https://bioconductor.org/packages/3.12/ComplexHeatmap)* package.
It provides functionality to specify the features to be shown, which assay to show, transformations to be applied, and which metadata variables to display as row and column heatmap annotations.

Further examples of concrete classes are also available in the *[iSEEu](https://bioconductor.org/packages/3.12/iSEEu)* package.

## Extending the classes

When creating a new panel class, it is necessary to inherit from a `Panel` subclass.
An appropriate choice of subclass can save a lot of work by providing that access to all of that subclass's generics.
This means that only a few methods need to be manually specialized to finish the panel, usually to change the output or interface.

**Case study 1.** 
If we wanted to create a `Panel` with a different plot for viewing the dimensionality reduction results, we could inherit from the `ReducedDimensionPlot` panel.
This gives us access to the interface elements to choose the reduced dimensions, plus the underlying code to extract the values for plotting.
Then, we only need to overwrite the generic responsible for generating the plot.
Note that the `ReducedDimensionPlot` is itself a subclass of the `ColumnDataPlot`, so there is an implicit assumption that each column of the `SummarizedExperiment` will be represented as a single point on the plot.
(Conceptually, at least. The point can be invisible.)

**Case study 2.**
Let's say we want to create a `Panel` to plot the assay values of a feature against the corresponding set of values in another assay, e.g., to plot gene expression against genotype in experiments with both RNA-seq and whole genome sequencing.
None of the existing concrete panels support the use of information from two separate assays, so we would not be able to inherit from them.
However, the plot would follow the expectations of the `ColumnDotPlot`, in that each column of the `SummarizedExperiment` would be represented no more than once on the plot.
Thus, we can inherit from the `ColumnDotPlot`, giving us automatic access to methods for managing the visual options and multiple column selections.
Then, we only need to define our own methods - to create the user interface elements to choose the two assays, and to extract those values in preparation for plotting.

**Case study 3.**
We might want to create a new heatmap that averages values across columns before displaying them.
This is not an uncommon request if our dataset contains many experimental groups and we want to compress them into averages for visualization.
However, we cannot easily inherit from the `ComplexHeatmapPlot` as it does not mandate the definition of a grouping factor.
We also cannot inherit from `DotPlot`s or `Table`s as we are not showing anything of the sort.
Thus, we must inherit from the `Panel` class and manually define methods for most of the generics.
(Though this task is not as hard as it seems, due to many utilities exported by *[iSEE](https://bioconductor.org/packages/3.12/iSEE)* to facilitate observer and interface set-up.)

**Case study 4.**
Finally, we want to create a table that dynamically computes statistics for each feature, e.g., for differential expression based on a user-selected factor.
We can thus inherit from the `RowTable` where each feature is represented by one row of the table.
Then, we just have to define interface elements to allow users to specify the nature of the computed statistics,
and to define a new method to actually generate the table of interest with said statistics.
