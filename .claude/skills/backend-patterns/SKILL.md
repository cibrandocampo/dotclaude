---
name: backend-patterns
description: Backend architecture patterns and conventions for this project. Customize this skill with the project's actual stack, layer structure, and coding conventions.
---

# Backend Patterns

> **Customize this skill for your project.** Fill in each section with the actual conventions and patterns used. Reference the `architecture` skill for layer responsibilities.

## Stack

> _Fill in: language, framework, database, cache, queue, test framework._

```
Language:        # e.g. Python 3.12, Node 20, Go 1.22
Framework:       # e.g. Django 5 + DRF, FastAPI, Express, Gin
Database:        # e.g. PostgreSQL 16, MySQL, SQLite
Cache:           # e.g. Redis 7, Memcached
Queue:           # e.g. Celery + Redis, BullMQ, none
Tests:           # e.g. pytest, Jest, Go test
```

## Project Structure

> _Fill in: directory layout and what lives where._

```
src/
  # Fill in the project's actual structure
  # e.g. for Django:
  # apps/<app>/models.py
  # apps/<app>/views.py
  # apps/<app>/serializers.py
  # apps/<app>/tests/
```

## Reuse-First Principle

Before writing new backend code:

1. **Search for existing utilities** — helpers, mixins, base classes that cover this.
2. **Check existing models/schemas** — does the data already exist somewhere?
3. **Check existing services** — does a use case already implement this logic?

The 1/2/3 rule: once is fine, twice is a pattern, three times means extract.

## Models / Entities

> _Fill in: how models are defined, naming conventions, field conventions._

```
Naming:          # e.g. singular PascalCase, snake_case fields
Timestamps:      # e.g. created_at / updated_at on every model
Soft delete:     # e.g. deleted_at field vs hard delete
Indexing:        # e.g. always index FK fields, search fields
```

## Services / Use Cases

> _Fill in: how business logic is structured._

```
Location:        # e.g. apps/<app>/services.py, src/use-cases/
Naming:          # e.g. verb_noun() functions, NounVerb classes
Input/output:    # e.g. dataclasses, typed dicts, plain dicts
Side effects:    # e.g. always explicit, never hidden in getters
```

## API Conventions

> _Fill in: URL structure, HTTP method semantics, response format, error format._

```
URL style:       # e.g. /api/v1/<resource>/, trailing slash, kebab-case
Methods:         # GET list/detail, POST create, PATCH update, DELETE remove
Success:         # e.g. 200 with body, 201 for create, 204 for delete
Errors:          # e.g. { "error": "message", "code": "ERROR_CODE" }
Auth:            # e.g. JWT Bearer, session cookie, API key header
Pagination:      # e.g. { "results": [], "count": N, "next": url }
```

## Error Handling

> _Fill in: how errors are caught, logged, and returned._

```
Exceptions:      # e.g. domain exceptions vs infrastructure exceptions
Logging:         # e.g. log at service boundary, not inside domain
User messages:   # e.g. generic for 5xx, specific for 4xx
Stack traces:    # e.g. never exposed to client, always in server logs
```

## Database Patterns

> _Fill in: query conventions, transaction handling, migration rules._

```
Queries:         # e.g. always through ORM, raw SQL only for complex reporting
Transactions:    # e.g. wrap multi-step writes in a transaction
Migrations:      # e.g. always reversible, never drop columns directly
N+1:             # e.g. use select_related/prefetch_related, never lazy load in loops
```

## Testing

> _Fill in: what to test and at which layer._

```
Unit:            # e.g. domain logic in isolation, no DB
Integration:     # e.g. services + DB, real DB in test transaction
API:             # e.g. full request/response cycle, authenticated
Fixtures:        # e.g. factory functions, not fixtures files
Coverage:        # e.g. 100% on domain layer, 80% minimum overall
```
