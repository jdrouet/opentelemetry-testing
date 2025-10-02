# OpenTelemetry Testing

This project provides a testing framework for OpenTelemetry instrumentation, enabling you to validate and debug your telemetry setup. It includes utilities for setting up an OpenTelemetry Collector, exporting traces, and verifying telemetry data.

## Features

- **OpenTelemetry Integration**: Provides a builder for configuring OpenTelemetry providers for metrics, traces, and logs.
- **Testcontainers Support**: Uses `testcontainers` to spin up an OpenTelemetry Collector for testing.
- **Trace Validation**: Includes utilities to parse and validate traces exported to a file.
- **Configurable**: Easily customize the OpenTelemetry Collector configuration.

## Getting Started

### Prerequisites

- Rust (edition 2024)
- Docker (required for `testcontainers`)

### Installation

Add this crate to your `Cargo.toml`:

```toml
[dependencies]
opentelemetry-testing = "0.1.0"
```

### Usage

#### Setting Up OpenTelemetry

Use the `OpenTelemetryBuilder` to configure and install OpenTelemetry providers:

```rust,skip
let builder = OpenTelemetryBuilder {
    otel_collector_endpoint: "http://127.0.0.1:4317".into(),
    otel_internal_level: "off".into(),
};
let provider = builder.build().unwrap();
provider.install().unwrap();
```

#### Running Tests with Testcontainers

The `ObservabilityContainer` struct simplifies setting up an OpenTelemetry Collector for testing:

```rust,skip
#[tokio::test]
async fn test_traces() {
    let container = ObservabilityContainer::create().await;
    let provider = container.install().await;

    // Your test logic here

    provider.flush();
    let traces = container.json_traces();
    assert!(traces.resource_spans.len() > 0);
}
```

#### Validating Traces

You can parse and validate traces exported to a file:

```rust,skip
let traces = container.json_traces();
let scope_span = traces.find_scope_span("my-instrumentation");
assert!(scope_span.is_some());
```

### OpenTelemetry Collector Configuration

The OpenTelemetry Collector is configured using the `otelcol-config.yml` file. By default, it exports traces to a JSON file at `/tmp/output/traces.json`.

Example configuration:

```yaml
receivers:
  otlp:
    protocols:
      grpc:
        endpoint: 0.0.0.0:4317
exporters:
  file/traces:
    path: /tmp/output/traces.json
    format: json
service:
  pipelines:
    traces:
      receivers: [otlp]
      exporters: [file/traces]
```

## Development

### Running Tests

Run the tests using `cargo`:

```bash
cargo test
```

### Formatting

Ensure your code is formatted:

```bash
cargo fmt
```

### Linting

Check for common issues:

```bash
cargo clippy
```

## License

This project is licensed under the MIT License. See the [LICENSE](LICENSE) file for details.

## Contributing

Contributions are welcome! Please open an issue or submit a pull request.

## Acknowledgments

- [OpenTelemetry](https://opentelemetry.io/)
- [Testcontainers](https://www.testcontainers.org/)
