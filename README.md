# Tinybird <> Datadog integration

This data project demonstrates how users can integrate [Tinybird](https://tinybird.co) with [Datadog](https://datadog.com/) bulding endpoints on top of the [Service Data Sources](Service Data Sources) and using [vector.dev](https://vector.dev).

The project is defined by:

- **Data project**: The Tinybird data project
- **Vector.dev files**: Files describing vector.dev workflows
- **GitHub Actions files**: Files describing the GitHub Action workflows used to call Tinybird endpoints, and execute vector.dev flows

## Push data project to Tinybird

The data project is defined in the `/data-project` folder. To push teh project to Tibynird, you just have to install and configure the Tinybird CLI following [these instructions](https://docs.tinybird.co/cli.html), and then execute

```bash
cd data-project
tb push
```

This will create two endpoints:

- `endpoints/ep_datadog_ops_log.pipe`: Used to extract datasources operations metrics from `tinybird.datasources_ops_log`
- `endpoints/ep_datadog_pipes_stats.pipe`: Used to extract endpoints metrics from `tinybird.pipe_stats_rt`

And a datasource:

- `datasources/datadog_integration.datasource`: Used to keep track of the last execution to avoid duplicating data in Datadog

The process will also create a token named `datadog_integration_token` with read access to the endpoints and append access to the datasource.

## Configure GitHub Actions

> :warning: Note: This repository illustrates how to do the integration using GitHub Action but you can use the job scheduler best fits your current architecture. 

To configure the GitHub Actions, you'll have to create the following environment variables:

- `TB_TOKEN`: Tinybird token named `datadog_integration_token`
- `DATADOG_API_KEY`: Datadog API Key
- `DATADOG_REGION`: Datadog region

## How everything works?

The process basically uses vector.dev to read data from Tinybird API endpoints, makes basic transformations to generate metrics, and sends data to Datadog. The jobs are scheduled to run every 10 minutes, running the following commands:

```bash
curl "https://api.us-east.tinybird.co/v0/pipes/ep_datadog_pipes_stats.ndjson?token=${TB_TOKEN}" | ~/.vector/bin/vector --config ./vector-pipes-stats.toml
curl "https://api.us-east.tinybird.co/v0/pipes/ep_datadog_ops_log.ndjson?token=${TB_TOKEN}" | ~/.vector/bin/vector --config ./vector-ops-log.toml
```

## Metrics

| metric_name             | metric_type | interval | unit_name  |
|-------------------------|-------------|----------|------------|
| tb.pipes.count          | count       | 1min     | requests   |
| tb.pipes.duration       | gauge       | 1min     | seconds    |
| tb.pipes.duration_p99   | gauge       | 1min     | seconds    |
| tb.pipes.read_bytes     | count       | 1min     | bytes      |
| tb.datasources.count    | count       | 1min     | operations |
| tb.datasources.rows     | count       | 1min     | rows       |
| tb.datasources.duration | gauge       | 1min     | seconds    |
