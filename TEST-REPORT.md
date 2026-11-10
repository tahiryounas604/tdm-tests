# TDM — Test Report

Verified 2026-11-11 (live downloads 2026-11-09 and 2026-11-11).
Environment: Linux VM, Python 3, Dart SDK 3.13.7 (standalone), pytest.

## 1. Desktop CLI — 31/31 passed (6.55s)

`python3 -m pytest tests/ -q` — raw output: `logs/pytest.txt`

- `test_chunker` — dynamic chunk splitting (1 MB minimum, 2 s cooldown).
- `test_resume` — atomic `.dmstate` JSON save/load, clean completion removes state.
- `test_speed` — sliding-window speed + ETA.
- `test_engine_e2e` — full download against a local Range-capable HTTP server.

**Live downloads (real servers, real bytes):**

| File | Size | Result |
|---|---|---|
| NumPy wheel (PyPI CDN) | 15,695,312 bytes | 8 connections, 7.96s, SHA-256 OK |
| OpenCV wheel (PyPI CDN) | 61,204,038 bytes | paused at 4.38 MB, resumed in 5.48s, SHA-256 OK |
| CacheFly 10 MB test | 10,485,760 bytes | 8 connections, 5.04s, avg 1.98 MB/s |

**Bugs found and fixed during testing:**

1. Critical — the state file was never updated mid-download (resume would have re-downloaded everything). Fixed in the engine's block callback.
2. httpx crashed on this VM because `NO_PROXY` contained a bracketed IPv6 address (`[::1]`). Sanitised in `connections.py`.

## 2. Mobile engine (Flutter/Dart) — 41/41 passed (7s)

`dart test` in a throwaway harness (engine + tests only, minimal pubspec `crypto` + `test`). Raw output: `logs/dart.txt`

- `chunker_test` (13): even splits, remainder bytes, connection clamping, split math, 2 MB minimum, 2 s cooldown, lagging-pick, JSON round-trip.
- `state_test` (8): save/load round-trip, bad version rejected, incomplete file rejected, missing file error, non-done chunks reset to pending with exact byte counts, atomic write leaves no `.tmp`.
- `speed_test` (9): size/speed/ETA formatting, sliding-window speed, window expiry, ETA math.
- `probe_test` (5): HEAD size+ranges, Range-GET fallback when HEAD is refused, no-range fallback, 404 → `ProbeError`, filename parsing.
- `downloader_test` (6): segmented 5 MB download, byte-exact + SHA-256 OK; **pause mid-download → resume → byte-exact**; single-stream fallback; SHA-256 mismatch fails; probe/engine agreement. All against a real local `HttpServer`.

SHA-256 cross-check: `hashFile()` output identical to system `sha256sum`.

**Bugs found and fixed during testing:**

1. `AccumulatorSink` does not exist in Dart 3.13 — `verify.dart` now uses a tiny custom `Sink<Digest>`.
2. `int.clamp()` returns `num`, not `int` — three assignments fixed with `.toInt()`.
3. **Real bug: `FileMode.write` truncates on open** — the shared output handle was destroying resumed bytes (the pause→resume test caught it). The engine now uses `FileMode.append`.

## 3. Browser extension native host — passed

`python3 extension/tests/test_native_host.py` — raw output: `logs/native-host.txt`

- Protocol test: 4-byte length-prefixed JSON messaging, ping/download/list/status/pause/resume, bad JSON / unknown command / oversize message → `{ok: false}`, never crashes.
- E2E test: 256 KiB local file downloaded through the native host, landed **byte-identical**, database marked completed.

**Bugs found and fixed during testing:**

1. `PROJECT_ROOT` pointed one level too low — the `dm` package import failed.
2. Oversize message desynchronised the stream — fixed with explicit length checks.

## 4. What was NOT tested (honest limits)

- **Mobile UI never compiled** — no Flutter SDK in this environment, and the VM's proxy blocks pub.dev. The engine is verified; the first real Flutter compile happens in GitHub Actions after the code is uploaded.
- **Extension never ran inside a real browser** — no Load-unpacked test yet. The native-host protocol is verified; the floating video button was only shown on a demo page.
- **No GUI yet** (v0.2) and **no video/torrent** (v0.4) — those milestones are not built, so nothing to test there.
- **No real-world daily-use test** — the final proof is running it on a real machine/phone.

## License

GPL-3.0 — Copyright (c) 2026 Tahir Younas / Tytechlabs.
