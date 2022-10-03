# CSV Storage Service

This `StorageService` implementation is responsible for accessing data stored in an InfluxDB database.
Its behaviour is in accordance with the interface contract.
The supported parameterization, i.e., the configuration properties, are defined below.

!!! tip
    The tables may be sorted by clicking on the respective column header.

## Initialize Configuration Properties

| Identifier     | Type        |               Optional?                | Constraint(s) | Description                                                                         |
|----------------|-------------|:--------------------------------------:|:--------------|:------------------------------------------------------------------------------------|
| `url`          | String      | :fontawesome-solid-xmark:{ .negative } |               | The URL the InfluxDB from which a time series should be extracted is accessible at. |
| `token`        | Character[] | :fontawesome-solid-xmark:{ .negative } |               | The access token required to establish a connection to the InfluxDB instance.       |
| `organization` | String      | :fontawesome-solid-xmark:{ .negative } |               | The InfluxDB organization containing the time series to extract.                    |
| `bucket`       | String      | :fontawesome-solid-check:{ .positive } |               | The bucket of the InfluxDB instance containing the time series to extract.          |

## Load Configuration Properties

| Identifier  | Type                |               Optional?                | Constraint(s)                                                                           | Description                                                                                                                     |
|-------------|---------------------|:--------------------------------------:|:----------------------------------------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------|
| `query`     | String              | :fontawesome-solid-check:{ .positive } | Either only `query` or (exclusively) `bucket`, `loadFrom`, `loadUntil` must be present. | If present, the InfluxDB query which is used to extract the time series from the database instance.                             |
| `bucket`    | Character           | :fontawesome-solid-check:{ .positive } | Either only `query` or (exclusively) `bucket`, `loadFrom`, `loadUntil` must be present. | If present, the name of the InfluxDB bucket containing the time series to extract.                                              |
| `loadFrom`  | Instant (Date/Time) | :fontawesome-solid-check:{ .positive } | Either only `query` or (exclusively) `bucket`, `loadFrom`, `loadUntil` must be present. | If present, the temporal lower bound from which data points contained in the bucket specified by `bucket` should be extracted.  |
| `loadUntil` | Instant (Date(Time) | :fontawesome-solid-check:{ .positive } | Either only `query` or (exclusively) `bucket`, `loadFrom`, `loadUntil` must be present. | If present, the temporal upper bound until which data points contained in the bucket specified by `bucket` should be extracted. |

## Transform Configuration Properties

| Identifier   | Type    |               Optional?                | Constraint(s)                                                                                               | Description                                                                                                                                                                                                                                                                                                                  |
|--------------|---------|:--------------------------------------:|:------------------------------------------------------------------------------------------------------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `tableIndex` | Integer | :fontawesome-solid-xmark:{ .negative } | Must be equal to or greater than `-1` (but less than the number of FluxTable instances returned by `load`). | The index of the FluxTable returned by `load` whose data points should be used to construct the canonical time series. If the value of equal to `-1`, the data points from all tables are used. Otherwise, i.e., for values equal to or greater than `0`, only the data points from the table with this very index are used. |

## Store Configuration Properties

!!! bug
    The `store` operation has not been implemented yet.

| Identifier         | Type      |               Optional?                | Constraint(s)                                              | Description                                                                                                                                                               |
|--------------------|-----------|:--------------------------------------:|:-----------------------------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
