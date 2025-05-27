# Crawler API

The Quillium-Crawler includes an API server that provides HTTP endpoints for controlling and monitoring the crawler. This document explains the available endpoints and how to use them.

## Overview

The API server provides a RESTful interface for:

- Starting and stopping crawlers
- Checking crawler status
- Retrieving crawled data
- Accessing metrics

## Configuration

The API server is automatically started when the crawler runs. By default, it listens on port 8080, but this can be configured in the code:

```go
// Start API server
if err := api.StartServer(":8080"); err != nil {
    log.Fatalf("Failed to start server: %v", err)
}
```

## Available Endpoints

### Status Endpoint

```
GET /status
```

Returns the current status of all crawler instances, including:

- Crawler ID
- Start URL
- Running status
- Pages crawled
- Errors encountered

#### Example Response

```json
{
  "crawlers": [
    {
      "id": "crawler_1",
      "start_url": "https://example.com",
      "running": true,
      "pages_crawled": 157,
      "errors": 12
    },
    {
      "id": "crawler_2",
      "start_url": "https://example2.com",
      "running": true,
      "pages_crawled": 83,
      "errors": 5
    }
  ]
}
```

### Start Crawler Endpoint

```
POST /crawler/{id}/start
```

Starts a crawler with the specified ID.

#### Example Request

```
POST /crawler/crawler_1/start
```

#### Example Response

```json
{
  "status": "success",
  "message": "Crawler crawler_1 started"
}
```

### Stop Crawler Endpoint

```
POST /crawler/{id}/stop
```

Stops a crawler with the specified ID.

#### Example Request

```
POST /crawler/crawler_1/stop
```

#### Example Response

```json
{
  "status": "success",
  "message": "Crawler crawler_1 stopped"
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
    // Register routes
    http.HandleFunc("/status", statusHandler)
    http.HandleFunc("/crawler/", crawlerHandler)

    // Start server
    log.Printf("Starting API server on %s", addr)
    return http.ListenAndServe(addr, nil)
}
```

### Route Handlers

#### Status Handler

The status handler retrieves the status of all crawler instances from the crawler manager:

```go
func statusHandler(w http.ResponseWriter, r *http.Request) {
    // Get status from crawler manager
    status := manager.GetStatus()

    // Return as JSON
    w.Header().Set("Content-Type", "application/json")
    json.NewEncoder(w).Encode(status)
}
```

#### Crawler Handler

The crawler handler processes requests to start and stop specific crawler instances:

```go
func crawlerHandler(w http.ResponseWriter, r *http.Request) {
    // Extract crawler ID from URL
    parts := strings.Split(r.URL.Path, "/")
    if len(parts) < 3 {
        http.Error(w, "Invalid request", http.StatusBadRequest)
        return
    }

    crawlerID := parts[2]
    action := ""
    if len(parts) >= 4 {
        action = parts[3]
    }

    // Process action
    switch action {
    case "start":
        if err := manager.StartCrawler(crawlerID); err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
        json.NewEncoder(w).Encode(map[string]string{
            "status":  "success",
            "message": fmt.Sprintf("Crawler %s started", crawlerID),
        })

    case "stop":
        if err := manager.StopCrawler(crawlerID); err != nil {
            http.Error(w, err.Error(), http.StatusInternalServerError)
            return
        }
        json.NewEncoder(w).Encode(map[string]string{
            "status":  "success",
            "message": fmt.Sprintf("Crawler %s stopped", crawlerID),
        })

    default:
        http.Error(w, "Invalid action", http.StatusBadRequest)
    }
}
```

## Using the API

### Checking Crawler Status

You can check the status of all crawlers using curl:

```bash
curl http://localhost:8080/status
```

### Starting a Crawler

To start a specific crawler:

```bash
curl -X POST http://localhost:8080/crawler/crawler_1/start
```

### Stopping a Crawler

To stop a specific crawler:

```bash
curl -X POST http://localhost:8080/crawler/crawler_1/stop
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
