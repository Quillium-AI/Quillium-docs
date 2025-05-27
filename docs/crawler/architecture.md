# Crawler Architecture

The Quillium-Crawler is built with a modular architecture that separates concerns and allows for flexible configuration. This document describes the main components and how they interact.

## High-Level Architecture

The crawler is organized into several key components:

```
Quillium-Crawler/
├── internal/
│   ├── api/              # API server and routes
│   ├── crawler/          # Crawling logic, config, anti-bot, proxies
│   ├── dedup/            # Deduplication (bloom filter)
│   ├── elasticsearch/    # Elasticsearch integration
│   └── metrics/          # Metrics and monitoring
├── main.go               # Application entry point
```

## Core Components

### Main Application (main.go)

The entry point of the application that:

1. Loads configuration from environment variables
2. Initializes the Elasticsearch client and creates the index if needed
3. Creates the crawler manager and individual crawler instances
4. Configures proxies if specified
5. Starts the API server

### Crawler Manager

The crawler manager coordinates multiple crawler instances, each responsible for crawling a specific start URL. It provides methods to:

- Add new crawler instances
- Start and stop crawlers
- Monitor crawler status

### Crawler Service

The crawler service is the core component responsible for:

- Navigating websites following links
- Extracting content from pages
- Applying URL filtering rules
- Implementing anti-bot measures
- Handling request delays and timeouts
- Deduplicating URLs using bloom filters

The crawler is built on top of the [Colly](https://github.com/gocolly/colly) framework, which provides efficient web scraping capabilities.

### Elasticsearch Service

The Elasticsearch service manages the storage and retrieval of crawled data:

- Initializes the Elasticsearch connection
- Creates and manages the index with appropriate mappings
- Stores page data with URL-based document IDs
- Handles document updates for revisited pages
- Provides methods to retrieve stored pages

### Deduplication Service

The deduplication service prevents crawling the same URL multiple times using a bloom filter:

- Efficiently checks if a URL has been seen before
- Optimizes memory usage with configurable filter size
- Calculates optimal bloom filter parameters based on expected URL count

### Metrics Service

The metrics service collects and exposes performance metrics:

- Pages crawled
- Request success/failure counts
- Content size statistics
- Error rates

Metrics are exposed in Prometheus-compatible format via an HTTP endpoint.

### API Server

The API server provides HTTP endpoints for controlling and monitoring the crawler:

- Start/stop crawling
- View crawler status
- Access metrics

## Data Flow

1. The crawler starts with one or more seed URLs
2. For each URL, it checks if it has been seen before using the deduplication service
3. If the URL is new, it sends an HTTP request to fetch the page
4. Anti-bot measures are applied to avoid detection
5. The page content is extracted and processed
6. The data is stored in Elasticsearch
7. Links are extracted from the page and added to the crawl queue
8. Metrics are updated
9. The process repeats until reaching configured limits (depth, max visits, etc.)

## Concurrency Model

The crawler uses Go's concurrency primitives to efficiently handle multiple requests in parallel:

- Each crawler instance runs in its own goroutine
- Multiple HTTP requests are made concurrently within configurable limits
- Mutex locks protect shared resources
- Context cancellation is used for graceful shutdown

This architecture allows the crawler to scale efficiently while maintaining control over resource usage and respecting website rate limits.
