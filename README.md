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

### Object API

The stub also declares the namespaced surface that the extension exposes at runtime:

- `OxPHP\Http\{RequestInterface, SessionInterface, UploadedFileInterface, AttributesInterface}` and their final implementations (`Request`, `Session`, `UploadedFile`, `Attributes`).
- `OxPHP\Http\Exception\{NoActiveRequestException, AsyncContextException, WorkerIdleException}` — thrown by `oxphp_http_request()` outside a live request context.
- `OxPHP\{AsyncException, AsyncTimeoutException, AsyncBorrowException}` — thrown by the `oxphp_async_*()` family.
- `OxPHP\Decorator\{AttributeInterface, Context, RejectedException}` — implement `AttributeInterface` and register with `oxphp_register_decorator()` to intercept calls.

See [`oxphp.stub.php`](oxphp.stub.php) for full PHPDoc with parameter types,
return types, and usage examples.

## License

MIT — see [LICENSE](LICENSE). The OxPHP server itself is licensed under
AGPL-3.0-or-later; the stubs are kept permissive so they can be safely added
to any project as a dev dependency.
