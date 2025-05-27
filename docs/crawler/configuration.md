# Crawler Configuration

The Quillium-Crawler is highly configurable through environment variables. This document provides a detailed explanation of all available configuration options.

## Environment Variables

Configuration is managed through environment variables, which can be set in a `.env` file or directly in the environment. The crawler includes a `.env.example` file that you can copy and modify.

```bash
cp .env.example .env
# Edit .env with your preferred settings
```

## Core Configuration

### Start URLs

```
CRAWLER_START_URL=https://example.com
CRAWLER_START_URLS=https://example.com,https://example2.com
```

- `CRAWLER_START_URL`: Single start URL (used if `CRAWLER_START_URLS` is not set)
- `CRAWLER_START_URLS`: Comma-separated list of start URLs (each will spawn a separate crawler instance)

At least one of these must be set. If both are set, `CRAWLER_START_URLS` takes precedence.

### Depth and Limits

```
CRAWLER_MAX_DEPTH=3
CRAWLER_MAX_VISITS=1000
```

- `CRAWLER_MAX_DEPTH`: Maximum crawl depth from the start URL
- `CRAWLER_MAX_VISITS`: Maximum number of pages to visit per crawler instance

### Parallelism and Delays

```
CRAWLER_PARALLEL_REQUESTS=10
CRAWLER_DELAY_MS=50
CRAWLER_RANDOM_DELAY_MS=50
CRAWLER_TIMEOUT_SEC=10
```

- `CRAWLER_PARALLEL_REQUESTS`: Number of parallel requests per crawler
- `CRAWLER_DELAY_MS`: Fixed delay between requests in milliseconds
- `CRAWLER_RANDOM_DELAY_MS`: Random delay added to each request in milliseconds
- `CRAWLER_TIMEOUT_SEC`: Request timeout in seconds

## Domain and URL Filtering

```
CRAWLER_ALLOWED_DOMAINS=example.com,example2.com
CRAWLER_DISALLOWED_DOMAINS=
CRAWLER_ALLOWED_URLS=
CRAWLER_DISALLOWED_URLS=
CRAWLER_IGNORE_QUERY_STRINGS=false
CRAWLER_RESPECT_ROBOTS_TXT=true
```

- `CRAWLER_ALLOWED_DOMAINS`: Only crawl these domains (comma-separated, empty for all)
- `CRAWLER_DISALLOWED_DOMAINS`: Domains to skip (comma-separated)
- `CRAWLER_ALLOWED_URLS`: Only crawl URLs containing these patterns (comma-separated)
- `CRAWLER_DISALLOWED_URLS`: Skip URLs containing these patterns (comma-separated)
- `CRAWLER_IGNORE_QUERY_STRINGS`: Whether to ignore query strings in URLs (true/false)
- `CRAWLER_RESPECT_ROBOTS_TXT`: Whether to respect robots.txt rules (true/false)

## Elasticsearch Configuration

```
CRAWLER_ELASTICSEARCH_ADDRESSES=http://elasticsearch:9200
CRAWLER_ELASTICSEARCH_USERNAME=elastic
CRAWLER_ELASTICSEARCH_PASSWORD=changeme
CRAWLER_INDEX_NAME=crawled_data
CRAWLER_ENABLE_FULL_CONTENT=false
```

- `CRAWLER_ELASTICSEARCH_ADDRESSES`: Comma-separated list of Elasticsearch endpoints
- `CRAWLER_ELASTICSEARCH_USERNAME`: Elasticsearch username for authentication
- `CRAWLER_ELASTICSEARCH_PASSWORD`: Elasticsearch password for authentication
- `CRAWLER_INDEX_NAME`: Name of the Elasticsearch index to store crawled data
- `CRAWLER_ENABLE_FULL_CONTENT`: Whether to store the full HTML content of pages (true/false)

## Metrics

```
CRAWLER_ENABLE_METRICS=true
```

- `CRAWLER_ENABLE_METRICS`: Enable Prometheus metrics endpoint (true/false)

## Proxy Configuration

```
CRAWLER_PROXIES=http://user:pass@proxyserver:8080
```

- `CRAWLER_PROXIES`: Comma-separated list of proxies to rotate through

Proxies should be specified in the format `protocol://[username:password@]host:port`

## Anti-Bot Measures

```
CRAWLER_ENABLE_USER_AGENT_ROTATION=true
CRAWLER_ENABLE_HEADER_RANDOMIZATION=true
CRAWLER_ENABLE_COOKIE_HANDLING=true
CRAWLER_ENABLE_SOPHISTICATED_DELAYS=true
CRAWLER_RANDOM_DELAY_FACTOR=1.5
CRAWLER_CUSTOM_USER_AGENTS=
CRAWLER_CUSTOM_ACCEPT_LANGUAGES=
```

- `CRAWLER_ENABLE_USER_AGENT_ROTATION`: Enable random user agent rotation
- `CRAWLER_ENABLE_HEADER_RANDOMIZATION`: Enable HTTP header randomization
- `CRAWLER_ENABLE_COOKIE_HANDLING`: Enable browser-like cookie handling
- `CRAWLER_ENABLE_SOPHISTICATED_DELAYS`: Enable more human-like delays
- `CRAWLER_RANDOM_DELAY_FACTOR`: Factor for random delay calculation (0.5-2.0 recommended)
- `CRAWLER_CUSTOM_USER_AGENTS`: Additional custom user agents (comma-separated)
- `CRAWLER_CUSTOM_ACCEPT_LANGUAGES`: Additional custom accept-language values (comma-separated)

## Configuration Precedence

The crawler loads configuration in the following order of precedence:

1. Environment variables
2. `.env` file
3. Default values

This means that environment variables will override values in the `.env` file, and any unspecified values will use defaults.
