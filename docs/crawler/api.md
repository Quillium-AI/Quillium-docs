# Crawler API

The Quillium-Crawler includes an API server that provides HTTP endpoints for health checks, version information, and metrics. This document explains the available endpoints and how to use them.

## Overview

The API server provides a RESTful interface for:

- Health and readiness checks
- Version information
- Prometheus metrics

## Configuration

The API server is automatically started when the crawler runs. By default, it listens on port 8080, but this can be configured in the code:

```go
// Start API server
if err := api.StartServer(":8080"); err != nil {
    log.Fatalf("Failed to start server: %v", err)
}
```

## Available Endpoints

### Health Check Endpoint

```
GET /livez
```

Returns the liveness status of the crawler service. This endpoint is used by container orchestration systems like Kubernetes to determine if the service is alive.

#### Example Response

```json
{
  "status": "ok"
}
```

### Readiness Check Endpoint

```
GET /readyz
```

Returns the readiness status of the crawler service. This endpoint is used by container orchestration systems to determine if the service is ready to accept traffic.

#### Example Response

```json
{
  "status": "ok"
}
```

### Version Endpoint

```
GET /version
```

Returns the current version of the crawler service.

#### Example Response

```json
{
  "version": "0.1.0"
}
```

### Metrics Endpoint

```
GET /metrics
```

Returns Prometheus-compatible metrics for the crawler. This endpoint is automatically provided by the Prometheus client library when metrics are enabled.

## Implementation Details

The API server is implemented in the `api` package using the standard Go HTTP server:

```go
func StartServer(addr string) error {
    mux := http.NewServeMux()
    SetupRoutes(mux)

    log.Printf("Starting server on %s", addr)
    return http.ListenAndServe(addr, mux)
}
```

The routes are set up in a separate function:

```go
func SetupRoutes(mux *http.ServeMux) {
    // Health and system endpoints
    mux.HandleFunc("/livez", livezHandler)
    mux.HandleFunc("/readyz", readyzHandler)
    mux.HandleFunc("/version", versionHandler)

    // Metrics endpoint
    mux.Handle("/metrics", promhttp.Handler())
}
```

### Route Handlers

#### Health Check Handlers

The health check handlers provide simple status responses:

```go
func livezHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    response := map[string]string{
        "status": "ok",
    }
    json.NewEncoder(w).Encode(response)
    log.Println("Live check request received")
}

func readyzHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    response := map[string]string{
        "status": "ok",
    }
    json.NewEncoder(w).Encode(response)
    log.Println("Ready check request received")
}
```

#### Version Handler

The version handler returns the current version of the crawler:

```go
func versionHandler(w http.ResponseWriter, r *http.Request) {
    w.Header().Set("Content-Type", "application/json")
    response := map[string]string{
        "version": "0.1.0",
    }
    json.NewEncoder(w).Encode(response)
}
```

## Using the API

### Checking Health Status

You can check the health status using curl:

```bash
curl http://localhost:8080/livez
```

### Checking Readiness Status

To check if the service is ready:

```bash
curl http://localhost:8080/readyz
```

### Getting Version Information

To get the current version:

```bash
curl http://localhost:8080/version
```

### Accessing Metrics

To view the Prometheus metrics:

```bash
curl http://localhost:8080/metrics
```

## Security Considerations

The API server does not include authentication or authorization by default. In production environments, consider:

1. Adding authentication to protect the API
2. Using HTTPS to encrypt communications
3. Implementing rate limiting to prevent abuse
4. Restricting access to trusted networks

These security measures can be implemented using standard Go middleware or by placing the API server behind a reverse proxy like Nginx or Traefik.
