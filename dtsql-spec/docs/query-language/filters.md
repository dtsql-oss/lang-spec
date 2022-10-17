# Filters

A filter enables query creators to exclude data points of an input time series from the query evaluation.
They are applied _after_ the computation of samples (otherwise they could not be referenced in filter definitions).
DTSQL filters are conceptually simple in that they do not apply complex transformations as, e.g., a low-pass filter does.
They merely decide whether a data point should be included in the query evaluation, solely based on the information provided by the respective data point itself.

A DTSQL query contains zero to one filter specifications.
The enumeration below summarizes how filters work in general.
A filter specification is a connection of filter functions in a propositional logical manner.

!!! abstract "Abstract: Principles of DTSQL Filters"
    1. A filter specification consists of multiple filter function applications.
    2. A filter function is always evaluated based on a single data point.
    3. Filter functions are connected to each other in a propositional logical manner.
    4. A data point is included in the query evaluation if and only if the thus resulting propositional formula is satisfied given its time and value component.

In order to compose a propositional formula of filter function instances, the following keywords are available:

* conjunction: `AND` (n-ary)
* disjunction: `OR` (n-ary)
* negation: `NOT` (unary)

???+ warning "Warning: Syntactic Restrictions of Filter Formulas"
    At the moment, DTSQL does **not** support nested conjunctions and disjunctions.
    This means, formulas like `AND(a, b, NOT(c))` or `OR(NOT(a), NOT(b), c)` are expressible, but ones like `AND(AND(a), b)` or `NOT(AND(a, b), OR(NOT(c), d))` are not.

    More specifically, root elements of a formula must be either `AND` or `OR` and may only contain arbitrarily many positive or negated (using `NOT`) filter function instances. 

As a result of this, the syntax of DTSQL's filter component is:

```dtsql
['APPLY FILTER:'
  {'AND('|'OR('}{'NOT('<filterFunction>')'|<filterFunction>}...')']
```

Again, the component is optional.
However, should it be present, then at least one filter function instance must be declared.
The `<filterFunction>` placeholder is to be replaced by filter function instances as explained below (with commas `,` as separator).
Note that, if only one filter function instance is used, then the choice of `AND` vs. `OR` does not make a difference.

## Filter Functions

### Threshold Filters

`lt({<threshold>|<sampleIdentifier>})`

:   Is satisfied if the value component is less than the threshold (which may refer to a sample).

`gt({<threshold>|<sampleIdentifier>})`

:   Is satisfied if the value component is greater than the threshold (which may refer to a sample).

### Deviation Filters

`around(abs, {<reference>|<sampleIdentifier>}, {<deviation>|<sampleIdentifier>})`

:   Is satisfied if the absolute difference between the value component and the reference value (which may refer to a sample) is less than the maximum deviation (which may refer to a sample).

`around(rel, {<reference>|<sampleIdentifier>}, {<deviation>|<sampleIdentifier>})`

:   Is satisfied if the relative (percentage-wise) difference between the value component and the reference value (which may refer to a sample) is less than the maximum deviation (which may refer to a sample). The reference value must not be equal to zero.

### Temporal Filters

`before(<threshold>)`

:   Is satisfied if the time component is before the temporal threshold.

`after(<threshold>)`

:   Is satisfied if the time component is after the temporal threshold.

## Example

The query below illustrates one of the three types of filter functions from above (all three conditions should be fulfilled):

1. Only keep data points starting with June 24th 2021 13:24 UTC.
2. Filter out data points which are equal to or less than 240.
3. Only keep data points which are in a ±5.25 % range of the global average.

??? example "Example: Filter Component"
    ```dtsql
    WITH SAMPLES:
      avg() AS globalAvg
    APPLY FILTER:
      AND(
           NOT(before("2021-06-24T13:24:00Z")),
           gt(240),
           around(rel, globalAvg, 5.25)
         )
    ```
