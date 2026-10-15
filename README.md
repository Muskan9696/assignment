# Employee Attendance & Analytics API

FastAPI + PyMongo implementation for the HROne Software Engineer Trainee assignment.

## Run locally

Requirements: Python 3.11+ and MongoDB 6.0+.

```bash
python -m venv .venv
# Windows: .venv\Scripts\activate
# macOS/Linux: source .venv/bin/activate
pip install -r requirements.txt
# Set MONGO_URI and MONGO_DB in your shell, or use a local .env file (do not commit it).
uvicorn app.main:app --port 8000
```

`GET /health` checks MongoDB readiness. On startup, the app creates indexes idempotently. Swagger UI is available at `/docs`.

## Data

The grader seeds `employees` and `attendance_logs` directly. The included `sample_seed.py` can load the example Extended JSON into the database configured by `MONGO_URI` and `MONGO_DB`.

## Important implementation notes

- All API instants use integer epoch milliseconds; MongoDB stores UTC BSON datetimes.
- Attendance dates and shift times are interpreted in IST.
- Attendance list pagination and analytics use MongoDB queries/aggregation pipelines.
- Corrections append a history entry; concurrent correction conflicts return 409.

## Before submission

- Run tests against a clean MongoDB database and the provided sample seed.
- Exercise duplicate requests and overnight shifts.
- Inspect `/admin/explain/{endpoint}` plans on a large dataset.
- Do not commit `.env`, credentials, virtual environments, or a Dockerfile.

## Known limitations to verify before submission

This is a candidate implementation and should not be submitted without local verification against the complete contract. In particular, verify trend gap-filling for ranges where the department has no logs in part or all of the requested period, explain plans for every named endpoint, and every aggregation against MongoDB 6.0. Add automated tests for boundary timestamps, weekend records, employees with no logs, and concurrent requests.
