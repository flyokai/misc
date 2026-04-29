# flyokai/misc

> User docs → [`README.md`](README.md) · Agent quick-ref → [`CLAUDE.md`](CLAUDE.md) · Agent deep dive → [`AGENTS.md`](AGENTS.md)

Utility helpers: CryptKey (OpenSSL), ProfilerFacade (nested timers), PsrServerAdapter (AMPHP ↔ PSR-7), SharedSequence (async ordering).

See [AGENTS.md](AGENTS.md) for detailed documentation.

## Quick Reference

- **CryptKey**: RSA/EC key holder with file permission validation
- **ProfilerFacade**: Static profiling with `start()`/`stop()`, silent when disabled
- **PsrServerAdapter**: Bidirectional AMPHP ↔ PSR-7 request/response conversion
- **SharedSequence**: Monotonic position-based coroutine synchronization for Revolt
