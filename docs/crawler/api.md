# Crawler API

The Quillium-Crawler includes an API server that provides HTTP endpoints for health checks, version information, and metrics. This document explains the available endpoints and how to use them.

## Overview

The API server provides a RESTful interface for:

- Health and readiness checks
- Version information
- Prometheus metrics

## Configuration

The API server is automatically started when the crawler runs. By default, it listens on port 8080.

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

## API Server

The API server is automatically started when the crawler runs and listens on port 8080 by default. It provides several endpoints for monitoring and health checks that are particularly useful in containerized environments like Kubernetes.

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
