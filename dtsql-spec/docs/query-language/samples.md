# Samples

A sample is a scalar, real value computed based on an input time series.
The computation of a sample is never reliant upon other, previously computed samples (they are independent of each other).
Essentially, samples are _aggregate_ values that can be (re-)used to make other components of the query depend on them (e.g., to compose [filters](filters.md) or define [events](events.md)).
They may also serve as [overall return value](yield.md) of a query.

The syntax of the samples section is as follows:

```dtsql
['WITH SAMPLES:'
  {<GlobalValueAggregate>|<LocalValueAggregate>|<TemporalAggregate>}
  'AS' <sampleIdentifier>...]
```

In other words, the section is optional, but if it is present, then one or more samples must be declared (with the comma `,` as separator).
Each sample consists of an _aggregate function_ and an _identifier_.
Furthermore, there are three types of samples.
_Global value aggregates_ operate on the value component of all data points, _local value aggregates_ only on those in a specifiable period, and _temporal aggregates_ operate on the durations of several periods.
The aggregate functions supported by DTSQL are introduced below.

???+ abstract "Abstract: DTSQL Aggregate Functions"
    
    | value       | temporal   | semantics                                                 |
    |-------------|------------|-----------------------------------------------------------|
    | `max`       | `max_t`    | maximum value/interval length                             |
    | `min`       | `min_t`    | minimum value/interval length                             |
    | `avg`       | `avg_t`    | arithmetic mean of values/interval lengths                |
    | `count`     | `count_t`  | number of values/intervals                                |
    | `sum`       | `sum_t`    | sum of values/interval lengths                            |
    | `integral`  | n/a        | definite integral ("area") of function modelled by input time series (with linear interpolation between data points) with time axis/time differences scaled to seconds  |
    | `stddev`    | `stddev_t` | population standard deviation of values/interval lengths  |

The identifier must start with a letter, followed by an arbitrary number of digits and/or letters.

## Global Value Samples
Global value samples have the following syntax:

```dtsql
<valueFunction>() AS <sampleIdentifier>
```

* The `<valueFunction>` placeholder ought to be substituted by a value aggregate function from the first column in the table above.
* The `<sampleIdentifier>` placeholder represents an identifier as explained above.

The example below defines two global value samples, an arithmetic mean and the integral.

??? example "Example: Global Value Samples"
    ```dtsql
    WITH SAMPLES:
      avg() AS globalAvg,
      integral() AS globalIntegral
    ```

## Local Value Samples
Local value samples have the following syntax:

```dtsql
<valueFunction>('"'[<lowerBound>]'"', '"'[<upperBound]'"'}) AS <sampleIdentifier>
```

* The `<valueFunction>` placeholder ought to be substituted by a value aggregate function from the first column in the table above.
* The placeholders `<lowerBound>` and `<upperBound>` are valid [timestamps](../data-structures/data-structures.md#time) which denote the boundaries that should be considered during sample calculation. More precisely, only data points of the input time series are considered whose time component is not earlier than `<lowerBound>`, and also not later than `<upperBound>`.
* If `<lowerBound>`/`<upperBound>` is not present, then the used bound is the start/end timestamp of the input time series. 
* The `<sampleIdentifier>` placeholder represents an identifier as explained above.

The example below defines two local value samples, the standard deviation of values between March 4th 2021 and April 28th 2021 (both 12 o'clock UTC), as well as the overall number of data points recorded starting with April 1st 2021.

??? example "Example: Local Value Samples"
    ```dtsql
    WITH SAMPLES:
      stddev("2021-03-04T12:00:00Z", "2021-04-28T12:00:00Z") AS localSample1,
      count("2021-04-01T00:00:00Z", "") AS localSample2
    ```

## Temporal Samples

Local value samples have the following syntax:

```dtsql
<temporalFunction>(<unit>, <interval>...) AS <sampleIdentifier>
```

* The `<temporalFunction>` placeholder ought to be substituted by a temporal aggregate function from the second column in the table above.
* The `<unit>` placeholder represents a [supported time unit](../data-structures/data-structures.md#time).
* The `<interval>` placeholder is a collection of intervals over whose durations the sample should be calculated. They are separated by commas (`,`) and expressed in an [appropriate format](../data-structures/data-structures.md#time).
* The `<sampleIdentifier>` placeholder represents an identifier as explained above.

!!! warning "Warning: Irregularities With Temporal Samples"
    1. There is no temporal sample corresponding to the `integral` function because the notion of integrals over interval duration is not reasonable.
    2. The `count_t` function **does not take a unit argument** because that would not make sense. 

The example below illustrates temporal samples by means of calculating the total duration of some periods as well as their number.

??? example "Example: Temporal Samples"
    ```dtsql
    WITH SAMPLES:
      sum_t(minutes, "2022-08-28T17:00:00Z/2022-08-28T22:00:00Z",
                     "2022-08-29T00:00:00Z/2022-08-29T02:00:00Z",
                     "2022-08-29T05:00:00Z/2022-08-29T13:00:00Z") AS totalDuration,
      count_t("2022-08-28T17:00:00Z/2022-08-28T22:00:00Z",
              "2022-08-29T00:00:00Z/2022-08-29T02:00:00Z",
              "2022-08-29T05:00:00Z/2022-08-29T13:00:00Z") AS totalPeriods
    ```
