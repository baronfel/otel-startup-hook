# An OpenTelemetry Startup Hook for .NET

This library is a ready-to-use [.NET CLR Startup Hook](https://github.com/dotnet/runtime/blob/52e1ad3779e57c35d2416cd10d8ad7d75b2c0c8b/docs/design/features/host-startup-hook.md) that allows applications that are instrumented with [System.Diagnostic.Activity tracing](https://learn.microsoft.com/en-us/dotnet/core/diagnostics/distributed-tracing-instrumentation-walkthroughs) to
send those traces to an [OpenTelemetry Collector](https://opentelemetry.io/docs/collector/) via an [OpenTelemetry Exporter](https://opentelemetry.io/docs/languages/net/exporters/) without having to add references to the [OpenTelemetry SDK](https://opentelemetry.io/docs/languages/net/) into their applications directly.


## Getting Started

To use this hook in your application, you'll need:

* An application that already has System.Diagnostic.Activity tracing enabled
* An OpenTelemetry Collector running and ready to receive traces (for example [otel-desktop-viewer](https://github.com/CtrlSpice/otel-desktop-viewer))
* This repo cloned locally
* A .NET 8+ SDK installed

### Building the Hook

Publishing this project is all you need. The project and all of its dependencies will be combined into one output assembly (at `./bin/Release/net8.0/publish/otel-startup-hook.dll`)

```terminal
dotnet publish
```

### Using the Hook

Get the absolute path of the freshly-built DLL from the previous step, and set it as the value of the `DOTNET_STARTUP_HOOKS` environment variable.

Bash/Zsh/etc:
```bash
export DOTNET_STARTUP_HOOKS=/path/to/otel-startup-hook.dll
```

Powershell
```powershell
$env:DOTNET_STARTUP_HOOKS = "C:\path\to\otel-startup-hook.dll"
```

Then, run your application as you normally would. The hook will automatically start up and begin sending traces to the OpenTelemetry Collector.


## Configuration

### Configuring what is collected

Currently, only Traces are collected. Each 'source' you want to collect from must be set in the `OTEL_TRACE_SOURCE_NAMES` environment variable. This is a comma-separated list of source names, and these source names are usually defined in your application when you create an ActivitySource:

```csharp
using System.Diagnostics;

public static class Telemetry
{
    //...

    // Name it after the service name for your app.
    // It can come from a config file, constants file, etc.
    public static readonly ActivitySource MyActivitySource = new("myapp");

    //...
}
```

### Configuring where the traces are sent

This hook uses the [OLTP endpoint exporter](https://opentelemetry.io/docs/languages/net/exporters/#otlp-endpoint) to send traces to the OpenTelemetry Collector. This defaults to sending traces over gRPC to `http://localhost:4317`. You can configure how the OTLP exporter behaves by setting various environment variables, as described in the [OLTP Exporter Configuration](https://opentelemetry.io/docs/languages/sdk-configuration/otlp-exporter/) documentation.
