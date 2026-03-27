# Telemetry Data Flow

## What Problem Does This Solve?

Mission control applications need to display real-time and historical telemetry data from multiple sources (spacecraft, ground systems, simulations). The Telemetry API provides a unified interface for requesting, subscribing to, and formatting telemetry data regardless of the underlying data source.

## Available Operations

### Historical Data Requests

- **Time-bounded queries**: Request data for a specific time range
- **Batch retrieval**: Fetch multiple telemetry points efficiently
- **Latest value**: Get most recent data point

### Real-time Subscriptions

- **Subscribe**: Receive streaming data as it arrives
- **Unsubscribe**: Stop receiving updates
- **Batch subscriptions**: Subscribe to multiple points simultaneously

### Data Formatting

- **Value formatting**: Format raw telemetry values for display (units, precision)
- **Metadata**: Access telemetry metadata (ranges, units, data types)
- **Limits**: Apply warning/alarm thresholds

## Design Choices

### Provider Pattern

Like the Object API, the Telemetry API uses providers to abstract different data sources:

- **Historical providers**: Serve time-series data from databases
- **Real-time providers**: Stream live data from spacecraft or simulations
- **Formatters**: Transform raw values for display

### Asynchronous Requests

All telemetry requests are asynchronous to handle:
- Network latency for remote data sources
- Large datasets that take time to query
- Streaming data that arrives over time

### Metadata-Driven Formatting

Telemetry objects define their own metadata (units, format, ranges) so the same formatter can handle different types of data without custom code.

## Common Workflow Patterns

### Displaying Historical Plot Data

1. User navigates to a plot view of a telemetry point
2. Plot requests historical data for the current time range
3. Telemetry API queries appropriate provider
4. Provider fetches data from database/cache
5. Data is formatted for display
6. Plot renders the time series

### Real-time Telemetry Streaming

1. User enables real-time mode on a telemetry view
2. View subscribes to telemetry point
3. Provider establishes connection to data source
4. New data arrives and is formatted
5. View updates display with new values
6. On view close, subscription is cancelled

### Combined Historical + Real-time

1. Plot loads historical data for time range
2. User switches to real-time mode
3. Plot maintains historical data in view
4. Subscription adds new points as they arrive
5. Time window slides forward, old data dropped

## Error Scenarios and Recovery

### Request Failures

- **Provider Unavailable**: Telemetry source is offline
  - Recovery: Display error message, retry with backoff, use cached data if available

- **Invalid Time Range**: Requested range is malformed or too large
  - Recovery: Validate inputs, limit query size, paginate results

- **No Data**: Valid request but no data exists for range
  - Recovery: Display "no data" message, suggest different time range

### Subscription Failures

- **Connection Lost**: Real-time connection drops
  - Recovery: Auto-reconnect with exponential backoff, notify user of stale data

- **Data Gap**: Missing packets in stream
  - Recovery: Mark gaps in visualization, attempt backfill from historical provider

### Performance Issues

- **Large Dataset**: Query returns too much data
  - Recovery: Implement decimation, downsample for display, paginate requests

- **High Frequency Updates**: Subscription overwhelms browser
  - Recovery: Throttle updates, batch multiple points, skip intermediate values
