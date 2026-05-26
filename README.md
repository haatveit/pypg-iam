
# pypg-iam

Python library for [pg-iam](https://github.com/unioslo/pg-iam).

## Installation

### Basic installation (sync only)

```bash
pip install pypg-iam
# or
poetry add pypg-iam
```

This installs the synchronous API only, which uses SQLAlchemy and psycopg2.

### Installation with async support

**Note:** Async support requires SQLAlchemy 2.0 or higher. The synchronous API works with SQLAlchemy 1.4+.

```bash
# Default: psycopg3 (official PostgreSQL driver, recommended)
pip install pypg-iam[async] 'sqlalchemy>=2.0.0'
# or
poetry add pypg-iam[async] 'sqlalchemy>=2.0.0'

# Alternative: asyncpg (optimized for performance)
pip install pypg-iam[async-asyncpg] 'sqlalchemy>=2.0.0'
# or
poetry add pypg-iam[async-asyncpg] 'sqlalchemy>=2.0.0'
```

This includes an optional async driver (psycopg3 by default, or asyncpg as an alternative) for async support with SQLAlchemy.

## Features

- **Synchronous API**: Traditional blocking operations using SQLAlchemy
- **Async API** *(optional)*: Native async/await support using SQLAlchemy's asyncio extension with psycopg3 or asyncpg for modern async frameworks (FastAPI, aiohttp, etc.)

## Usage

### Synchronous (existing API)

```python
from iam.pgiam import Db, iam_engine

dsn = "postgresql://user:pw@host:5432/dbname"
engine = iam_engine(dsn)
db = Db(engine)

# Query data
result = db.exec_sql("select * from persons where name=:name", {'name': 'Alice'})
groups = db.person_groups(person_id)
```

### Asynchronous (new API)

```python
import asyncio
from iam.async_pgiam import AsyncDb, async_iam_engine

async def main():
    dsn = "postgresql://user:pw@host:5432/dbname"
    
    # Auto-detect: tries psycopg3 first, then asyncpg
    engine = async_iam_engine(dsn)
    
    # Or explicitly request a specific driver:
    # engine = async_iam_engine(dsn, driver='psycopg')  # psycopg3
    # engine = async_iam_engine(dsn, driver='asyncpg')  # asyncpg
    
    db = AsyncDb(engine)
    
    # Query data
    result = await db.exec_sql("select * from persons where name=:name", {'name': 'Alice'})
    groups = await db.person_groups(person_id)
    
    # Clean up
    await engine.dispose()

asyncio.run(main())
```

#### Choosing an Async Driver

Both psycopg3 and asyncpg are supported. Choose based on your needs:

- **psycopg3** (default): Official PostgreSQL driver, full feature coverage
- **asyncpg**: Faster, optimized for high-throughput applications

When using `driver='auto'` (the default), the function will use psycopg3 if available, otherwise asyncpg. If you explicitly request a driver that isn't installed, you'll get an ImportError with installation instructions.

# Running tests

## Synchronous tests

```bash
poetry install

# set postgres environment variables for pg-iam db access
export PYPGIAM_USER=""
export PYPGIAM_PW=""
export PYPGIAM_HOST=""
export PYPGIAM_DB=""

# run sync tests
poetry run pytest iam/tests.py
```

## Async tests

Async tests require pytest-asyncio and at least one async driver:

```bash
# Install test dependencies and async driver(s)
poetry install
pip install pytest-asyncio

# Install psycopg3 (default)
pip install pypg-iam[async]

# Or install asyncpg (alternative)
pip install pypg-iam[async-asyncpg]

# Or install both to test both drivers
pip install 'psycopg>=3.1' 'asyncpg>=0.27.0'

# Run async tests (tests both drivers if both are installed)
poetry run pytest iam/tests_async.py -v
```

# LICENSE

BSD.
