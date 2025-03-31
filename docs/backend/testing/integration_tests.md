# Integration Tests

This document explains how to set up and run the integration tests for the Quillium backend.

## Overview

The Quillium backend includes comprehensive integration tests that verify the functionality of various components including:

1. **Chat Handlers**: Tests for creating, retrieving, and deleting chat conversations
2. **Settings Handlers**: Tests for user and admin settings management
3. **Authentication**: Tests for login, logout, and user management
4. **CORS Middleware**: Tests for proper cross-origin resource sharing configuration

## Test Structure

All tests follow the table-driven testing pattern which is idiomatic in Go. This approach makes it easy to add new test cases and maintain existing ones.

Example:
```go
tests := []struct {
    name       string
    input      SomeType
    expected   ExpectedType
    shouldFail bool
}{
    {"Test case 1", input1, expected1, false},
    {"Test case 2", input2, expected2, true},
}

for _, tc := range tests {
    t.Run(tc.name, func(t *testing.T) {
        // Test logic here
    })
}
```

## Setting Up the Test Environment

### Test Database

The integration tests require a PostgreSQL test database. By default, the tests will try to connect to:

```
postgres://postgres:postgres@localhost:5432/quillium_test
```

You can customize this by setting the `TEST_DATABASE_URL` environment variable.

### Setting Up the Test Database

1. Create a PostgreSQL database for testing:

```bash
psql -U postgres -c "CREATE DATABASE quillium_test;"
```

2. The test database schema will be automatically created when running the tests.

## Running Integration Tests

By default, integration tests are skipped to allow for faster unit testing during development. To run the integration tests, set the `INTEGRATION_TESTS` environment variable to `1`.

### Running All Tests

```bash
# Windows PowerShell
$env:INTEGRATION_TESTS=1
go test ./...

# Linux/macOS
INTEGRATION_TESTS=1 go test ./...
```

### Running Specific Test Packages

```bash
# Windows PowerShell
$env:INTEGRATION_TESTS=1
go test -v ./internal/api/restapi/handlers

# Linux/macOS
INTEGRATION_TESTS=1 go test -v ./internal/api/restapi/handlers
```

### Running Individual Tests

```bash
# Windows PowerShell
$env:INTEGRATION_TESTS=1
go test -v ./internal/api/restapi/handlers -run TestGetChats

# Linux/macOS
INTEGRATION_TESTS=1 go test -v ./internal/api/restapi/handlers -run TestGetChats
```

## Test Coverage

### Chat Handlers Tests

- **TestGetChats**: Verifies that the GetChats handler correctly retrieves all chats for a user
- **TestCreateChat**: Tests the creation of a new chat
- **TestDeleteChat**: Ensures that a chat can be properly deleted

### Settings Handlers Tests

- **TestGetAdminSettings**: Verifies that admin settings can only be accessed by admin users
- **TestUpdateAdminSettings**: Tests updating admin settings with proper authorization
- **TestUpdateUserSettings**: Ensures that user settings can be updated correctly

### Authentication Tests

- **TestLogin**: Verifies the login process and JWT token generation
- **TestLogout**: Tests the logout functionality
- **TestGenerateAPIKey**: Ensures API keys can be generated for users
- **TestGetCurrentUser**: Verifies that user information can be retrieved

### CORS Middleware Tests

- **TestWithCORS**: Tests that CORS headers are correctly set based on the CORS type
- **TestPreflightRequests**: Ensures that preflight (OPTIONS) requests are handled correctly

## Refresh Token Implementation

The backend is designed to support a refresh token pattern for authentication, which involves:

1. Short-lived access tokens (JWTs)
2. Long-lived refresh tokens stored in the database

This approach enhances security by limiting the lifetime of access tokens while providing a seamless user experience through automatic token refresh.

## Troubleshooting

### Database Connection Issues

If you encounter database connection errors:

1. Ensure PostgreSQL is running
2. Verify the database credentials
3. Check that the test database exists
4. Confirm that the `TEST_DATABASE_URL` environment variable is set correctly if you're using a custom database URL

### Test Failures

If tests are failing:

1. Check the error messages for specific issues
2. Verify that the database schema is up to date
3. Ensure that the test environment has the necessary permissions

## Adding New Tests

When adding new functionality to the backend, follow these guidelines for creating tests:

1. Create table-driven tests for comprehensive coverage
2. Include both positive and negative test cases
3. Properly clean up test data to avoid affecting other tests
4. Use descriptive test names that explain what is being tested
