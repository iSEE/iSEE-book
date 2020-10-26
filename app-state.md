# Application state {#server}



## Overview

*[iSEE](https://bioconductor.org/packages/3.11/iSEE)* uses global variables to keep track of the application state and to trigger reactive expressions.
These are passed in the ubiquitous `pObjects` and `rObjects` arguments for non-reactive and reactive variables, respectively.
Both of these objects have pass-by-reference semantics, meaning that any modifications to their contents within functions will persist outside of the function scope.
This enables their use in communicating changes across all components of the running `iSEE()` application.

For most part, developers of new panels do not need to be aware of these variables.
Only panels with relatively complex customizations need to manually specify the reactive logic or memory updates,
in which case they should use the various utilities provided by *[iSEE](https://bioconductor.org/packages/3.11/iSEE)* to mediate the interactions with `rObjects` and `pObjects`.
Developers should also refrain from adding their own application-wide variables.
Respecting this paradigm will ensure that custom panels behave correctly in the context of the entire application.

## Updating parameters

The application memory is a list of `Panel` instances in `pObjects$memory` that captures the current state of the *[iSEE](https://bioconductor.org/packages/3.11/iSEE)* application.
Conceptually, one should be able to extract this list from a running application, pass it to the `initial=` argument of the `iSEE()` function and expect to recover the same state.
All modifications to the state should be recorded in the memory, meaning that observer expressions will commonly contain code like:

```r
pObjects$memory[[panel_name]][[param_name]] <- new_value
```

By itself, modifying the application memory will not trigger any further actions.
The memory is too complex to be treated as a reactive value as it would affect too many downstream observers.
Instead, we provide the `.requestUpdate()` function to indicate to the application that a particular panel needs to be updated.
This sets a flag in `rObjects` that will eventually trigger re-rendering of the specified panel.

The `.requestCleanUpdate()` function provides a variant of this approach where the panel should be updated _and_ any active or saved multiple selections should be wiped.
This is useful for dealing with changes to "protected" parameters that modify the panel contents such that any selection parameters are no longer relevant (e.g., invalidating brushes when the plot coordinates change).
Yet another variant is the `.requestActiveSelectionUpdate()` function, which indicates whether a panel's active multiple selection has changed; this should be used in the observer expression that responds to the panel's multiple selection mechanism.

The two-step process of memory modification and calling `.requestUpdate()` is facilitated by functions like `.createUnprotectedParameterObservers()`, which sets up simple observers for parameter modifications.
However, more complex observers will have to do this manually.

## Reading the memory

In a similar vein, expressions to render output should _never_ touch the Shiny `input` object directly.
(Indeed, `.renderOutput()` does not even have access to the `input`.)
As all parameter changes pass through the memory, the updated values of each parameter should also be retrieved from memory.
This involves extracting the desired `Panel` from `pObjects$memory` in methods for generics like `.createObservers()` that rely on pass-by-reference semantics for correct evaluation of reactive expressions.
Other generics that are not setting up reactive expressions can directly extract values from the supplied `Panel` object.

Each `Panel` object can be treated as a list of panel parameters.
Retrieving values is as simple as using the `[[` operator with the name of the parameter.
(Similarly, setting parameters is as easy as using `[[<-`, though this should only be done in dedicated observers and never in rendering expressions.)
Direct slot access should be avoided, consistent with best practice for S4 programming.

## Reacting to events

Developers can respond to events by calling functions like `.trackUpdate()` within an observer or rendering expression.
This touches `rObjects` to ensure that the enclosing expression is re-evaluated if the panel is updated elsewhere by `.requestUpdate()`.
Other variants like `.trackMultiSelection()` will trigger re-evaluation upon changes to the panel's multiple selections.

Direct use of `.trackUpdate()` and related functions is generally unnecessary as it is handled by higher-level functions like `.retrieveOutput()`.
Nonetheless, if some action needs to be taken after, e.g., a multiple selection, it may be appropriate to create an observer that calls `.trackMultiSelection()`.
Note that developers should only use these functions to track updates to the same panel for which the observer/rendering expression is written; 
management of communication across panels is outside of the scope of these expressions.

## Guidelines for user globals

Developers are free to define global parameters that affect all instances of their panel class.
This makes it easy for the user to modify the behavior of all instances of a particular panel.
However, we suggest that such user-visible globals limit their effects to the panel's constructor.
This enables users to reproduce the app state from one session to another by simply saving the memory; 
otherwise, users would also have to export the state of the global variables.

This guideline implies that any global parameters should be represented as slots in the panel class.
Technically, this also means that different instances of a particular class might have different values for that same slot.
If all panels must have the same value, this can be enforced via some creative use of `.cacheCommonInfo` and `.refineParameters`;
see, for example, the `MAPlot` class from the *[iSEEu](https://bioconductor.org/packages/3.11/iSEEu)* package.
