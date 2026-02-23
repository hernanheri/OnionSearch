# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

OnionSearch is a Python 3 CLI tool that scrapes `.onion` search engines in parallel and aggregates results into CSV files. It requires a running Tor proxy (default: `127.0.0.1:9050`).

## Installation & Running

```bash
# Install from source
python3 setup.py install

# Run
onionsearch "search term"
onionsearch "search term" --engines tor66 deeplink phobos --limit 3 --output results.csv
onionsearch "search term" --exclude ahmia --continuous_write True
```

There is no test suite or linting configuration in this project.

## Architecture

All logic lives in `onionsearch/core.py` (~966 lines). Key components:

- **`ENGINES` dict**: Maps engine names to their base URLs (16 engines: ahmia, darksearchio, onionland, notevil, darksearchenginer, phobos, onionsearchserver, torgle, onionsearchengine, tordex, tor66, tormax, haystack, multivac, evosearch, deeplink)
- **Per-engine scraper functions** (e.g., `ahmia()`, `tor66()`): Each implements custom pagination and HTML parsing for that engine's structure
- **`link_finder()`**: Parses scraped HTML/JSON using BeautifulSoup CSS selectors and regex to extract result links
- **`scrape()`**: Entry point (registered as console script). Parses CLI args, spawns a `multiprocessing.Pool`, and aggregates results into a CSV

**Data flow:** `scrape()` → `multiprocessing.Pool` → per-engine functions → `link_finder()` → CSV output

**Multiprocessing:** Defaults to `cpu_count() - 1` parallel workers. Set `--mp_units 1` for sequential execution. Progress bars may display incorrectly with >1 worker but results are unaffected.

**Output fields:** Default CSV columns are `engine, name, link`. Available fields also include `domain`. Customizable via `--fields` and `--field_delimiter`.

## Adding a New Search Engine

1. Add an entry to the `ENGINES` dict with the engine name and base URL
2. Write a scraper function following the pattern of existing engines (pagination loop, requests via SOCKS5 proxy, pass HTML to `link_finder()`)
3. Register the function in the engines dispatch table used by `scrape()`
