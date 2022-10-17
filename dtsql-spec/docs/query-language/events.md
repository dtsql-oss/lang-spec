# Events

An event is a declarative specification of intervals in which some condition(s) hold(s).
In other words, they are used to detect periods where areas of interest occur.
Events are evaluated _after_ filter application.
In fact, they extend the notion of filters by considering detected intervals as _continuous_ (i.e., without interruption) (sub-)series of data points for which a propositional formula composed of event function instances is satisfied.
The difference to filters is that, with events, data points are not examined individually anymore, but they are grouped together to form periods satisfying the criteria imposed by the formula.

!!! abstract "Abstract: Types of DTSQL Events"
    1. _Filter Events_: They are syntactically and semantically equivalent to [filter functions](filters.md#filter-functions) with respect to the filtered time series.
    2. _Complex Events_: They are broader and, typically, conceptually more involved and may capture any type of occurrence in a time series as long as it can be specified formally.

There is one key commonality among all kinds of event functions.
Generally, DTSQL is only interested in _maximal interval_, i.e., ones that cannot be extended without violating the event function.
For filter events, this means that extensions of the respective interval by exactly one data point to the left and right, respectively, are examined (looking further than that could produce intervals with gaps, which are not considered).
For complex events, the situation is more difficult: It may be possible to extend an interval in either direction (temporarily violating the event function) and extending it even more to, in the end, satisfy it again.
In other words, extending a valid _locally maximal_ interval might yield a valid _globally maximal_ interval despite potentially invalidating it temporarily.

In order to compose a propositional formula of event function instances, the following keywords are available:

* conjunction: `AND` (n-ary)
* disjunction: `OR` (n-ary)
* negation: `NOT` (unary)

???+ warning "Warning: Syntactic Restrictions of Event Formulas"
    At the moment, DTSQL does **not** support nested conjunctions and disjunctions.
    This means, formulas like `AND(a, b, NOT(c))` or `OR(NOT(a), NOT(b), c)` are expressible, but ones like `AND(AND(a), b)` or `NOT(AND(a, b), OR(NOT(c), d))` are not.

    More specifically, root elements of a formula must be either `AND` or `OR` and may only contain arbitrarily many positive or negated (using `NOT`) filter function instances. 

??? bug "Bug: Shortcoming of Reference Implementation Regarding Event Formulas"
    The current DTSQL reference implementation is not yet able to combine a complex event function instance with other event function instances.
    In other words, if an event formula contains a complex event function instance, it must not contain any other event function instance.
    Mixing and combining filter event functions is supported the same way it is for [filter definitions](filters.md#filters), though. 

In summary, the syntax of DTSQL's events component is:

```dtsql
['USING EVENTS:'
  {'AND('|'OR('}{'NOT('<eventFunction>')'|<eventFunction>}...')'
  ['FOR' {'('|'['} [<minLength>]',' [<maxLength>] {')'|']'} <unit>]
  'AS' <eventIdentifier>...]
```

Again, the component is optional.
However, should it be present, then at least one event function instance must be declared.
The `<eventFunction>` placeholder is to be replaced by event function instances as explained below (with commas `,` as separator).
Note that, if only one event function instance is used, then the choice of `AND` vs. `OR` does not make a difference.
Unlike the filter component, the events component may declare multiple events (with commas `,` as separator) and assign identifiers to them such that they may be reused in the [selection component](selection.md).
Furthermore, an event may be equipped with a duration constraint: `<minLength>` and `<maxLength>` are optional integers specifying the required length of the event (leaving them out represents 0 and "infinity", respectively).
The inclusiveness or exclusiveness of the minimum and maximum is determined based on the parenthesis used:

|           | lower bound | upper bound |
|-----------|-------------|-------------|
| inclusive | `[`         | `]`         |
| exclusive | `(`         | `)`         |

Finally, `<unit>` should be replaced by a [supported time unit](../data-structures/data-structures.md#time).

## Event Functions

### Filter Events

`lt({<threshold>|<sampleIdentifier>})`

:   Represents maximal intervals where the value component is less than the threshold (which may refer to a sample).

`gt({<threshold>|<sampleIdentifier>})`

:   Represents maximal intervals where the value component is greater than the threshold (which may refer to a sample).

`around(abs, {<reference>|<sampleIdentifier>}, {<deviation>|<sampleIdentifier>})`

:   Represents maximal intervals where the absolute difference between the value component and the reference value (which may refer to a sample) is less than the maximum deviation (which may refer to a sample).

`around(rel, {<reference>|<sampleIdentifier>}, {<deviation>|<sampleIdentifier>})`

:   Represents maximal intervals where if the relative (percentage-wise) difference between the value component and the reference value (which may refer to a sample) is less than the maximum deviation (which may refer to a sample). The reference value must not be equal to zero.

`before(<threshold>)`

:   Represents maximal intervals where  the time component is before the temporal threshold.

`after(<threshold>)`

:   Represents maximal intervals where  the time component is after the temporal threshold.

### Complex Events

`const({<slope>|<sampleIdentifier>}, {<deviation>|<sampleIdentifier>})`

: Detects maximal intervals with (approximately) constant values.
  This means the absolute value of the slope of the regression line in such an interval is not greater than the specified slope (which may refer to a sample) in percent (must not be less than 0).
  Furthermore, the values of the interval may not deviate more than the specified deviation (which may refer to a sample) in percent from the interval's average (must not be less than 0).

`increase({<minChange>|<sampleIdentifier>}, {<maxChange>|<sampleIdentifier>}, {<tolerance>|<sampleIdentifier>})`

:  Detects maximal intervals depicting (approximately) monotonic increases.
   The change in value captured by the interval must be non-negative and between the specified minimum and maximum change (which both may refer to a sample and must not be less than 0; "infinity" upper bound can be expressed using `-`).
   Temporary decreases in value are tolerated as long as their respective instantaneous rate of change does not fall below `-t` percent, where `t` represents the specified tolerance (which may refer to a sample).

`decrease({<minChange>|<sampleIdentifier>}, {<maxChange>|<sampleIdentifier>}, {<tolerance>|<sampleIdentifier>})`

:  Detects maximal intervals depicting (approximately) monotonic decreases.
   The change in value captured by the interval must be non-positive and between the specified minimum and maximum change (which both may refer to a sample and must not be less than 0; "infinity" upper bound can be expressed using `-`).
   Temporary increases in value are tolerated as long as their respective instantaneous rate of change does not exceed `+t` percent, where `t` represents the specified tolerance (which may refer to a sample).

## Examples

??? example "Example: Events Component (1)"
    The events component below should detect two kinds of incidents:

    1. Periods in which the recorded value is either above 300 or below 100 for at least 30 minutes.
    2. Periods of rapid, steep monotonic increases of at least 50 % in the space of 45 seconds with a tolerance against decreases of 5.75 %.
       Also, such periods should not appear before January 10th 2022.
    
    ```dtsql
    USING EVENTS:
      OR(gt(300), lt(100)) FOR [30,] minutes AS e1,
      AND(
           increase(50, -, 5.75),
           NOT(before("2022-01-10T00:00:00Z"))
         ) FOR [,45] seconds AS e2
    ```

??? example "Example: Events Component (2)"
    The events component below should detect two kinds of incidents:

    1. Periods in which the value is above the global average.
    2. Periods in which the value is below or equal to the global average.

    ```dtsql
    WITH SAMPLES:
      avg() AS avgGlobal
    USING EVENTS:
      AND(gt(avgGlobal)) AS e1,
      AND(NOT(gt(avgGlobal))) AS e2
    ```
