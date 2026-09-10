# AGENTS.md

Small PoC (PyCon Challenge 2023): compares two Fortify on Demand (FoD) scans. FastAPI backend serves sample comparison data; PyScript frontend (browser-side Python) renders tasks.

## Layout
- `fod-sample-info.py` — the entire backend. Single FastAPI app with one endpoint: `POST /differences` (takes `{releaseid1, releaseid2}`, currently returns hardcoded sample data from `differences4.json`).
- `web/` — frontend: `index.html` + PyScript (`pyscript.toml` declares runtime packages `requests`, `pyodide-http`). Python logic lives in `web/request.py`, `web/service.py`, `web/utils.py`; these run in the browser via PyScript, not via regular Python.
- No tests, no linter, no CI.

## Running
- Backend: `pip install -r requirements.txt` then `uvicorn fod-sample-info:app --host 0.0.0.0 --port 8000 --reload` (must be run from repo root — the app reads `differences4.json` and `web/` via relative paths).
- Frontend: `cd web && python -m http.server -b 0.0.0.0 80` (separate server from the API).
- Both auto-reload Dockerfiles exist (backend on Python 3.11, frontend on nginx).

## Gotchas
- CORS `allow_origins` in `fod-sample-info.py` is hardcoded to an AWS EC2 hostname; locally the browser client may be blocked unless origins are edited.
- `/differences` ignores real FoD integration: it writes `differences2.json` (generated) but always returns `differences4.json` as the response. Real scan data comes from https://github.com/youngcs97/FoD-Compare; a `.config.json` with FoD keys would be needed for real scans (not in repo).
- Backend Python code has mixed tabs/spaces inside `fod-sample-info.py` — preserve formatting carefully when editing.
- Frontend scripts load PyScript from the CDN (`pyscript.net/latest`), so an internet connection is required to serve the UI.
