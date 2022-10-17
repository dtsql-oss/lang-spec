# CSV Storage Service

This `StorageService` implementation is responsible for accessing (locally stored) CSV files.
Its behaviour is in accordance with the interface contract.
The supported parameterization, i.e., the configuration properties, are defined below.

!!! tip
    The tables may be sorted by clicking on the respective column header.

## Initialize Configuration Properties

At the moment, this service does not have a special initialization routine.

| Identifier               | Type      |               Optional?                | Constraint(s)            | Description                                                                                                                                                                                                                                                                    |
|--------------------------|-----------|:--------------------------------------:|:-------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|


## Load Configuration Properties

| Identifier               | Type      |               Optional?                | Constraint(s)              | Description                                                                                                                                                                                                                                                                    |
|--------------------------|-----------|:--------------------------------------:|:---------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `filePath`               | String    | :fontawesome-solid-xmark:{ .negative } |                            | The path of the local file to read the time series from.                                                                                                                                                                                                                       |
| `fieldSeparator`         | Character | :fontawesome-solid-xmark:{ .negative } |                            | The character that separates fields (columns) in the file.                                                                                                                                                                                                                     |
| `skipHeaders`            | Integer   | :fontawesome-solid-xmark:{ .negative } | Must not be less than `0`. | The number of rows to skip before the data to actually load starts.                                                                                                                                                                                                            |
| `customEndOfFileMarkers` | String[]  | :fontawesome-solid-check:{ .positive } |                            | If present, instructs the CSV parser to abort if a line whose content is equivalent to one of the custom EOF markers is encountered. This is useful for when you know a CSV file contains additional information beyond a certain point which is not relevant in that context. |


## Transform Configuration Properties

| Identifier    | Type    |               Optional?                | Constraint(s)                                                              | Description                                                                                           |
|---------------|---------|:--------------------------------------:|:---------------------------------------------------------------------------|:------------------------------------------------------------------------------------------------------|
| `valueColumn` | Integer | :fontawesome-solid-xmark:{ .negative } | Must be less than the number of columns in the file and not less than `0`. | The index of the column which contains the value component of data points in the time series to load. |
| `timeColumn`  | Integer | :fontawesome-solid-xmark:{ .negative } | Must be less than the number of columns in the file and not less than `0`. | The index of the column which contains the time component of data points in the time series to load.  |
| `timeFormat`  | String  | :fontawesome-solid-xmark:{ .negative } | Must be a valid (Java) date-time pattern.                                  | The date-time pattern (format) to be used when parsing the time component of a data point.            |

## Store Configuration Properties

| Identifier         | Type      |               Optional?                | Constraint(s)                                              | Description                                                                                                                                                               |
|--------------------|-----------|:--------------------------------------:|:-----------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `filePath`         | String    | :fontawesome-solid-xmark:{ .negative } |                                                            | The path of the file the time series should be serialized to.                                                                                                             |
| `fieldSeparator`   | Character | :fontawesome-solid-xmark:{ .negative } |                                                            | The character to use to separate fields (columns) in the file.                                                                                                            |
| `timeFormat`       | String    | :fontawesome-solid-xmark:{ .negative } | Must be a valid (Java) date-time pattern.                  | The date-time pattern (format) to use when serializing the time component of a data point.                                                                                |
| `append`           | Boolean   | :fontawesome-solid-xmark:{ .negative } |                                                            | If `true`, the serialized time series is appended to the file specified by `filePath`. Otherwise, it (re-)creates the file, potentially truncating pre-existing contents. |
| `includeHeaders`   | Boolean   | :fontawesome-solid-xmark:{ .negative } |                                                            | Specifies whether CSV headers should be included during time series serialization.                                                                                        |
| `timeColumnLabel`  | String    | :fontawesome-solid-check:{ .positive } | If `includeHeaders` is `true`, this property is mandatory. | The header of the column containing time components of the data points of the serialized time series.                                                                     |
| `valueColumnLabel` | String    | :fontawesome-solid-check:{ .positive } | If `includeHeaders` is `true`, this property is mandatory. | The header of the column containing value components of the data points of the serialized time series.                                                                    |
