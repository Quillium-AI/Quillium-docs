# Database Schema

This document describes the database schema used in the Quillium backend.

## Overview

Quillium uses PostgreSQL as its primary database. The schema is designed to support the core features of the application, including user management, chat functionality, and settings management.

## Entity Relationship Diagram

```
┌─────────────┐       ┌─────────────┐       ┌─────────────┐
│    Users    │       │    Chats    │       │  Messages   │
├─────────────┤       ├─────────────┤       ├─────────────┤
│ id          │       │ id          │       │ id          │
│ email       │       │ user_id     │──┐    │ chat_id     │──┐
│ password    │       │ title       │  │    │ content     │  │
│ name        │       │ model       │  │    │ role        │  │
│ created_at  │       │ created_at  │  │    │ created_at  │  │
│ updated_at  │       │ updated_at  │  │    │ tokens      │  │
└─────────────┘       └─────────────┘  │    └─────────────┘  │
       │                               │                     │
       │                               │                     │
       │                               │                     │
       ▼                               │                     │
┌─────────────┐                        │                     │
│ UserSettings│                        │                     │
├─────────────┤                        │                     │
│ user_id     │◄──────────────────────┘                     │
│ theme       │                                              │
│ language    │                                              │
│ created_at  │                                              │
│ updated_at  │                                              │
└─────────────┘                                              │
                                                             │
┌─────────────┐                                              │
│AdminSettings│                                              │
├─────────────┤                                              │
│ id          │                                              │
│ key         │                                              │
│ value       │                                              │
│ created_at  │                                              │
│ updated_at  │                                              │
└─────────────┘                                              │
                                                             │
┌─────────────┐                                              │
│RefreshTokens│                                              │
├─────────────┤                                              │
│ id          │                                              │
│ user_id     │◄──────────────────────────────────────────┘
│ token       │
│ expires_at  │
│ created_at  │
│ revoked     │
└─────────────┘
```

## Table Definitions

### Users

Stores user account information.

```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) NOT NULL UNIQUE,
    password VARCHAR(255) NOT NULL,
    name VARCHAR(255) NOT NULL,
    is_admin BOOLEAN NOT NULL DEFAULT FALSE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### Chats

Stores chat conversations.

```sql
CREATE TABLE chats (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    title VARCHAR(255) NOT NULL,
    model VARCHAR(50) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### Messages

Stores individual messages within chats.

```sql
CREATE TABLE messages (
    id SERIAL PRIMARY KEY,
    chat_id INTEGER NOT NULL REFERENCES chats(id) ON DELETE CASCADE,
    content TEXT NOT NULL,
    role VARCHAR(50) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    tokens INTEGER NOT NULL DEFAULT 0
);
```

### UserSettings

Stores user-specific settings.

```sql
CREATE TABLE user_settings (
    user_id INTEGER PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,
    theme VARCHAR(50) NOT NULL DEFAULT 'light',
    language VARCHAR(10) NOT NULL DEFAULT 'en',
    notifications_enabled BOOLEAN NOT NULL DEFAULT TRUE,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### AdminSettings

Stores global application settings.

```sql
CREATE TABLE admin_settings (
    id SERIAL PRIMARY KEY,
    key VARCHAR(255) NOT NULL UNIQUE,
    value TEXT NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()
);
```

### RefreshTokens

Stores refresh tokens for authentication.

```sql
CREATE TABLE refresh_tokens (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    token VARCHAR(255) NOT NULL,
    expires_at TIMESTAMP NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    revoked BOOLEAN NOT NULL DEFAULT FALSE
);
```

### ApiKeys

Stores API keys for programmatic access.

```sql
CREATE TABLE api_keys (
    id SERIAL PRIMARY KEY,
    user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
    key VARCHAR(255) NOT NULL UNIQUE,
    name VARCHAR(255) NOT NULL,
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),
    last_used_at TIMESTAMP
);
```

## Indexes

```sql
-- Improve query performance for common operations
CREATE INDEX idx_chats_user_id ON chats(user_id);
CREATE INDEX idx_messages_chat_id ON messages(chat_id);
CREATE INDEX idx_refresh_tokens_user_id ON refresh_tokens(user_id);
CREATE INDEX idx_api_keys_user_id ON api_keys(user_id);
```

## Migrations

Database migrations are managed using the [golang-migrate](https://github.com/golang-migrate/migrate) tool. Migration files are stored in the `migrations` directory.

### Creating a Migration

```bash
migrate create -ext sql -dir migrations -seq migration_name
```

This creates two files:
- `migrations/NNNNNN_migration_name.up.sql`: Changes to apply
- `migrations/NNNNNN_migration_name.down.sql`: Changes to revert

### Running Migrations

```bash
# Apply all pending migrations
migrate -database ${DATABASE_URL} -path migrations up

# Revert the last migration
migrate -database ${DATABASE_URL} -path migrations down 1
```

## Database Access

The backend uses a repository pattern to abstract database access. Each entity has a corresponding repository that handles CRUD operations.

Example repository interface:

```go
type UserRepository interface {
    GetByID(id int) (*User, error)
    GetByEmail(email string) (*User, error)
    Create(user *User) (int, error)
    Update(user *User) error
    Delete(id int) error
}
```

## Connection Management

Database connections are managed using a connection pool to ensure efficient resource utilization. The pool is configured with:

- Maximum open connections: 25
- Maximum idle connections: 5
- Connection lifetime: 5 minutes

## Transactions

For operations that require atomicity, the repository methods accept an optional transaction parameter:

```go
func (r *PostgresUserRepository) Create(user *User, tx *sql.Tx) (int, error) {
    // Use provided transaction or create a new one
    // Execute SQL statements
    // Return result
}
```

## Testing

The database layer includes comprehensive tests that verify the functionality of each repository. Integration tests use a test database to ensure that the repositories work correctly with the actual database schema.

See the [Integration Tests](../testing/integration_tests.md) documentation for more information on setting up and running database tests.
