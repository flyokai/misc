# flyokai/misc

> User docs → [`README.md`](README.md) · Agent quick-ref → [`CLAUDE.md`](CLAUDE.md) · Agent deep dive → [`AGENTS.md`](AGENTS.md)

> A grab-bag of small utilities that don't deserve their own packages: cryptographic key handling, profiling, PSR-7 ↔ AMPHP bridging, and async sequencing.

These helpers exist because they're shared between two or more flyokai packages and are too small to release individually.

## Features

- **`CryptKey`** — secure OpenSSL key holder (RSA, EC) with file-permission validation, derived from `league/oauth2-server`
- **`ProfilerFacade` / `ProfilerStat`** — Magento-style nested profiler with memory tracking
- **`PsrServerAdapter`** — bidirectional AMPHP HTTP ↔ PSR-7 converter
- **`SharedSequence`** — monotonic position-based fiber synchronisation primitive

## Installation

```bash
composer require flyokai/misc
```

## CryptKey

```php
use Flyokai\Misc\CryptKey;

$key = new CryptKey('/path/to/private.pem', passPhrase: null, keyPermissionsCheck: true);

$key->getKeyContents();   // PEM contents (string)
$key->getKeyPath();
$key->getPassPhrase();
```

- Accepts a path with or without `file://` prefix, **or** raw PEM material
- Validates RSA / EC types via OpenSSL on construction
- Refuses files whose mode isn't one of `400`, `440`, `600`, `640`, `660` (configurable via `$keyPermissionsCheck`)

## ProfilerFacade

A static profiling facade with nested timers and memory metrics:

```php
use Flyokai\Misc\ProfilerFacade;

ProfilerFacade::setEnabled(true);
ProfilerFacade::start('request');
ProfilerFacade::start('request->db->query');
// … work …
ProfilerFacade::stop('request->db->query');
ProfilerFacade::stop('request');

foreach (ProfilerFacade::getAllStat() as $line) {
    echo $line, "\n";
    // request: 12.34 ms, count=1, avg=12.34 ms, real=512 KB, emalloc=200 KB
}
```

When disabled (the production default), every call is a silent no-op. Hierarchy is expressed through `->` separators in timer IDs.

`ProfilerStat::getFilteredTimerIds($thresholds, $filterRegex)` returns timers above given thresholds, optionally filtered by regex.

## PsrServerAdapter

Used by the OAuth server to bridge League OAuth2 (PSR-7) with the AMPHP HTTP server:

```php
use Flyokai\Misc\PsrServerAdapter;

$adapter = new PsrServerAdapter();

$psrRequest  = $adapter->toPsrServerRequest($ampRequest);
$ampResponse = $adapter->fromPsrServerResponse($psrResponse, $ampRequest);
```

| Method | Direction |
|--------|-----------|
| `toPsrServerRequest(Request)` | AMPHP → PSR-7 |
| `fromPsrServerRequest(PsrServerRequest, Client)` | PSR-7 → AMPHP |
| `toPsrServerResponse(Response)` | AMPHP → PSR-7 |
| `fromPsrServerResponse(PsrResponse, Request)` | PSR-7 → AMPHP |

Headers, cookies, query params, attributes and the body are mapped. Pass `withBody: false` to drop the body.

## SharedSequence

A coroutine-ordering primitive — fibers wait until a monotonic position is reached:

```php
use Flyokai\Misc\SharedSequence;

$seq = new SharedSequence();

// Fiber A
$seq->await(5);   // suspends until position >= 5

// Fiber B
$seq->resume(5);  // advances to position 6, resumes any waiters whose target is <= 5
```

- `await(int $position): void` — suspends the current fiber until `$position` is reached
- `resume(int $position): void` — sets position to `max($position, $current) + 1` and resumes all matching waiters

Position only ever moves forward — once advanced past a value, you can't await it again.

## Gotchas

- **`CryptKey` may try to read your raw key as a path** — if the literal happens to be a valid file path, OpenSSL will load that file instead. Inspect input.
- **`ProfilerFacade` is silent when disabled** — `getAllStat()` returns an empty array. There's no warning that profiling is off.
- **`PsrServerAdapter` buffers the body** in `toPsrServerRequest()` — inefficient for very large requests.
- **`SharedSequence` is strictly monotonic** — once advanced, earlier positions can't be awaited.

## See also

- [`flyokai/oauth-server`](../oauth-server/README.md) uses `CryptKey` and `PsrServerAdapter`.
- [`flyokai/symfony-console`](../symfony-console/README.md) ships a `CryptKeyValidator` built on `CryptKey`.

## License

MIT
