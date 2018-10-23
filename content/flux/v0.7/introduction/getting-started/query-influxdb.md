---
title: Query InfluxDB with Flux
description: Learn the basics of using Flux to query data from InfluxDB.
menu:
  flux_0_7:
    name: Query InfluxDB
    parent: get-started
    weight: 1
---

This guide walks through the basics of using Flux to query data from InfluxDB.
If you haven't already, make sure to install both InfluxDB v1.7+ and Chronograf v1.7+.

The easiest way to write your first Flux script and see actual results is to use **Chronograf's Data Explorer**.
Open the Data Explorer and click **Add a Query**.
To the right of the source dropdown above the graph placeholder, select **Flux** as the source type.

![Flux in the Data Explorer](/img/flux/flux-builder-start.gif)

This will modify the user interface to provide a **Script** pane and an **Explore** pane.
The Script pane is where you write your Flux script.
The Explore pane allows you to explore your data.

Every Flux query needs the following:

1. [A data source](#1-define-your-data-source)
2. [A time range](#2-specify-a-time-range)
3. [Data filters](#3-filter-your-data)


## 1. Define your data source
Flux's [`from()`](#) function defines an InfluxDB data source.
It requires a [`bucket`](/flux/v0.7/introduction/getting-started/#buckets) property.
For this example, use `telegraf/autogen`, a combination of the default database and retention policy provided by the TICK stack.

```js
from(bucket:"telegraf/autogen")
```

## 2. Specify a time range
Flux requires a time range when querying time series data.
"Unbounded" queries are very resource-intensive and as a protective measure,
Flux will not query the database without a specified range.

Use pipe forward operator (`|>`)to pipe data from your data source into the [`range()`](#)
function, which specifies a time range for your query.
It accepts two properties: `start` and `stop`.
Ranges can be **relative** using negative [durations](#) or **absolute** using [timestamps](#).


###### Example relative time ranges
```js
// Relative time range with start only. Stop defaults to now.
from(bucket:"telegraf/autogen")
  |> range(start: -1h)

// Relative time range with start and stop
from(bucket:"telegraf/autogen")
  |> range(start: -1h, stop: -10m)
```

> Relative ranges are relative to "now."

###### Example absolute time range
```js
from(bucket:"telegraf/autogen")
  |> range(start: 2018-11-05T23:30:00Z, stop: 2018-11-06T00:00:00Z)
```

#### Use the following:
For this guide, use the relative time range, `-15m`, to limit query results to data from the last 15 minutes:

```js
from(bucket:"telegraf/autogen")
  |> range(start: -15m)
```

## 3. Filter your data
Pass your ranged data into the `filter()` function to narrow results based on
[data attributes](/flux/v0.7/introduction/getting-started/#flat-schemas).
The `filter()` function has one property, `fn`, which expects an anonymous function
with logic that filters data based on columns or attributes.

Flux's anonymous function syntax is very similar to Javascript's.
Data tables are passed into the `filter()` function as an object.
The anonymous function takes the object and evaluates each line to see if it matches the defined filters.
Use the `AND` relational operator to chain multiple filters.

```js
// Pattern
(objectName) => (objectName.objectProperty comparisonOperator comparisonExpression)

// Example with single filter
(r) => (r._measurement == "cpu")

// Example with multiple filters
(r) => (r._measurement == "cpu") AND (r._field != "usage_system" )
```

#### Use the following:
For this example, filter by the `cpu` measurement, the `usage_system` field, and the `cpu-total` tag value:

```js
from(bucket:"telegraf/autogen")
  |> range(start: -15m)
  |> filter(fn: (r) =>
    r._measurement == "cpu" AND
    r._field == "usage_system" AND
    r.cpu == "cpu-total")
```

## 4. Yield your queried data
Use Flux's `yield()` function to output the filtered tables as the result of the query.

```js
from(bucket:"telegraf/autogen")
  |> range(start: -15m)
  |> filter(fn: (r) =>
    r._measurement == "cpu" AND
    r._field == "usage_system" AND
    r.cpu == "cpu-total")
  |> yield()
```

> Chronograf automatically assumes a `yield()` function at the end of each script in order to visualized the data.

## Congratulations!
You have now queried data from InfluxDB using Flux.
This is a barebones query that can now be transformed in other ways.

<div class="page-nav-btns">
  <a class="btn prev" href="/flux/v0.7/introduction/getting-started/query-influxdb/">Get started with Flux</a>
  <a class="btn next" href="/flux/v0.7/introduction/getting-started/query-influxdb/">Transform your data</a>
</div>