# Crawler Metrics

The Quillium-Crawler includes a metrics system that provides insights into the crawler's performance and behavior. This document explains the available metrics and how to use them.

## Overview

The metrics system is built on Prometheus, a popular open-source monitoring solution. It provides:

- Real-time performance monitoring
- Operational insights
- Troubleshooting capabilities
- Historical data for analysis

## Enabling Metrics

Metrics collection is enabled through an environment variable:

```
CRAWLER_ENABLE_METRICS=true
```

When enabled, the crawler exposes metrics on the `/metrics` endpoint of the API server.

## Available Metrics

The crawler collects several key metrics:

### Pages Crawled

```go
PagesCrawled = promauto.NewCounter(prometheus.CounterOpts{
    Name: "crawler_pages_crawled_total",
    Help: "The total number of pages crawled",
})
```

This counter tracks the total number of pages successfully crawled and processed.

### Requests Total

```go
RequestsTotal = promauto.NewCounter(prometheus.CounterOpts{
    Name: "crawler_requests_total",
    Help: "The total number of HTTP requests made",
})
```

This counter tracks all HTTP requests made by the crawler, regardless of outcome.

### Request Errors

```go
RequestErrors = promauto.NewCounter(prometheus.CounterOpts{
    Name: "crawler_request_errors_total",
    Help: "The total number of HTTP request errors",
})
```

This counter tracks failed HTTP requests, including timeouts, connection errors, and HTTP error status codes.

### Requests by Status

```go
RequestsByStatus = promauto.NewCounterVec(
    prometheus.CounterOpts{
        Name: "crawler_requests_by_status_total",
        Help: "The total number of HTTP requests by status code",
    },
    []string{"status"},
)
```

This counter vector breaks down requests by HTTP status code, allowing you to track successful (2xx), redirect (3xx), client error (4xx), and server error (5xx) responses separately.

### Content Size

```go
ContentSize = promauto.NewHistogram(prometheus.HistogramOpts{
    Name:    "crawler_content_size_bytes",
    Help:    "The size of crawled page content in bytes",
    Buckets: prometheus.ExponentialBuckets(1024, 2, 10), // 1KB to 1MB
})
```

This histogram tracks the distribution of page content sizes, providing insights into the amount of data being processed.

## Accessing Metrics

Metrics are exposed via an HTTP endpoint that can be scraped by Prometheus or other compatible monitoring systems.

### Endpoint

The metrics are available at:

```
http://localhost:8080/metrics
```

Where `8080` is the port the API server is running on.

### Example Output

```
# HELP crawler_pages_crawled_total The total number of pages crawled
# TYPE crawler_pages_crawled_total counter
crawler_pages_crawled_total 1523

# HELP crawler_requests_total The total number of HTTP requests made
# TYPE crawler_requests_total counter
crawler_requests_total 1842

# HELP crawler_request_errors_total The total number of HTTP request errors
# TYPE crawler_request_errors_total counter
crawler_request_errors_total 319

# HELP crawler_requests_by_status_total The total number of HTTP requests by status code
# TYPE crawler_requests_by_status_total counter
crawler_requests_by_status_total{status="200"} 1523
crawler_requests_by_status_total{status="404"} 187
crawler_requests_by_status_total{status="500"} 132

# HELP crawler_content_size_bytes The size of crawled page content in bytes
# TYPE crawler_content_size_bytes histogram
crawler_content_size_bytes_bucket{le="1024"} 12
crawler_content_size_bytes_bucket{le="2048"} 156
crawler_content_size_bytes_bucket{le="4096"} 423
crawler_content_size_bytes_bucket{le="8192"} 892
crawler_content_size_bytes_bucket{le="16384"} 1245
crawler_content_size_bytes_bucket{le="32768"} 1478
crawler_content_size_bytes_bucket{le="65536"} 1512
crawler_content_size_bytes_bucket{le="131072"} 1520
crawler_content_size_bytes_bucket{le="262144"} 1522
crawler_content_size_bytes_bucket{le="524288"} 1523
crawler_content_size_bytes_bucket{le="1048576"} 1523
crawler_content_size_bytes_bucket{le="+Inf"} 1523
crawler_content_size_bytes_sum 9876543
crawler_content_size_bytes_count 1523
```

## Integration with Prometheus

### Prometheus Configuration

To scrape metrics from the crawler, add the following to your Prometheus configuration:

```yaml
scrape_configs:
  - job_name: 'quillium-crawler'
    scrape_interval: 15s
    static_configs:
      - targets: ['localhost:8080']
```

Adjust the target address to match your crawler's API server address.

### Grafana Dashboard

You can create a Grafana dashboard to visualize the crawler metrics. Here's a simple example dashboard JSON that you can import into Grafana:

