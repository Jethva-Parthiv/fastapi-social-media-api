# FastAPI Project (Image Upload + Feed)

A small FastAPI backend that:
- Accepts file uploads (`/upload`) using `multipart/form-data`
- Uploads the file to **ImageKit**
- Stores post metadata in a database (SQLAlchemy async, SQLite by default)
- Returns a feed of posts (`/feed`) ordered by newest first

---

## Project Structure

- `main.py`
  - Uvicorn entrypoint
- `app/app.py`
  - FastAPI application + API routes
- `app/db.py`
  - SQLAlchemy async engine/session
  - `Post` model and table creation
- `app/images.py`
  - ImageKit client initialization using environment variables
- `app/schema.py`
  - (Currently minimal / placeholder)

---

## Endpoints

### 1) `POST /upload`
Uploads an image/video, forwards it to ImageKit, then stores metadata.

**Request** (`multipart/form-data`):
- `file` (required): `UploadFile`
- `caption` (optional): `str` (default: `""`)

**Response**:
- The created `Post` record (SQLAlchemy model instance serialized by FastAPI)

---

### 2) `GET /feed`
Returns all posts ordered by `created_at` descending.

**Response**:
```json
{
  "posts": [
    {
      "id": "...",
      "caption": "...",
      "url": "...",
      "file_type": "image|video",
      "file_name": "...",
      "created_at": "..."
    }
  ]
}
```

---

## Requirements

Uses:
- FastAPI
- Uvicorn
- SQLAlchemy (async)
- aiosqlite (SQLite)
- python-dotenv
- imagekitio

Dependencies are listed in `pyproject.toml`.

---

## Configuration (.env)

Create a `.env` file in the project root with at least:

- `IMAGEKIT_PRIVATE_KEY`
- `IMAGEKIT_URL_ENDPOINT`

The `IMAGEKIT_URL_ENDPOINT` variable is read in `app/images.py` (note: `private_key` is required for the ImageKit client as written).

---

## Running the App

### 1) Install dependencies
Using your Python environment (project uses `pyproject.toml`).

### 2) Start the server
From `Project_Root/`:

```bash
uvicorn app.app:app --host 0.0.0.0 --port 8000 --reload
```

Or run:

```bash
python main.py
```

---

## Notes / Implementation Details

- Database:
  - Default `DATABASE_URL` in `app/db.py` is:
    - `sqlite+aiosqlite:///./test.db`
  - Tables are created on app startup via FastAPI `lifespan`.

- File handling:
  - The API writes the incoming upload to a temporary file, uploads it to ImageKit, then deletes the temporary file.

- `file_type`:
  - Determined from `file.content_type`:
    - Starts with `video` => `video`
    - Otherwise => `image`

---

## Example Usage

### Upload an image
```bash
curl -X POST "http://localhost:8000/upload" \
  -F "caption=My caption" \
  -F "file=@/path/to/image.jpg"
```

### Get the feed
```bash
curl "http://localhost:8000/feed"
```

---

## TODO / Improvements (Optional)
- Add proper Pydantic response schemas (`app/schema.py` is currently minimal).
- Improve error handling (currently re-raises raw exceptions).
- Ensure ImageKit is configured with required fields (`file_name`/tags/return fields) for your use case.

