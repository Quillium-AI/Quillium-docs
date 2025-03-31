# Backend Architecture

This document provides an overview of the Quillium backend architecture, explaining the design principles, component organization, and data flow.

## Architecture Overview

The Quillium backend follows a clean architecture approach, separating concerns into distinct layers that interact through well-defined interfaces. This design promotes maintainability, testability, and flexibility.

```
┌─────────────────────────────────────────────────────────────┐
│                                                             │
│  ┌─────────────┐    ┌─────────────┐    ┌─────────────────┐  │
│  │             │    │             │    │                 │  │
│  │  API Layer  │───▶│  Services   │───▶│  Data Access    │  │
│  │             │    │             │    │                 │  │
│  └─────────────┘    └─────────────┘    └─────────────────┘  │
│        ▲                                       │            │
│        │                                       │            │
│        └───────────────────────────────────────┘            │
│                                                             │
└─────────────────────────────────────────────────────────────┘
```

## Core Principles

1. **Separation of Concerns**: Each component has a single responsibility
2. **Dependency Inversion**: High-level modules don't depend on low-level modules
3. **Interface Segregation**: Clients depend only on the interfaces they use
4. **Clean Boundaries**: Clear separation between architectural layers
5. **Testability**: Components designed to be easily testable in isolation

## Architectural Layers

### 1. API Layer

The API layer is responsible for handling HTTP requests and WebSocket connections. It includes:

- **REST API Handlers**: Process HTTP requests and return responses
- **WebSocket Handlers**: Manage real-time communication
- **Middleware**: Handle cross-cutting concerns like authentication, CORS, and logging
- **Request/Response Models**: Define the structure of API payloads

Key components:
```go
// Handler example
func GetChatsHandler(w http.ResponseWriter, r *http.Request) {
    // Extract user ID from context (set by auth middleware)
    // Call service layer to get chats
    // Format and return response
}

// Middleware example
func WithAuth(next http.HandlerFunc) http.HandlerFunc {
    return func(w http.ResponseWriter, r *http.Request) {
        // Validate JWT token
        // Set user in request context
        next(w, r)
    }
}
```

### 2. Service Layer

The service layer contains the business logic of the application. It:

- Orchestrates operations across multiple repositories
- Implements business rules and validation
- Manages transactions when necessary
- Remains independent of HTTP or database specifics

Key components:
```go
// Service interface
type ChatService interface {
    GetChats(userID int) ([]Chat, error)
    CreateChat(userID int, title string, model string) (*Chat, error)
    DeleteChat(userID int, chatID int) error
}

// Service implementation
type chatService struct {
    chatRepo    repository.ChatRepository
    messageRepo repository.MessageRepository
}
```

### 3. Data Access Layer

The data access layer handles interactions with the database and external services:

- **Repositories**: Provide CRUD operations for domain entities
- **Models**: Define the structure of database entities
- **Migrations**: Manage database schema changes
- **External Clients**: Interact with third-party services

Key components:
```go
// Repository interface
type ChatRepository interface {
    GetByID(id int) (*Chat, error)
    GetByUserID(userID int) ([]Chat, error)
    Create(chat *Chat) (int, error)
    Update(chat *Chat) error
    Delete(id int) error
}

// Repository implementation
type PostgresChatRepository struct {
    db *sql.DB
}
```

## Cross-Cutting Concerns

### Authentication and Authorization

Authentication is implemented using JWT tokens with a refresh token pattern:

1. **Access Tokens**: Short-lived JWTs for API access
2. **Refresh Tokens**: Long-lived tokens stored in the database
3. **Middleware**: Validates tokens and sets user context

### Error Handling

The application uses a consistent error handling approach:

1. **Domain Errors**: Business logic errors with specific types
2. **HTTP Errors**: Mapped from domain errors in the API layer
3. **Logging**: Structured logging of errors with context

```go
// Error types
var (
    ErrNotFound      = errors.New("resource not found")
    ErrUnauthorized  = errors.New("unauthorized access")
    ErrInvalidInput  = errors.New("invalid input")
)

// Error handling
func handleError(w http.ResponseWriter, err error) {
    switch {
    case errors.Is(err, ErrNotFound):
        respondJSON(w, http.StatusNotFound, ErrorResponse{Error: "Resource not found"})
    case errors.Is(err, ErrUnauthorized):
        respondJSON(w, http.StatusUnauthorized, ErrorResponse{Error: "Unauthorized"})
    default:
        respondJSON(w, http.StatusInternalServerError, ErrorResponse{Error: "Internal server error"})
    }
}
```

### Logging

The application uses structured logging to capture important information:

```go
// Structured logging
logger.WithFields(log.Fields{
    "user_id": userID,
    "action": "create_chat",
    "chat_id": chatID,
}).Info("Chat created successfully")
```

## Data Flow

### Request Handling Flow

1. **HTTP Request**: Client sends request to an endpoint
2. **Middleware**: Request passes through middleware chain (CORS, authentication, etc.)
3. **Handler**: Handler extracts parameters and calls appropriate service
4. **Service**: Service implements business logic and calls repositories
5. **Repository**: Repository interacts with the database
6. **Response**: Data flows back through the layers to the client

```
Client → Middleware → Handler → Service → Repository → Database
       ←           ←         ←         ←           ←
```

### WebSocket Flow

1. **Connection**: Client establishes WebSocket connection
2. **Authentication**: Connection is authenticated via token
3. **Subscription**: Client subscribes to specific chat rooms
4. **Message Handling**: Messages are processed by appropriate handlers
5. **Broadcasting**: Messages are broadcast to subscribed clients

## Dependency Injection

The application uses a simple dependency injection pattern:

```go
// Create repositories
chatRepo := repository.NewChatRepository(db)
messageRepo := repository.NewMessageRepository(db)

// Create services
chatService := service.NewChatService(chatRepo, messageRepo)

// Create handlers
chatHandler := handler.NewChatHandler(chatService)

// Register routes
router.HandleFunc("/api/chats", chatHandler.GetChats).Methods("GET")
```

## Configuration Management

Configuration is managed through environment variables and loaded at startup:

```go
type Config struct {
    DatabaseURL      string
    Port             string
    JWTSecret        string
    AccessTokenExp   time.Duration
    RefreshTokenExp  time.Duration
    LogLevel         string
}

func LoadConfig() (*Config, error) {
    // Load from environment variables
}
```

## Testing Strategy

The architecture supports different testing approaches:

1. **Unit Tests**: Test individual components in isolation
2. **Integration Tests**: Test interactions between components
3. **API Tests**: Test HTTP endpoints end-to-end

```go
// Unit test example
func TestChatService_GetChats(t *testing.T) {
    // Create mock repository
    mockRepo := new(MockChatRepository)
    mockRepo.On("GetByUserID", 1).Return([]Chat{...}, nil)
    
    // Create service with mock
    service := NewChatService(mockRepo)
    
    // Call method and assert results
    chats, err := service.GetChats(1)
    assert.NoError(t, err)
    assert.Len(t, chats, 2)
}
```

## Scalability Considerations

The architecture supports horizontal scaling:

1. **Stateless Design**: No server-side session state
2. **Database Connection Pooling**: Efficient use of database connections
3. **Caching**: Optional caching layer for frequently accessed data
4. **Message Queues**: Can be added for asynchronous processing

## Future Architecture Evolution

The architecture is designed to evolve with the application's needs:

1. **Microservices**: Components can be extracted into separate services
2. **Event Sourcing**: Can be implemented for specific domains
3. **CQRS**: Command Query Responsibility Segregation can be added
4. **GraphQL**: Can be implemented alongside REST API
