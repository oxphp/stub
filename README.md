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

| Function | Purpose |
|----------|---------|
| `oxphp_request_id()` | Unique 16-hex-char request ID (mirrors `X-Request-ID`) |
| `oxphp_worker_id()` | Zero-based PHP ZTS worker index |
| `oxphp_server_info()` | SAPI name, version, worker id, request time |
| `oxphp_finish_request()` | Flush response, continue work in background |
| `oxphp_is_worker()` | True when running under `oxphp_worker()` |
| `oxphp_is_streaming()` | True for SSE / chunked-transfer requests |
| `oxphp_request_heartbeat()` | Extend the request timeout from inside long loops |
| `oxphp_stream_flush()` | Activate streaming mode and flush a chunk |
| `oxphp_worker(callable)` | Enter the worker-mode request loop |

See [`oxphp.stub.php`](oxphp.stub.php) for full PHPDoc with parameter types,
return types, and usage examples.

## License

MIT — see [LICENSE](LICENSE). The OxPHP server itself is licensed under
AGPL-3.0-or-later; the stubs are kept permissive so they can be safely added
to any project as a dev dependency.
