# vb_sync_api

FastAPI service for managing synchronisation lists between VB and downstream systems, including batch processing and status tracking.

---

## Overview

This service exposes a batch-based synchronisation API used to exchange change lists between systems.

### Flow

- VB creates synchronisation lists
- Each list contains affected records (`vb_uuid`)
- Each record is marked as:
  - `updated`
  - `deleted`
- Lobster polls the API for new lists
- Lobster processes records and forwards them to downstream systems (e.g. BI)
- Lobster updates the processing status of each list

### Lifecycle

```
NEW → IN_PROCESSING → PROCESSED
```

---

## Features

- List synchronisation batches
- Filter by processing status
- Fetch batch details
- Update batch status
- PostgreSQL support
- SQLAlchemy ORM
- Alembic migrations
- Auto-generated OpenAPI docs (FastAPI)

---

## API Endpoints

### `GET /syncronisation`

Returns synchronisation lists.

#### Query Parameters

- `status` (optional): `NEW | IN_PROCESSING | PROCESSED`

#### Example

```http
GET /syncronisation?status=NEW
```

#### Response

```json
{
  "entry_lists": [
    {
      "list_id": "7f3a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
      "create_datetime": "2026-04-02T09:15:30Z",
      "status": "NEW"
    }
  ]
}
```

---

### `GET /syncronisation/{list_id}`

Returns full details of a synchronisation list.

#### Example

```http
GET /syncronisation/7f3a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c
```

#### Response

```json
{
  "list_id": "7f3a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
  "create_datetime": "2026-04-02T09:15:30Z",
  "status": "NEW",
  "entry_list": [
    {
      "vb_uuid": "a3f1c2d4-8b7e-4f2a-9c1d-0e5f6a7b8c9d",
      "status": "updated"
    },
    {
      "vb_uuid": "b1e2d3c4-7a6f-4e5b-8d9c-1f0a2b3c4d5e",
      "status": "deleted"
    }
  ]
}
```

---

### `PATCH /syncronisation/{list_id}`

Updates the status of a synchronisation list.

#### Request

```http
PATCH /syncronisation/{list_id}
Content-Type: application/json
```

```json
{
  "status": "IN_PROCESSING"
}
```

#### Response

```json
{
  "list_id": "7f3a1b2c-4d5e-6f7a-8b9c-0d1e2f3a4b5c",
  "status": "IN_PROCESSING"
}
```

---

## Processing Flow

```text
1. GET /syncronisation?status=NEW
2. GET /syncronisation/{list_id}
3. PATCH /syncronisation/{list_id} → IN_PROCESSING
4. Process entries
5. PATCH /syncronisation/{list_id} → PROCESSED
```

### Entry Processing

- `updated` → fetch full data from VB (separate endpoints)
- `deleted` → remove in downstream system

---

## Project Structure

```
app/
  main.py          # FastAPI entrypoint
  database.py      # DB connection
  models.py        # SQLAlchemy models
  schemas.py       # Pydantic schemas
  crud.py          # DB logic

alembic/
  versions/        # migrations
```

---

## Setup

### 1. Create virtual environment

```bash
python -m venv venv
source venv/bin/activate
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure database

```bash
export DATABASE_URL=postgresql+psycopg2://user:password@localhost/dbname
```

### 4. Run migrations

```bash
alembic upgrade head
```

### 5. Start API

```bash
uvicorn app.main:app --reload
```

---

## OpenAPI Docs

Available at:

```
http://localhost:8000/docs
```

---

## Notes

- Endpoint path uses `/syncronisation` (typo kept for compatibility)
- Recommended to rename to `/synchronisation` in future versions
- Enforce state transitions:

```
NEW → IN_PROCESSING → PROCESSED
```

---

## Future Improvements

- Authentication (JWT / API key)
- Retry & dead-letter queue for failed batches
- Pagination for large lists
- Event-driven alternative (webhooks / Kafka)
