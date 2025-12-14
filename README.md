# Contacts API

A production-ready REST API for managing contacts built with FastAPI, SQLAlchemy 2.0, and Pydantic v2.

## Features

- **CRUD Operations**: Create, read, update, and delete contacts
- **Search**: Case-insensitive search by first name, last name, or email
- **Upcoming Birthdays**: Find contacts with birthdays in the next N days
- **Pagination**: Configurable limit/offset pagination
- **Validation**: Strict input validation with Pydantic v2
- **API Documentation**: Interactive Swagger UI and ReDoc

## Tech Stack

- **Python** 3.11+
- **FastAPI** - Modern async web framework
- **SQLAlchemy 2.0** - ORM with native async support
- **Pydantic v2** - Data validation
- **PostgreSQL** - Database
- **Alembic** - Database migrations
- **Poetry** - Dependency management
- **Docker** - Containerization

## Quick Start

### Option 1: Docker (Recommended)

1. **Clone and navigate to the project:**
   ```bash
   cd goit-pythonweb-hw-08
   ```

2. **Create environment file:**
   ```bash
   cp .env.example .env
   ```

3. **Start services:**
   ```bash
   docker-compose up -d
   ```

4. **Run migrations (automatic with docker-compose, or manually):**
   ```bash
   docker-compose exec api alembic upgrade head
   ```

5. **Access the API:**
   - Swagger UI: http://localhost:8000/docs
   - ReDoc: http://localhost:8000/redoc
   - Health check: http://localhost:8000/health

### Option 2: Local Development

1. **Prerequisites:**
   - Python 3.11+
   - PostgreSQL 15+
   - Poetry

2. **Install dependencies:**
   ```bash
   poetry install
   ```

3. **Create and configure `.env`:**
   ```bash
   cp .env.example .env
   # Edit .env with your database credentials
   ```

4. **Create PostgreSQL database:**
   ```bash
   createdb contacts_db
   # Or with Docker:
   docker run -d --name postgres -e POSTGRES_PASSWORD=mysecretpassword -e POSTGRES_DB=contacts_db -p 5432:5432 postgres:15-alpine
   ```

5. **Run migrations:**
   ```bash
   poetry run alembic upgrade head
   ```

6. **Start the server:**
   ```bash
   poetry run uvicorn app.main:app --reload
   ```

7. **Access the API:**
   - Swagger UI: http://localhost:8000/docs
   - ReDoc: http://localhost:8000/redoc

## API Endpoints

### Contacts

| Method | Endpoint | Description |
|--------|----------|-------------|
| POST | `/api/contacts` | Create a new contact |
| GET | `/api/contacts` | List contacts with filters |
| GET | `/api/contacts/{id}` | Get contact by ID |
| PUT | `/api/contacts/{id}` | Full update contact |
| PATCH | `/api/contacts/{id}` | Partial update contact |
| DELETE | `/api/contacts/{id}` | Delete contact |
| GET | `/api/contacts/upcoming-birthdays` | Get upcoming birthdays |

### Query Parameters for List Endpoint

| Parameter | Type | Description |
|-----------|------|-------------|
| `q` | string | General search (OR across first_name, last_name, email) |
| `first_name` | string | Filter by first name |
| `last_name` | string | Filter by last name |
| `email` | string | Filter by email |
| `limit` | int | Max items to return (1-100, default: 20) |
| `offset` | int | Items to skip (default: 0) |

### Search Semantics

- **Using `q`**: Searches first_name OR last_name OR email (OR semantics)
- **Using individual fields**: Filters with AND semantics
- **All searches**: Case-insensitive partial matches (ILIKE)

## Example Usage

### Create a Contact
```bash
curl -X POST "http://localhost:8000/api/contacts" \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "John",
    "last_name": "Doe",
    "email": "john.doe@example.com",
    "phone": "+1234567890",
    "birthday": "1990-05-15",
    "notes": "Met at conference"
  }'
```

### List Contacts with Search
```bash
# General search
curl "http://localhost:8000/api/contacts?q=john"

# Filter by specific field
curl "http://localhost:8000/api/contacts?first_name=john&limit=10"

# Pagination
curl "http://localhost:8000/api/contacts?limit=10&offset=20"
```

### Get Upcoming Birthdays
```bash
# Next 7 days (default)
curl "http://localhost:8000/api/contacts/upcoming-birthdays"

# Next 30 days
curl "http://localhost:8000/api/contacts/upcoming-birthdays?days=30"
```

### Update a Contact
```bash
# Full update (PUT)
curl -X PUT "http://localhost:8000/api/contacts/1" \
  -H "Content-Type: application/json" \
  -d '{
    "first_name": "John",
    "last_name": "Smith",
    "email": "john.smith@example.com",
    "phone": "+1234567890",
    "birthday": "1990-05-15"
  }'

# Partial update (PATCH)
curl -X PATCH "http://localhost:8000/api/contacts/1" \
  -H "Content-Type: application/json" \
  -d '{"notes": "Updated notes"}'
```

### Delete a Contact
```bash
curl -X DELETE "http://localhost:8000/api/contacts/1"
```

## Birthday Calculation

The upcoming birthdays endpoint computes each contact's next birthday:

1. If the birthday (month/day) has already passed this year → next year
2. If not yet passed → this year

**Leap Year Handling:**
- Feb 29 birthdays are treated as Feb 28 on non-leap years

## Running Tests

```bash
# Run all tests
poetry run pytest

# Run with coverage
poetry run pytest --cov=app

# Run specific test file
poetry run pytest tests/test_contacts.py -v
```

## Database Migrations

```bash
# Create a new migration
poetry run alembic revision --autogenerate -m "description"

# Apply migrations
poetry run alembic upgrade head

# Rollback one migration
poetry run alembic downgrade -1

# View migration history
poetry run alembic history
```

## Project Structure

```
.
├── app/
│   ├── __init__.py
│   ├── main.py           # FastAPI application
│   ├── db.py             # Database configuration
│   ├── models.py         # SQLAlchemy models
│   ├── schemas.py        # Pydantic schemas
│   ├── crud.py           # Data access layer
│   ├── deps.py           # FastAPI dependencies
│   ├── core/
│   │   ├── __init__.py
│   │   └── config.py     # Settings management
│   └── routers/
│       ├── __init__.py
│       └── contacts.py   # Contacts router
├── alembic/
│   ├── env.py
│   ├── script.py.mako
│   └── versions/
│       └── 0001_initial.py
├── tests/
│   ├── __init__.py
│   └── test_contacts.py
├── .env.example
├── alembic.ini
├── docker-compose.yaml
├── Dockerfile
├── pyproject.toml
├── requirements.txt
└── README.md
```

## Troubleshooting

### Database Connection Issues

1. **Check PostgreSQL is running:**
   ```bash
   docker ps | grep postgres
   ```

2. **Verify connection string in `.env`:**
   ```
   DATABASE_URL=postgresql+psycopg2://postgres:mysecretpassword@localhost:5432/contacts_db
   ```

3. **Test connection:**
   ```bash
   psql -h localhost -U postgres -d contacts_db
   ```

### Migration Issues

1. **Reset migrations (development only):**
   ```bash
   poetry run alembic downgrade base
   poetry run alembic upgrade head
   ```

2. **Check current revision:**
   ```bash
   poetry run alembic current
   ```

### Docker Issues

1. **Rebuild containers:**
   ```bash
   docker-compose down -v
   docker-compose build --no-cache
   docker-compose up -d
   ```

2. **View logs:**
   ```bash
   docker-compose logs -f api
   ```

## License

MIT

