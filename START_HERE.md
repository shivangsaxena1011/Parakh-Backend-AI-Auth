# Parakh backend integration package

This package is the updated Backend + Authentication + AI/ML integration bundle, reviewed against the team's current frontend repository.

## Contents

- `deepfake-backend/` — FastAPI backend, authentication, persisted detection history, image/video inference hooks, evaluation scripts, Docker files and automated tests.
- `frontend-integration/parakh-api.ts` — typed browser API client to copy into the Next.js frontend (example path `lib/parakh-api.ts`).
- `frontend-integration/FRONTEND_INTEGRATION.md` — frontend findings, API integration contract, environment variables, and an example replacement for the current mock analysis handler.

## Main difference from the previous ZIP

- CORS now allows the current GitHub Pages origin `https://anuragverma0907.github.io` and the project domains listed in the frontend README, along with local dev origins.
- The application title is branded as Parakh.
- A typed Next.js-compatible API client and specific instructions explain how to wire the current frontend upload form to actual backend inference.
- A CORS preflight test was added.

## Validation performed

Run from `deepfake-backend/`:

```bash
pytest -q
```

The included tests passed in the packaging environment (11 tests). Model inference uses the real configured pretrained checkpoint on the server; tests mock inference only for backend/API wiring. The pretrained weights were not downloaded and run during packaging, so the first real model inference must still be validated after installing `requirements-ml.txt`.

## Current frontend caveat

The live GitHub Pages page is currently an informational/coming-soon page. The Next.js repository contains an upload prototype, but it simulates analysis progress and displays fixed mock scores. The included integration guide explains what the frontend teammate needs to replace. The backend cannot become reachable from the deployed static site until it is separately deployed to a public HTTPS URL and `NEXT_PUBLIC_PARAKH_API_URL` is configured.
