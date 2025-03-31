# Backend Documentation

Welcome to the Quillium backend documentation. This section covers the Go backend that powers the Quillium application.

## Architecture

The Quillium backend is built with Go and follows a clean architecture approach with the following layers:

- **API Layer**: HTTP handlers and WebSocket connections
- **Service Layer**: Business logic and application services
- **Data Layer**: Database interactions and data models

## Key Components

- **REST API**: HTTP endpoints for client-server communication
- **WebSocket**: Real-time communication for chat functionality
- **Authentication**: JWT-based authentication system with refresh tokens
- **Database**: PostgreSQL database for data persistence

## Sections

- [Testing](testing/index.md): Documentation on testing practices and procedures
- API Reference: Details on available API endpoints (coming soon)
- Database Schema: Information about the database structure (coming soon)
- Deployment: Guidelines for deploying the backend (coming soon)

## Getting Started

To set up the backend for development:

1. Clone the repository
2. Install Go 1.20 or later
3. Install PostgreSQL
4. Set up environment variables
5. Run `go run cmd/server/main.go`

See the README in the repository for detailed setup instructions.
