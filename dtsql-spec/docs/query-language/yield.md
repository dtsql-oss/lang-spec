# Yield

The yield statement ultimately determines the result of a DTSQL query.
Its syntax along with the supported yield formats are defined as follows:

```dtsql
'YIELD:'
  {'all periods'|'longest period'|'shortest period'|'data points'|'sample' <sampleIdentifier>|'samples' <sampleIdentifier>...}
```

Unlike the other components, the yield section is mandatory.
It determines one of six different result collection semantics to be used:

1. `all periods`:  If neither events nor selection component is present, an empty range of intervals is returned. If only the events component is present, then the set of captured event intervals is returned. If both events and selection component are present, all composite intervals are returned.
2. `longest period`: Returns the _first_ period (i.e., the one with the earliest start date) whose length is equal to the maximum duration of captured periods.
3. `shortest period`: Returns the _first_ period (i.e., the one with the earliest start date) whose length is equal to the minimum duration of captured periods.
4. `data points`: All data points that have not been filtered out and/or are contained in a t least one detected interval are returned.
5. `sample`: The computed value of the sample referenced by the placeholder `<sampleIdentifier>` is returned.
6. `samples`: The computed values of the samples referenced by the placeholders `<sampleIdentifier>` (separated by comma `,`) are returned.

The first example below returns the number of data points as well as the definite integral in the month of November in 2022.

??? example "Example: Yield Samples"
    ```dtsql
    WITH SAMPLES:
      integral("2022-11-01T00:00:00Z", "2022-11-30T23:59:59.999Z") AS novemberIntegral,
      count("2022-11-01T00:00:00Z", "2022-11-30T23:59:59.999Z") AS novemberCount
    YIELD:
      samples novemberIntegral, novemberCount
    ```

The second example below returns all periods that exhibit values that are continuously between 245.34 (inclusively) and the global average (exclusively).

??? example "Example: Yield All Periods"
    ```dtsql
    WITH SAMPLES:
      avg() AS myAvg
    USING EVENTS:
      AND(NOT(lt(245.34)), lt(myAvg)) AS inRange
    YIELD:
      all periods
    ```
