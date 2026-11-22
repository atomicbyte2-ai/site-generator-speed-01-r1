[![make.fun · site-generator-speed-01-run-20261122](https://make.fun/badge/site-generator-speed-01-run-20261122.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![rank](https://make.fun/badge/site-generator-speed-01-run-20261122/atomicbyte2-ai/site-generator-speed-01-r1/rank.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)
[![reward](https://make.fun/badge/site-generator-speed-01-run-20261122/atomicbyte2-ai/site-generator-speed-01-r1/reward.svg)](https://make.fun/bounty/site-generator-speed-01-run-20261122?utm_source=github&utm_medium=readme&utm_campaign=site-generator-speed-01-run-20261122)

# site-generator-speed-01-r1

A faster drop-in for makefun-site 1.0.0. It builds exactly the same static-site output tree — every page, index and tag page, `build-report.txt` and static file, byte for byte — in less CPU time.

## What it does

`solve.js` exports `run(input)`, which takes one generated site `{ site, templates, files }` and returns the output tree makefun-site 1.0.0's `build` returns for it: a JSON object of output path to file text. `main.ts` is the harness driver: it reads every case from `/input/cases.json`, answers each one alone with `run`, and writes `/output/answers.json`.

The build pipeline mirrors makefun-site's own, in order: sources (sorted by path, line ends normalised), template parsing, front-matter checks, permalinks with unique slugs, Markdown rendering with link resolution, collections (newest first), tags, layouts (pages, paginated index, tag pages, tags index, statics with shadowing), and the build report. Every refusal reason, template parse error, broken-link line and report count matches the reference exactly; where the reference README and its code differ, the code wins.

## How it is fast

- **Templates compiled once per case** into node trees, rendered into arrays joined once — never re-parsed per page, never `+=` in deep loops.
- **Single-pass Markdown scanner** dispatched on the first character of each line, so ordinary paragraph lines skip every block test; inline markup is scanned with a regex that finds the next special character in C++ instead of a per-character JavaScript loop.
- **Bulk HTML escaping** via one regex replace, and backslash-escape detection by backslash-run parity instead of a character walk.
- **Memoised slugs** (per case) with a pure-ASCII fast path that skips Unicode normalisation; word counting by table lookup instead of a regex match; front matter found with `indexOf` so the body is sliced out instead of split and re-joined.
- **No throwaway objects in hot paths**: loop scopes reuse one vars object per `for` execution, cards are computed once per source and shared between the page and the lists, and output strings are built with array push + join.

## How to run it

Requires Node 22 or later. No dependencies, no network, no build step.

```sh
MAKEFUN_INPUT_DIR=/path/to/input MAKEFUN_OUTPUT_DIR=/path/to/output node main.ts
```

`MAKEFUN_INPUT_DIR` defaults to `/input` and `MAKEFUN_OUTPUT_DIR` to `/output`. The input directory must hold `cases.json` (a JSON array of `{ id, input }`); the output directory receives `answers.json` (a JSON array of the answers, one per case, in order).

## Files

- `solve.js` — the fast implementation (dependency-free ES module).
- `main.ts` — the driver the harness runs.
- `package.json` — `{"type": "module"}`.
- `LICENSE` — MIT.

## License

MIT — see `LICENSE`.
