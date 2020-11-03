# Panel generics {#api}



## Overview

This chapter runs through all generics provided by *[iSEE](https://bioconductor.org/packages/3.12/iSEE)* to implement class-specific behaviors.
More exhaustive documentation about each generic can be obtained in the usual way, e.g., `?.defineInterface`.
Do not be intimidated; it is rarely necessary to define methods for all of the generics shown here.
If your class inherits from an existing `Panel` subclass, many of these methods will be implemented for free, and all you have to do is to override a handful of methods to achieve the desired customization.
To this end, examining the [R code](https://github.com/iSEE/iSEEu) underlying the various panels in the downstream *[iSEEu](https://bioconductor.org/packages/3.12/iSEEu)* package can be highly instructive.

## Class basics

Class names are expected to be formatted with camelCase, e.g., `ReducedDimensionPlot`.
Abbreviations are acceptable if they are well-understood, e.g., `MAPlot`, otherwise full words should be used to describe the class.

If your new class contains new slots beyond those provided by the parent, we suggest defining an `initialize()` method to specify the defaults for each new slot.
We ensures that any `new()` call to create your class will do something sensible, even if not all arguments are explicitly provided.
We prefer specializing `initialize()` rather than specifying `prototype=` as the former provides more flexibility, especially when there are dependencies between parameters.
Note that this usually requires a `callNextMethod()` to ensure that initialization of parent slots is also performed.

We also suggest defining an appropriate validity method via `setValidity()` to ensure that all user-provided arguments for new slots are valid.
Note that the validity method will not have access to the `SummarizedExperiment` so you cannot, e.g., check whether a particular feature name exists in the dataset - 
such checks are deferred to the `.refineParameters()` generic.
The validity method should only check the sensibility of a `Panel`'s slot values in isolation.

Finally, we define a lightweight constructor function with the same name as the class.
This should wrap `new()` to make it easy to construct a new instance of your class.

## Parameter set-up

One of the very first tasks performed by `iSEE()` is to run through the list of provided `Panel`s to set up constants and parameters.
This is done before the Shiny session itself is fully launched to ensure that the app is initialized in a valid state.

[`.cacheCommonInfo()`](https://isee.github.io/iSEE/reference/setup-generics.html) caches common values to be used for all instances of a particular panel class.
These cached values can be used to, e.g., populate the UI or set up constants to be used in the panel's output. 
This avoids potentially costly re-calculations throughout the lifetime of the `iSEE()` application.
Note that the cached values are generated only once for a given class and will be applied globally to all instances of that class;
thus, this method should only be storing class-specific constants.

[`.refineParameters()`](https://isee.github.io/iSEE/reference/setup-generics.html) edits the parameters of a panel to ensure that they are valid.
For example, we may need to restrict the choices of a selectize element to some pre-defined possibilities.
One can consider this generic to be the version of the validity method that has access to the `SummarizedExperiment`,
and thus can be used to "correct" the slot values based on the known set of valid possibilities.
This generic is run for each panel during the `iSEE()` application set-up to validate the user-supplied panel configuration.

## Defining the user interface

### In general

The next task performed by `iSEE()` is to define the user interface (UI) for each panel.
Each panel follows the general structure of having all UI elements contained in a `box` element with a panel-specific header color.
It is mandatory that each input UI element is named according to the `PANEL_SLOT` format,
where `PANEL` is the "encoded name" of the panel (see `?.getEncodedName`) and `SLOT` is the name of the slot that receives the input.

[`.defineInterface()`](https://isee.github.io/iSEE/reference/interface-generics.html) defines the panel's UI for modifying parameters.
Widgest should be bundled into collapsible boxes (see `?collapseBox`) according to their approximate purpose.
By default, two boxes are created containing data-related and selection-related parameters, though more boxes can be added in subclasses.
This generic provides the most general mechanism for controlling the panel's UI.

[`.defineDataInterface()`](https://isee.github.io/iSEE/reference/interface-generics.html) defines the UI for modifying all data-related parameters in a given panel.
Such parameters are fundamental to the interpretation of the panel's output, as opposed to their aesthetic counterparts.
This generic allows developers to fine-tune the data UI for subclasses without reimplementing the parent class's `.defineInterface()`, 
especially if we wish to re-use the parent's UI for visual-related parameters.
As each panel's data input is likely to require customization, this is the interface-related generic that has the greatest need for (non-trivial) specialization.

[`.defineSelectionEffectInterface()`](https://isee.github.io/iSEE/reference/interface-generics.html) defines the UI for controlling the effects of a multiple row/column selection.
For example, in a `DotPlot`, this generic could provide UI elements to change the color of all selected points.
The idea here is to, again, provide a simpler alternative to specializing `.defineInterface` when only the selection effect needs to be changed in a subclass.

[`.hideInterface()`](https://isee.github.io/iSEE/reference/interface-generics.html) determines whether certain UI elements should be hidden from the user.
This allows subclasses to hide easily inappropriate or irrelevant parts of the parent's UI, again without redefining `.defineInterface()` in its entirety.
For example, we can remove row selection UI elements for panels that only accept column selections.

[`.fullName()`](https://isee.github.io/iSEE/reference/getEncodedName.html) returns the full name of a panel class.
This is typically a more English-readable version of the camelCase'd class name.

[`.panelColor()`](https://isee.github.io/iSEE/reference/getPanelColor.html) is a very important generic that returns the color associated with the class.
This should be sufficiently dark that white text is visible on a background using this color.

### The `DotPlot` visual interface

For `DotPlot` subclasses, the default interface automatically includes another collapsible box containing visual-related parameters.
We provide a number of additional API points to change the visual-related UI for a subclass without completely reimplementing `.defineInterface()`.
Of course, if we have already specialized `.defineInterface()`, then there's no need to define methods for these generics.
Similarly, these generics do not need to be specialized if the defaults are adequate.

- [`.defineVisualColorInterface()`](https://isee.github.io/iSEE/reference/visual-parameters-generics.html) for color-related parameters. 
- [`.defineVisualFacetInterface()`](https://isee.github.io/iSEE/reference/visual-parameters-generics.html) for facet-related parameters.
- [`.defineVisualShapeInterface()`](https://isee.github.io/iSEE/reference/visual-parameters-generics.html) for shape-related parameters.
- [`.defineVisualSizeInterface()`](https://isee.github.io/iSEE/reference/visual-parameters-generics.html) for size-related parameters.
- [`.defineVisualPointInterface()`](https://isee.github.io/iSEE/reference/visual-parameters-generics.html) for other point-related parameters.
- [`.defineVisualTextInterface()`](https://isee.github.io/iSEE/reference/visual-parameters-generics.html) for text-related parameters.
- [`.defineVisualOtherInterface()`](https://isee.github.io/iSEE/reference/visual-parameters-generics.html) for other parameters.

## Creating observers

Once the interface is defined, `iSEE()` runs through all panels to set up its specific observers.
This is done once during app initialization and again whenever new panels are interactively added by the user.

[`.createObservers()`](https://isee.github.io/iSEE/reference/observer-generics.html) sets up Shiny observers for the panel in the current session.
This is the workhorse function to ensure that the panel actually responds to user input.
Developers can define arbitrarily complex observer logic here as long as it is self-contained within a single panel -
interactive mechanics that involve communication between panels are handled elsewhere.
One should also remember to call `callNextMethod()` to ensure that the parent class's observers are also defined.

Note that, unlike typical *[shiny](https://CRAN.R-project.org/package=shiny)* applications, the `input` never directly interacts with the `output`.
All observers in an *[iSEE](https://bioconductor.org/packages/3.12/iSEE)* panel are expected to change the application's "memory" upon changes to the `input` - this concept is discussed more in Chapter \@ref(server).
Most developers can ignore this subtlety by using *[iSEE](https://bioconductor.org/packages/3.12/iSEE)*-provided utilities to set up the observers rather than calling `observeEvent()` directly.

## Defining panel outputs

### In general

Finally, `iSEE()` runs through each panel to define its output elements and rendering expressions.
When the app appears on the browser, each rendering expression will be triggered to generate the desired visual output.
Panels transmitting multiple selections will also have their output explicitly generated beforehand by `iSEE()` so that downstream panels can be initialized properly.

[`.defineOutput()`](https://isee.github.io/iSEE/reference/output-generics.html) defines the interface element containing the output of the panel.
Examples include `plotOutput()` for plots or `dataTableOutput()` for tables.
Note that this generic only defines the output in the `iSEE()` interface; it does not control the rendering.

[`.renderOutput()`](https://isee.github.io/iSEE/reference/output-generics.html) assigns a reactive expression to populate the output interface element with content.
This is usually as simple as calling functions like `renderPlotOutput()` with an appropriate rendering expression containing a call to `.retrieveOutput()`.

[`.generateOutput()`](https://isee.github.io/iSEE/reference/output-generics.html) actually generates the panel output, be it a plot or table or something more exotic.
This is usually the real function that does all the work, being called by `.retrieveOutput()` prior to rendering the output. 
Some effort is required here to ensure that the commands used to generate the output are also captured.

[`.exportOutput()`](https://isee.github.io/iSEE/reference/output-generics.html) converts the panel output into a form that is downloadable, such as a PDF file for plots or CSVs for tables. 
This is called whenever the user requests a download of the panel outputs.

### For `DotPlot`s

For `DotPlot`s, additional generics are provided to customize specific aspects of the output.
These can be specialized to achieve the desired output without rewriting `.generateOutput()` in its entirety.
Of course, these are all optional and can be left as the defaults if those are satisfactory.

[`.generateDotPlot()`](https://isee.github.io/iSEE/reference/plot-generics.html) creates the `ggplot` object for `DotPlot` subclasses, given a `data.frame` of data inputs.
Developers can specialize this generic if they only need to change the visualization while continuing to use the default data management.

[`.generateDotPlotData()`](https://isee.github.io/iSEE/reference/plot-generics.html) creates the `data.frame` that is used by `.generateDotPlot()`.
This allows developers to change the data setup for a `DotPlot` subclass without having to specialize `.generateDotPlot()`, if they are satisfied with the default `DotPlot` aesthetics.

[`.prioritizeDotPlotData()`](https://isee.github.io/iSEE/reference/plot-generics.html) determines how points should be prioritized during overplotting.
This usually doesn't need to be specialized but can be helpful if some points are more important than others (e.g., DE genes versus non-DE genes in a volcano plot).

[`.colorByNoneDotPlotField()`](https://isee.github.io/iSEE/reference/plot-generics.html) and [`.colorByNoneDotPlotScale()`](https://isee.github.io/iSEE/reference/plot-generics.html) define the default color scale when `ColorBy="None"`.
This usually doesn't need to be specialized but can be helpful, e.g., to change the color of DE genes according to the sign of the log-fold change.

[`.allowableYAxisChoices()`](https://isee.github.io/iSEE/reference/metadata-plot-generics.html) and [`.allowableXAxisChoices()`](https://isee.github.io/iSEE/reference/metadata-plot-generics.html) specifies the acceptable fields for the x- or y-axes of `ColumnDataPlot` or `RowDataPlot` subclasses.
This is typically used to constrain the choices for customized panels that only accept certain column names or types.
For example, a hypothetical MA plot panel would only accept log-fold changes on the y-axis.

### For `Table`s

For `Table`s, the most important aspect is the generation of the underlying `data.frame`.
This can be customized without requiring the developer to rewrite the *[DT](https://CRAN.R-project.org/package=DT)*-related rendering of the table.

[`.generateTable()`](https://isee.github.io/iSEE/reference/table-generics.html) creates the `data.frame` that is rendered into the table widget for `Table` subclasses.
Each row of the `data.frame` is generally expected to correspond to a row or column of the dataset.
If this is specialized, there is usually no need to specialize `.generateOutput()` for such subclasses.

## Handling selections

### Multiple

Some panels can transmit a selection of multiple row or columns (never both) to other panels.
`iSEE()` determines whether a particular panel is a multiple selection transmitter along the rows or columns (or neither) by interrogating a suite of generics.
Most new panels do not have to care about this if they inherit from `RowDotPlot`, `RowTable`, `ColumnDotPlot` or `ColumnTable`;
however, more custom panels will have to specialize these generics manually if they intend to transmit multiple selections.

[`.multiSelectionDimension()`](https://isee.github.io/iSEE/reference/multi-select-generics.html) specifies whether the panel transmits multiple selections along the rows or columns.
It can also be used to indicate that the panel does not transmit anything.

[`.multiSelectionActive()`](https://isee.github.io/iSEE/reference/multi-select-generics.html) returns the parameters that define the "active" multiple selection in the current panel.
This is defined as the selection that the user can actively change by interacting with the panel.
(In contrast, the "saved" selections are fixed and can only be deleted.)

[`.multiSelectionCommands()`](https://isee.github.io/iSEE/reference/multi-select-generics.html) creates the character vector of row or column names for a multiple selection in the current panel. 
More specifically, it returns the commands that will then be evaluated to generate such character vectors.
The identity of the selected rows/columns will ultimately be transmitted to other panels to affect their behavior.

[`.multiSelectionAvailable()`](https://isee.github.io/iSEE/reference/multi-select-generics.html) reports how many total points are available for selection in the current panel.
This is used for reporting "percent selected" statistics below each panel.

[`.multiSelectionClear()`](https://isee.github.io/iSEE/reference/multi-select-generics.html) eliminates the active multiple selection in the current panel.
This is used to wipe selections in response to changes to the plot content that cause those selections to be invalid.

[`.multiSelectionRestricted()`](https://isee.github.io/iSEE/reference/multi-select-generics.html) indicates whether the current panel's data should be restricted to the rows/columns that it receives from an incoming multiple selection.
This is used to determine how changes in the upstream transmitters should propagate through to the current panel's children.

[`.multiSelectionInvalidated()`](https://isee.github.io/iSEE/reference/multi-select-generics.html) indicates whether the current panel is invalidated when it receives a new multiple selection.
This usually doesn't need to be specialized.

### Single 

Some panels can transmit a identity of a single feature or sample to other panels.
`iSEE()` determines whether a particular panel is a single selection transmitter along the features or samples (or neither) by interrogating a suite of generics.
Most new panels do not have to care about this if they inherit from `RowDotPlot`, `RowTable`, `ColumnDotPlot` or `ColumnTable`;
however, more custom panels will have to specialize these generics manually if they intend to transmit single selections.

[`.singleSelectionDimension()`](https://isee.github.io/iSEE/reference/single-select-generics.html) specifies whether the panel transmits single selections of a row or column.
It can also be used to indicate that the panel does not transmit anything.

[`.singleSelectionValue()`](https://isee.github.io/iSEE/reference/single-select-generics.html) determines the row or column that has been selected in the current panel.
The identity of the row/column is passed onto other panels to affect their behavior.

[`.singleSelectionSlots()`](https://isee.github.io/iSEE/reference/single-select-generics.html) determines how the current panel should respond to single selections from other panels.
This will also automatically set up some of the more difficult observers if sufficient information is supplied by the class.

## Miscellaneous

[`.definePanelTour()`](https://isee.github.io/iSEE/reference/documentation-generics.html) defines an *[rintrojs](https://CRAN.R-project.org/package=rintrojs)* tour for the functionalities of the current panel.
This guides users through a short tour of the current panel's most important features, reducing the need to consult external documentation.
