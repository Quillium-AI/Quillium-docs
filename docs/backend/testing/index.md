# Backend Testing

This section covers the testing infrastructure and practices for the Quillium backend.

## Testing Philosophy

The Quillium backend follows a comprehensive testing approach that includes:

- **Unit Tests**: Testing individual functions and components in isolation
- **Integration Tests**: Testing how components work together with real dependencies
- **API Tests**: Testing the API endpoints from an external perspective

## Test Categories

- [Integration Tests](integration_tests.md): How to set up and run integration tests
- Unit Tests: Guidelines for writing effective unit tests (coming soon)
- API Tests: How to test the API endpoints (coming soon)

## Test Coverage

The goal is to maintain high test coverage across the codebase, focusing on critical paths and business logic. The current test suite covers:

- Authentication and authorization
- Chat functionality
- User settings management
- Admin settings management
- CORS middleware

## Running Tests

For quick development cycles, you can run unit tests without any additional setup:

```bash
go test ./...
```

For more comprehensive testing including integration tests, see the [Integration Tests](integration_tests.md) documentation.
