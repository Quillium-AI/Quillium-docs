# Quillium Crawler

Quillium-Crawler is a high-performance, extensible web crawler written in Go. It supports advanced anti-bot evasion, flexible domain/URL filtering, metrics, and Elasticsearch integration.

## Overview

The crawler is designed to efficiently navigate websites, extract content, and store it in Elasticsearch for further processing and analysis. It features a modular architecture with several key components that handle different aspects of the crawling process.

## Key Features

- **High Performance**: Optimized for speed and efficiency with parallel request handling
- **Configurable**: Extensive environment variable configuration options
- **Anti-Bot Measures**: Sophisticated evasion techniques to avoid being blocked
- **Flexible Filtering**: Domain and URL pattern filtering
- **Deduplication**: Efficient bloom filter-based URL deduplication
- **Elasticsearch Integration**: Seamless storage and indexing of crawled content
- **Metrics**: Prometheus-compatible metrics for monitoring
- **Proxy Support**: Rotate through multiple proxies to avoid IP-based blocking

## Architecture

The crawler is built around several core components:

- **Crawler Manager**: Coordinates multiple crawler instances
- **Crawler Service**: Handles the crawling logic and website navigation
- **Elasticsearch Service**: Manages data storage and retrieval
- **Deduplication Service**: Prevents crawling the same URL multiple times
- **Metrics Service**: Collects and exposes performance metrics
- **API Server**: Provides HTTP endpoints for control and monitoring

Each component is described in detail in the following sections.
