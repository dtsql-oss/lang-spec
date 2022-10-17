# DTSQL Documentation

DTSQL (_"Declarative Time Series Query Language"_) is a high-level domain-specific language geared towards querying time series data (sensor data) declaratively.
This documentation provides a specification of DTSQL, i.e., its language features, their syntax and semantics as well as illustrating example queries.

## Fundamental Language Concepts

In general, a DTSQL query consists of five components:

1. A set of [samples](query-language/samples.md). They represent scalar values expressed by aggregation functions (e.g., `avg`, `sum`). These values may be referenced/reused in other query components (e.g., filters or events) or also serve as overall query result.
2. A [filter](query-language/filters.md) specification that excludes irrelevant data points of the input time series from the query evaluation.
3. A set of [events](query-language/events.md). They are a declarative specification of periods (intervals) to be captured during query evaluation.
4. A [selection](query-language/selection.md) component which defines temporal relations between detected event periods (e.g., `precedes`, `follows`). This allows one to express that events ought to occur in a certain sequence in order to be part of the query result.
5. A [yield](query-language/yield.md) statement which ultimately determines the query result. The actual value of this statement decides whether the result is made up of data points, intervals or sample values.

All components, except for _yield_, are optional.
More specifically, this means:

??? note "Note: Required/Optional DTSQL Components"

    * the samples component may be omitted
    * the filter component may be omitted
    * the events component may be omitted
    * the selection component may be omitted
    * the yield component is mandatory

## Temporal Concepts and Result Objects

DTSQL utilizes notions such as time series, data points, time intervals (periods) and points in time in general.
They are mostly encapsulated in their own respective data types.
All of them are explained in detail in the section dedicated to [data structures](data-structures/data-structures.md).

## Providing Input Data

A main goal of DTSQL is keeping it generic and independent of a concrete underlying storage mechanism.
Therefore, queries expressed in this language operate on a canonical representation of time series.
This representation is explained in the section about the [data structures](data-structures/data-structures.md).
In addition to that, the section about [query API](query-api/query-api-overview.md) explains how arbitrary data sources may be connected to and queried using DTSQL.

## Example Query

???+ example "Example: Query Utilizing Core Language Features"

    ```dtsql
    WITH SAMPLES:
      avg() AS globalArithmeticMean,
      min("2022-05-28T14:15:00Z", "") AS localMinimum
    APPLY FILTER:
      AND(NOT(before("2022-05-28T14:15:00Z")))
    USING EVENTS:
      AND(gt(globalArithmeticMean)) FOR (30,] seconds AS aboveAvg,
      AND(around(rel, localMinimum, 25)) FOR [2,] minutes AS aroundMin
    SELECT PERIODS:
      (aboveAvg follows aroundMin WITHIN [,4) seconds)
    YIELD:
      all periods
    ```

The "teaser" query above has the following meaning:

??? info "Info: Explanation of Example Query"

    1. declare two samples
        * a _global_ average (considering the entire input time series)
        * a _local_ minimum value (from `2022-05-28T14:15:00Z` until the end of the input time series)
    2. filter out data points which were measured before the very same point in time `2022-05-28T14:15:00Z`
    3. characterize two kinds of periods (events)
        * those with values consistently above the global average for more than 30 seconds
        * those with ony values within a ±25 % range of the previously declared local minimum for at least two minutes
    4. select composite (merged) periods capturing binary event sequences where, within less than four seconds, a periods of the first event _follows_ (occurs after) a period of the second event
    5. yield these captured composite periods as final query result 
