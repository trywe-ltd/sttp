# zio-telemetry opentracing backend 

To use, add the following dependency to your project:

```
"com.softwaremill.sttp.client" %% "zio-telemetry-opentracing-backend" % "2.1.1"
```

This backend depends on [zio-opentracing](https://github.com/zio/zio-telemetry).

The opentracing backend wraps a `Task` based ZIO backend and yields a backend of type `SttpBackend[RIO[OpenTracing, *], Nothing, WS_HANDLER]`. The yielded effects are of type `RIO[OpenTracing, *]` which mean they can be a child of a other span created in your ZIO program.

Here's how you construct `ZioTelemetryOpenTracingBackend`. I would recommend wrapping this is in `ZLayer`

```scala
new ZioTelemetryOpenTracingBackend(zioBackend)
```

Additionally you can add tags per request by supplying a `ZioTelemetryOpenTracingTracer`

```scala
def sttpTracer: ZioTelemetryOpenTracingTracer = new ZioTelemetryOpenTracingTracer {
    def before[T](request: Request[T, Nothing]): RIO[OpenTracing, Unit] =
      OpenTracing.tag("span.kind", "client") *>
      OpenTracing.tag("http.method", request.method.method) *>
      OpenTracing.tag("http.url", request.uri.toString()) *>
      OpenTracing.tag("type", "ext") *>
      OpenTracing.tag("subtype", "http")
    
    def after[T](response: Response[T]): RIO[OpenTracing, Unit] =
      OpenTracing.tag("http.status_code", response.code.code)
}

new ZioTelemetryOpenTracingBackend(zioBackend, sttpTracer)
```


