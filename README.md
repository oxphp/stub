# OxPHP Stubs

Function stubs for the [OxPHP](https://github.com/oxphp/oxphp) PHP extension
(`oxphp_sapi`). Enables IDE autocompletion and static analysis for the
`oxphp_*()` functions that the extension exposes at runtime.

This package contains **declarations only** — no executable code. The actual
implementations live inside the `oxphp_sapi` extension that ships with the
OxPHP server binary.

## Install

```bash
composer require --dev oxphp/stubs
```

## Usage

### PHPStan

If you use [`phpstan/extension-installer`](https://github.com/phpstan/extension-installer),
the stubs are registered automatically — nothing else to configure.

Otherwise, add the stubs manually to your `phpstan.neon`:

```neon
parameters:
    scanFiles:
        - vendor/oxphp/stubs/oxphp.stub.php
```

### Psalm

Add the stub file to your `psalm.xml`:

```xml
<psalm>
    <stubs>
        <file name="vendor/oxphp/stubs/oxphp.stub.php" />
    </stubs>
</psalm>
```

### PhpStorm / VS Code (Intelephense)

Both editors index everything inside `vendor/` automatically, so once the
package is installed you get autocompletion and signature hints for free.

## Why isn't the stub file autoloaded?

The `oxphp_sapi` extension declares the same function names at the C level. If
this package autoloaded `oxphp.stub.php` at runtime under OxPHP, PHP would
fatal with `Cannot redeclare function oxphp_request_id()`. Static analyzers
read the file directly without executing it, so they don't hit that problem.

## Functions

### Request & server

| Function | Purpose |
|----------|---------|
| `oxphp_http_request()` | Read-only `OxPHP\Http\RequestInterface` for the current request |
| `oxphp_superglobals_enabled()` | Whether `$_GET` / `$_POST` / etc. are populated |
| `oxphp_request_id()` | Unique 16-hex-char request ID (mirrors `X-Request-ID`) |
| `oxphp_worker_id()` | Zero-based PHP ZTS worker index |
| `oxphp_server_info()` | SAPI name, version, worker id, request time |
| `oxphp_finish_request()` | Flush response, continue work in background |
| `oxphp_is_worker()` | True when running under `oxphp_worker()` |
| `oxphp_is_streaming()` | True for SSE / chunked-transfer requests |
| `oxphp_request_heartbeat()` | Extend the request timeout from inside long loops |
| `oxphp_stream_flush()` | Activate streaming mode and flush a chunk |
| `oxphp_worker(callable)` | Enter the worker-mode request loop |

### Cooperative scheduling

| Function | Purpose |
|----------|---------|
| `oxphp_sleep(float)` | Cooperative fiber sleep in seconds (falls back to `usleep`) |
| `oxphp_usleep(int)` | Cooperative fiber sleep in microseconds |

### Async tasks

| Function | Purpose |
|----------|---------|
| `oxphp_async(Closure, ...$args)` | Dispatch a closure to the async worker pool |
| `oxphp_async_await(int, ?float)` | Block until a single promise resolves |
| `oxphp_async_await_all(int[], ?float)` | Wait for every promise, return all results |
| `oxphp_async_await_any(int[], ?float)` | Race promises, return the first to complete |

### Decorators

| Function | Purpose |
|----------|---------|
| `oxphp_register_decorator(string)` | Register an `#[Attribute]` class as a decorator |

### APM (OpenTelemetry tracing)

| Function | Purpose |
|----------|---------|
| `oxphp_apm_trace(string, callable, array)` | Run callback inside an auto-closed child span |
| `oxphp_apm_start(string, array)` | Open a manual child span (pair with `oxphp_apm_end()`) |
| `oxphp_apm_end(int)` | Close a manually opened span |
| `oxphp_apm_attribute(string, mixed, ?int)` | Add an attribute to the current (or given) span |
| `oxphp_apm_event(string, array, ?int)` | Add a named timestamp event to a span |
| `oxphp_apm_error(Throwable, ?int)` | Record an exception event and mark the span as error |
| `oxphp_apm_status(int, ?string, ?int)` | Set explicit span status (`OXPHP_APM_OK` / `OXPHP_APM_ERROR`) |
| `oxphp_apm_trace_id()` | Current 32-hex W3C trace ID |
| `oxphp_apm_span_id()` | Current 16-hex active span ID |
| `oxphp_apm_header()` | Build a W3C `traceparent` header value for outbound HTTP |

Constants: `OXPHP_APM_OK`, `OXPHP_APM_ERROR`. Plus the `#[OxPHP\Apm\Trace]` attribute, auto-registered when APM is enabled, that wraps the annotated function/method in a span.

### Profiler (`OxPHP\Profile`)

| Function | Purpose |
|----------|---------|
| `OxPHP\Profile\is_active()` | Whether profiling is actively capturing spans |
| `OxPHP\Profile\start()` | Enable profiling for the rest of the current request |
| `OxPHP\Profile\stop()` | Stop further span capture |
| `OxPHP\Profile\pause()` / `resume()` | Soft pause / resume around hot sections |
| `OxPHP\Profile\mark(string, array)` | Attach a named marker event to the topmost open span |
| `OxPHP\Profile\metric(string, float)` | Append a numeric `metric.<name>` attribute to the current span |

Profiler attributes: `#[Profile]`, `#[Exclude]`, `#[Sample(rate)]`, `#[Tag(key, value)]` (repeatable), `#[Mark(label?)]`, `#[SlowThreshold(ms)]`, `#[MemoryThreshold(kb)]`.

### Shared primitives (`OxPHP\Shared`)

Process-wide concurrent primitives visible from every PHP worker thread.

| Class | Purpose |
|----------|---------|
| `Counter` | Lock-free atomic signed 64-bit counter |
| `Flag` | Atomic boolean flag |
| `Once` | Run-once container with cross-worker initialisation |
| `Mutex` | Poisoning mutex with optional timeout |
| `Channel` | Bounded MPMC queue with fiber-aware blocking send / recv |
| `Map` | Concurrent `string → mixed` key-value store with cycle detection |
| `Pool` | Bounded object pool with idle-timeout eviction |

Exception hierarchy under `OxPHP\Shared\`: `Exception` ← `StaleHandleException`, `TypeException` ← `CycleException`, `CapacityException`, `ClosedException`, `PoisonedException`, `TimeoutException` ← `DeadlockException`, `UninitializedException`. All Shared types implement the `OxPHP\Shared\Shareable` marker interface so they can be nested inside `Map` / `Channel` without serialisation.

### Object API

The stub also declares the namespaced surface that the extension exposes at runtime:

- `OxPHP\Http\{RequestInterface, SessionInterface, UploadedFileInterface, AttributesInterface}` and their final implementations (`Request`, `Session`, `UploadedFile`, `Attributes`).
- `OxPHP\Http\Exception\{NoActiveRequestException, AsyncContextException, WorkerIdleException}` — thrown by `oxphp_http_request()` outside a live request context.
- `OxPHP\Async\{Exception, TimeoutException, BorrowException, BorrowedProxy}` — thrown by the `oxphp_async_*()` family. `BorrowedProxy` is the opaque stand-in injected when a `use`-captured value is still borrowed by the source thread; every access throws `BorrowException`.
- `OxPHP\Decorator\{AttributeInterface, Context, RejectedException}` — implement `AttributeInterface` and register with `oxphp_register_decorator()` to intercept calls.
- `OxPHP\Apm\Trace` — automatic span attribute (auto-registered when APM is enabled).
- `OxPHP\Profile\*` — profiler functions and attributes (see above).
- `OxPHP\Shared\*` — process-wide concurrent primitives (see above).

See [`oxphp.stub.php`](oxphp.stub.php) for full PHPDoc with parameter types,
return types, and usage examples.

## License

MIT — see [LICENSE](LICENSE). The OxPHP server itself is licensed under
AGPL-3.0-or-later; the stubs are kept permissive so they can be safely added
to any project as a dev dependency.
