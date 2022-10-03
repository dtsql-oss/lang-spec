# DTSQL Query Overview

## Notational Conventions

This documentation follows (and goes beyond) common conventions known from usage messages (program synopsis), e.g., in UNIX man pages.
They are summarized below.

!!! abstract "Abstract: Notational Conventions for Syntax Definitions"
    * **Optional arguments `[a]`:** `a` is optional.
    * **Alternative options `[a|b]`:** May be left out, but if present, choose either `a` or `b`.
    * **Alternative required options `{a|b}`:** Must not be left out; choose either `a` or `b` .
    * **Conditionally valid options `[-a arg [-b]]`:** If `-a arg` is present, then optionally adding `-b` is also valid.
    * **Multiple occurrence `a...`:** A range of `a` occurrences is allowed. The context must specify both whether cardinalities `0..n` or `1..n` are valid as well as the separator to use (e.g., whitespace or comma).
    * **Placeholders `<a>`:** The placeholder `<a>` represents a concept that is explained later on or below.
    * **Lexicographic elements `'a'`:** Single-quoted text like `'a'` stands for lexicographic elements to be used verbatim in a query (to distinguish them from syntax definition symbols). Whitespaces are implicitly understood as such.

## Query Syntax

The [Getting Started](../index.md) page has already provided an exemplary query.
The general structure of DTSQL queries is as follows:

```dtsql
['WITH SAMPLES:'
  {<GlobalValueAggregate>|<LocalValueAggregate>|<TemporalAggregate>}
  'AS' <sampleIdentifier>...]

['APPLY FILTER:'
  {'AND('|'OR('}{'NOT('<filterFunction>')'|<filterFunction>}...')']

['USING EVENTS:'
  {'AND('|'OR('}{'NOT('<eventFunction>')'|<eventFunction>}...')'
  ['FOR' {'('|'['} [<minLength>]',' [<maxLength>] {')'|']'} <unit>]
  'AS' <eventIdentifier>...]
  
['SELECT PERIODS:'
  <selection>]

'YIELD:'
  {'all periods'|'longest period'|'shortest period'|'data points'|'sample' <sampleIdentifier>|'samples' <sampleIdentifier>...}
```

As already mentioned before, all components (apart from `YIELD`) are optional.
Furthermore, whitespaces do **not** carry semantics.
In other words, a DTSQL query may be formatted arbitrarily.
There are instances, however, that require a whitespace (space, new line, tab), e.g., after the colon (`:`) at the beginning of each component.

## Examples

In order to illustrate the syntax depicted above, the following paragraphs feature some (simple and not practice-oriented) exemplary queries along with their purpose.
The next pages will explain how the elements of the components mentioned above, and contained in the sample queries, are defined exactly.

??? example "Example: DTSQL Query (1)"
    Capture and return global, local and temporal aggregates:

    ```dtsql
    WITH SAMPLES:
      avg() AS globalAverage,
      integral("2017-09-09T10:00:00Z", "") AS integralTillTheEnd,
      sum_t(days, "2017-09-09T10:00:00Z/2019-09-04T10:00:00Z",
                  "2018-06-11T10:00:00Z/2019-01-08T10:00:00Z",
                  "2019-06-27T10:00:00Z/2019-09-12T10:00:00Z") AS totalDuration
    YIELD:
      samples globalAverage, integralTillTheEnd, totalDuration
    ```

??? example "Example: DTSQL Query (2)"
    Return data points after a temporal threshold which are greater than the global average and within a ±10 % range of the global maximum.

    ```dtsql
    WITH SAMPLES:
      avg() AS globalAvg,
      max() AS globalMax
    APPLY FILTER:
      AND(after("2018-11-12T07:12:59+03:30"), gt(globalAvg), around(rel, globalMax, 10))
    YIELD:
      data points
    ```

??? example "Example: DTSQL Query (3)"
    Capture periods where values are either above a hard-coded threshold or outside an (absolute) range of ±45 around a local average.
    Only consider periods that last less than 30 minutes.

    ```dtsql
    WITH SAMPLES:
      avg("", "2023-02-28T12:45:55-01:45") AS localAvg
    USING EVENTS:
      OR(gt(300.25), NOT(around(abs, localAvg, 45))) FOR [0,30) minutes AS myEvent
    YIELD:
      all periods
    ```

??? example "Example: DTSQL Query (4)"
    Return the longest period where a monotonic decrease directly precedes a constant period.

    ```dtsql
    USING EVENTS:
      AND(const(10.0, 5.0)) AS constantEvent,
      AND(increase(5.25, 25.5, 600)) FOR (2,5] weeks AS increaseEvent
    SELECT PERIODS:
      (constantEvent precedes increaseEvent)
    YIELD:
      longest period
    ```

??? example "Example: DTSQL Query (5)"
    First apply a temporal filter, then detect periods where a constant period appears before a switch from values above to below the global average, with not more than two days in between.

    ```dtsql
    WITH SAMPLES:
      avg() AS globalAvg
    APPLY FILTER:
      AND(NOT(after("2019-08-29T10:00:00Z")))
    USING EVENTS:
      AND(const(10, 5)) AS constantEvent
      AND(lt(globalAvg)) FOR [20,] days AS low,
      AND(gt(globalAvg)) FOR [20,] days AS high
    SELECT PERIODS:
      (constantEvent precedes (low follows high) WITHIN [0,2] days)
    YIELD:
      all periods
    ```
