# TDM — Test Evidence

Verified test results for [TDM — Tytech Download Manager](https://github.com/tahiryounas604/tdm): the free, open-source, resumable download manager (desktop CLI, Android/iOS app, browser extension).

Every result in this repo was produced by actually running the tests — no claims without logs. Raw terminal output lives in `logs/`.

## Results (verified 2026-11-11)

| Suite | Result |
|---|---|
| Desktop CLI unit + integration tests (pytest) | **31/31 passed** (6.55s) |
| Mobile engine tests (Dart) | **41/41 passed** (7s) |
| Browser extension native-host protocol + e2e | **Passed** (byte-identical file) |
| Live download — CacheFly 10 MB, 8 connections | **Done in 5.04s**, avg 1.98 MB/s |
| Live download — NumPy wheel 15.7 MB, 8 connections | **Done in 7.96s**, SHA-256 OK |
| Live pause → resume — OpenCV wheel 61.2 MB | **Resumed in 5.48s**, SHA-256 OK |

## What's in here

- `TEST-REPORT.md` — the full report: what was tested, how, and the honest limits.
- `logs/` — raw terminal output for every run above.

## Reproduce it

The test suites live in the main repo ([tahiryounas604/tdm](https://github.com/tahiryounas604/tdm)):

```bash
git clone https://github.com/tahiryounas604/tdm.git
cd tdm
pip install -r requirements.txt
python3 -m pytest tests/ -q                     # CLI tests (31)
python3 extension/tests/test_native_host.py      # extension native host
```

Mobile engine tests run from `mobile/` with the Dart SDK (`dart test`, 41 tests).

## Honest limits

- Mobile UI never compiled here (no Flutter SDK in this environment) — engine only.
- Extension never tested inside a real browser yet — native-host protocol only.
- No GUI yet (v0.2), no video/torrent yet (v0.4).

## License

GPL-3.0 — Copyright (c) 2026 Tahir Younas / Tytechlabs.
