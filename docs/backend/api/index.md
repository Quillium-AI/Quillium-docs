# API Reference

This document provides a comprehensive reference for the Quillium backend API endpoints.

## API Structure

The Quillium API follows RESTful principles and is organized around resources. All requests and responses use JSON format.

## Base URL

```
https://api.quillium.ai/v1
```

For local development:

```
http://localhost:8080/api/v1
```

## Authentication

Most API endpoints require authentication using JWT tokens. Include the token in the Authorization header:

```
Authorization: Bearer <access_token>
```

## Error Handling

The API uses conventional HTTP response codes to indicate success or failure:

- `2xx`: Success
- `4xx`: Client error (invalid request)
- `5xx`: Server error

Error responses include a JSON body with details:

```json
{
  "error": {
    "code": "invalid_request",
    "message": "The request was invalid",
    "details": "Additional information about the error"
  }
}
```

## Rate Limiting

API requests are limited to 100 requests per minute per user. Rate limit information is included in response headers:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 99
X-RateLimit-Reset: 1617120000
```

## Endpoints

### Authentication

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/auth/login` | POST | Authenticate user and get tokens |
| `/auth/refresh` | POST | Refresh access token |
| `/auth/logout` | POST | Invalidate tokens |
| `/auth/register` | POST | Register new user |

#### Login Example

Request:
```json
POST /auth/login
{
  "email": "user@example.com",
  "password": "secure_password"
}
```

Response:
```json
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expires_in": 900,
  "token_type": "Bearer"
}
```

### User Management

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/users/me` | GET | Get current user profile |
| `/users/me` | PUT | Update user profile |
| `/users/api-key` | POST | Generate API key |

### Chat Management

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/chats` | GET | List all chats |
| `/chats` | POST | Create new chat |
| `/chats/{id}` | GET | Get chat details |
| `/chats/{id}` | DELETE | Delete chat |
| `/chats/{id}/messages` | GET | Get chat messages |
| `/chats/{id}/messages` | POST | Send new message |

#### Create Chat Example

Request:
```json
POST /chats
{
  "title": "New Conversation",
  "model": "gpt-4"
}
```

Response:
```json
{
  "id": "chat_123456",
  "title": "New Conversation",
  "model": "gpt-4",
  "created_at": "2023-04-01T12:00:00Z",
  "updated_at": "2023-04-01T12:00:00Z"
}
```

### Settings Management

| Endpoint | Method | Description |
|----------|--------|-------------|
| `/settings/user` | GET | Get user settings |
| `/settings/user` | PUT | Update user settings |
| `/settings/admin` | GET | Get admin settings (admin only) |
| `/settings/admin` | PUT | Update admin settings (admin only) |

#### User Settings Example

Request:
```json
PUT /settings/user
{
  "theme": "dark",
  "language": "en",
  "notifications_enabled": true
}
```

Response:
```json
{
  "theme": "dark",
  "language": "en",
  "notifications_enabled": true,
  "updated_at": "2023-04-01T12:00:00Z"
}
```

## WebSocket API

Real-time chat functionality is provided through WebSocket connections.

### Connection

```
wss://api.quillium.ai/v1/ws?token=<access_token>
```

### Message Format

Messages sent and received through the WebSocket connection use the following format:

```json
{
  "type": "message",
  "data": {
    "chat_id": "chat_123456",
    "content": "Hello, world!",
    "sender": "user",
    "timestamp": "2023-04-01T12:00:00Z"
  }
}
```

### Message Types

- `message`: Chat message
- `typing`: Typing indicator
- `read`: Message read receipt
- `error`: Error notification

## API Versioning

The API version is included in the URL path (`/v1/`). When breaking changes are introduced, a new API version will be created.

## Pagination

List endpoints support pagination using the following query parameters:

- `limit`: Number of items per page (default: 20, max: 100)
- `offset`: Number of items to skip (default: 0)

Response includes pagination metadata:

```json
{
  "data": [...],
  "pagination": {
    "total": 100,
    "limit": 20,
    "offset": 0,
    "next": "/api/v1/chats?limit=20&offset=20",
    "previous": null
  }
}
```