```json
{
  "annotations": {
    "list": []
  },
  "editable": true,
  "fiscalYearStartMonth": 0,
  "graphTooltip": 0,
  "links": [],
  "liveNow": false,
  "panels": [
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 10,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 0,
        "y": 0
      },
      "id": 1,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "rate(crawler_pages_crawled_total[5m])",
          "refId": "A"
        }
      ],
      "title": "Pages Crawled Rate",
      "type": "timeseries"
    },
    {
      "datasource": {
        "type": "prometheus",
        "uid": "prometheus"
      },
      "fieldConfig": {
        "defaults": {
          "color": {
            "mode": "palette-classic"
          },
          "custom": {
            "axisCenteredZero": false,
            "axisColorMode": "text",
            "axisLabel": "",
            "axisPlacement": "auto",
            "barAlignment": 0,
            "drawStyle": "line",
            "fillOpacity": 10,
            "gradientMode": "none",
            "hideFrom": {
              "legend": false,
              "tooltip": false,
              "viz": false
            },
            "lineInterpolation": "linear",
            "lineWidth": 1,
            "pointSize": 5,
            "scaleDistribution": {
              "type": "linear"
            },
            "showPoints": "never",
            "spanNulls": false,
            "stacking": {
              "group": "A",
              "mode": "none"
            },
            "thresholdsStyle": {
              "mode": "off"
            }
          },
          "mappings": [],
          "thresholds": {
            "mode": "absolute",
            "steps": [
              {
                "color": "green",
                "value": null
              }
            ]
          },
          "unit": "short"
        },
        "overrides": []
      },
      "gridPos": {
        "h": 8,
        "w": 12,
        "x": 12,
        "y": 0
      },
      "id": 2,
      "options": {
        "legend": {
          "calcs": [],
          "displayMode": "list",
          "placement": "bottom",
          "showLegend": true
        },
        "tooltip": {
          "mode": "single",
          "sort": "none"
        }
      },
      "targets": [
        {
          "datasource": {
            "type": "prometheus",
            "uid": "prometheus"
          },
          "expr": "rate(crawler_request_errors_total[5m]) / rate(crawler_requests_total[5m])",
          "refId": "A"
        }
      ],
      "title": "Error Rate",
      "type": "timeseries"
    }
  ],
  "refresh": "5s",
  "schemaVersion": 38,
  "style": "dark",
  "tags": [],
  "templating": {
    "list": []
  },
  "time": {
    "from": "now-1h",
    "to": "now"
  },
  "timepicker": {},
  "timezone": "",
  "title": "Quillium Crawler Dashboard",
  "uid": "quillium-crawler",
  "version": 1,
  "weekStart": ""
}
```

## Using Metrics for Monitoring

### Key Performance Indicators

1. **Crawl Rate**: Monitor `rate(crawler_pages_crawled_total[5m])` to track how many pages are being crawled per minute
2. **Error Rate**: Calculate `rate(crawler_request_errors_total[5m]) / rate(crawler_requests_total[5m])` to monitor the percentage of failed requests
3. **Status Distribution**: Track the distribution of HTTP status codes to identify patterns of client or server errors
4. **Content Size**: Monitor the distribution of page sizes to understand the data volume being processed

### Alerting

You can set up alerts in Prometheus or Grafana based on these metrics. For example:

- Alert when the error rate exceeds 20% for more than 5 minutes
- Alert when the crawl rate drops below a certain threshold
- Alert when no new pages have been crawled for an extended period

## Implementation Details

The metrics system is implemented in the `metrics` package:

```go
func Initialize(enableFullContent bool) {
    // Initialize metrics only once
    once.Do(func() {
        // Register metrics
        PagesCrawled = promauto.NewCounter(prometheus.CounterOpts{
            Name: "crawler_pages_crawled_total",
            Help: "The total number of pages crawled",
        })

        RequestsTotal = promauto.NewCounter(prometheus.CounterOpts{
            Name: "crawler_requests_total",
            Help: "The total number of HTTP requests made",
        })

        RequestErrors = promauto.NewCounter(prometheus.CounterOpts{
            Name: "crawler_request_errors_total",
            Help: "The total number of HTTP request errors",
        })

        RequestsByStatus = promauto.NewCounterVec(
            prometheus.CounterOpts{
                Name: "crawler_requests_by_status_total",
                Help: "The total number of HTTP requests by status code",
            },
            []string{"status"},
        )

        // Only initialize content size histogram if full content is enabled
        if enableFullContent {
            ContentSize = promauto.NewHistogram(prometheus.HistogramOpts{
                Name:    "crawler_content_size_bytes",
                Help:    "The size of crawled page content in bytes",
                Buckets: prometheus.ExponentialBuckets(1024, 2, 10), // 1KB to 1MB
            })
        }

        initialized = true
    })
}
```

The metrics are updated in the crawler's callbacks:

```go
// Increment metrics counter
if c.Options.EnableMetrics {
    metrics.PagesCrawled.Inc()
}

// Track content size for metrics
if c.Options.EnableMetrics {
    metrics.ContentSize.Observe(float64(len(e.Response.Body)))
}
```
