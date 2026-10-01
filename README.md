# FoD Compare — FastAPI + PyScript

**Compare two Fortify on Demand (FoD) scans and turn the differences into a checkable task list — with a Python backend *and* Python in the browser.**

![FoD Compare banner](web/img2.png)

> Built for the **PyCon Challenge 2023**.

## Overview

Security teams run multiple scans over the life of a project, but figuring out *what changed* between two releases is usually a manual, error-prone process.

FoD Compare solves that: enter two release IDs, and the app returns the list of files whose issues differ between the scans. Each file becomes a task in the UI, so reviewers can tick them off as they work through the results.

What makes it a fun Python project: the **entire application logic is written in Python** — the API runs on FastAPI, and the frontend runs on **PyScript**, which executes Python directly in the browser via Pyodide. No frontend framework required.

## Demo

Demo is hosted on AWS.

<!-- TODO: paste the live demo URL here -->
> **URL:** _TBD_

## Features

- Compare two FoD releases by ID and list the files with changed issues
- Interactive checklist UI — mark files as reviewed (strikethrough on done)
- Runs fully on sample data out of the box (plug in your own FoD keys for real scans)
- Async HTTP calls made from Python in the browser (`pyfetch`)
- Dockerfiles for both the API and the frontend
- Two independent servers (API + static frontend) — simple to reason about and deploy

## Tech Stack

| Layer    | Technology                              | Role                                        |
| -------- | --------------------------------------- | ------------------------------------------- |
| Backend  | [FastAPI](https://fastapi.tiangolo.com/) + Uvicorn | JSON API, input validation, CORS            |
| Frontend | [PyScript](https://pyscript.net/) (Pyodide)        | Runs Python in the browser                  |
| Browser libs | `requests`, `pyodide-http`          | Familiar request APIs, adapted for the browser |
| UI       | HTML, CSS, JS                           | Layout, styling, template-based rendering   |
| Runtime  | Python 3.11, Docker                     | Local dev and containerized deployment      |

## How It Works

```
┌──────────────────────────────┐         ┌──────────────────────────────┐
│  Browser (web/)              │         │  API (fod-sample-info.py)    │
│                              │  POST   │                              │
│  index.html                  │ ──────► │  /differences                │
│   └─ PyScript (Pyodide)      │  JSON   │   ├─ validate release IDs    │
│       service.py  ─ fill UI  │ ◄────── │   ├─ compute/write diff      │
│       utils.py    ─ fetch    │         │   └─ return sample results   │
│       request.py  ─ pyfetch  │         │       (differences4.json)    │
└──────────────────────────────┘         └──────────────────────────────┘
```

1. The user enters two release IDs and clicks **Compare** (`py-click="fill_tasks()"` in `web/index.html`).
2. `fill_tasks()` in `web/service.py` calls `get_JSON_Fetch()` in `web/utils.py`.
3. `web/request.py` sends an async `POST /differences` from the browser using `pyodide.http.pyfetch` — no page reload.
4. The FastAPI endpoint in `fod-sample-info.py` validates the IDs and responds with the comparison JSON.
5. The frontend iterates over the returned files and clones an HTML `<template>` for each one, attaching a checkbox handler so tasks can be marked done.

## Project Structure

```
.
├── fod-sample-info.py      # FastAPI app — POST /differences endpoint
├── differences4.json       # Sample comparison data (project, files)
├── requirements.txt        # Backend dependencies
├── Dockerfile              # API image
└── web/                    # Frontend (static, Python-in-the-browser)
    ├── index.html          # UI + PyScript bootstrap + task template
    ├── pyscript.toml       # Runtime packages for the browser
    ├── service.py          # Task list rendering & UI logic
    ├── utils.py            # API client (fetch + JSON parsing)
    ├── request.py          # Async fetch wrapper (pyfetch)
    └── Dockerfile          # Frontend image (nginx)
```

## Getting Started

### Prerequisites

- Python 3.11+
- (Optional) Docker

### 1. Run the API

From the repository root:

```console
pip install -r requirements.txt
uvicorn fod-sample-info:app --host 0.0.0.0 --port 8000 --reload
```

Or with Docker:

```console
docker build . --tag api.fod:2023
docker run -d -p 8000:8000 api.fod:2023
```

Verify it:

```console
curl -X POST http://localhost:8000/differences \
  -H "Content-Type: application/json" \
  -d '{"releaseid1": 6395564, "releaseid2": 6395413}'
```

### 2. Run the frontend

The frontend is plain static files served by any web server — Python's built-in server works:

```console
cd web
python -m http.server -b 0.0.0.0 80
```

Or with Docker:

```console
docker build -t frontend-fod web
docker run -d -p 80:80 frontend-fod
```

Then open `http://localhost` in your browser.

> **Local development note:** the API's CORS `allow_origins` list lives in `fod-sample-info.py`. When running locally, add your frontend origin (e.g. `http://localhost:80`) to that list so the browser is allowed to call the API.

## API

### `POST /differences`

**Request**

```json
{
  "releaseid1": 6395564,
  "releaseid2": 6395413
}
```

**Response** (sample data)

```json
{
  "releaseid1": 6395564,
  "releaseid2": 6395413,
  "project": "PetClinic",
  "files": [
    "Owner.java",
    "Specialty.java",
    "AbstractTraceAspect.java"
  ]
}
```

Both IDs are required and must be integers.

## What You Can Learn From This Project

A compact, real-world example of concepts that show up in most full-stack Python apps:

- **FastAPI basics** — defining an endpoint, reading a JSON body, returning a `JSONResponse`, and basic input validation.
- **CORS** — why browsers block cross-origin calls and how to configure allowed origins with middleware.
- **Python in the browser** — PyScript/Pyodide: importing modules, using `async/await`, and calling `pyfetch` instead of `requests`.
- **Interop** — running PyScript with runtime packages (`requests`, `pyodide-http`) declared in `pyscript.toml`.
- **Template-driven UI** — cloning an HTML `<template>` node per task and attaching event handlers from Python.
- **Service separation** — a stateless JSON API and a static frontend, deployable independently (with Dockerfiles for each).
- **Working with sample data** — structuring a PoC so it runs without external credentials or live integrations.

## Roadmap / To-Do

- [ ] Enhance the UI/UX for better task visualization
- [ ] Add export functionality for generated tasks (PDF/CSV)
- [ ] Wire the endpoint to real FoD scan results via [FoD-Compare](https://github.com/youngcs97/FoD-Compare)

## Credits

- FastAPI — https://fastapi.tiangolo.com/
- PyScript example — https://pyscript.net/examples/todo.html
- FoD Compare by Chris Young — https://github.com/youngcs97/FoD-Compare
- Challenge_pycon2023 demo — https://bitbucket.org/endava-pycon2023/challenge_pycon2023/src/master/
