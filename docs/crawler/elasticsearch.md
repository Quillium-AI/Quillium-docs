# Elasticsearch Integration

The Quillium-Crawler integrates with Elasticsearch to store and index crawled data. This document explains how the integration works and how to configure it.

## Overview

Elasticsearch is used as the primary storage backend for the crawler. It provides:

- Efficient storage of crawled page data
- Full-text search capabilities
- Scalability for large datasets
- Document versioning for updated content

## Configuration

Elasticsearch integration is configured through environment variables:

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

## Connection Initialization

The crawler initializes the Elasticsearch client during startup:

1. It creates a client using the provided addresses, username, and password
2. It implements a retry mechanism with exponential backoff to wait for Elasticsearch to be ready
3. It creates the index if it doesn't exist, with appropriate mappings

## Index Mapping

The crawler creates an Elasticsearch index with the following mapping:

```json
{
  "mappings": {
    "properties": {
      "url": { "type": "keyword" },
      "title": { "type": "text", "analyzer": "standard" },
      "snippet": { "type": "text", "analyzer": "standard" },
      "full_content": { "type": "text", "analyzer": "standard" },
      "created_at": { "type": "date" },
      "updated_at": { "type": "date" }
    }
  }
}
```

This mapping provides:

- Exact matching on URLs using the `keyword` type
- Full-text search on titles, snippets, and content
- Timestamp tracking for creation and updates

## Data Storage

The crawler stores the following data for each crawled page:

- **URL**: The full URL of the page
- **Title**: The page title extracted from the `<title>` tag or first `<h1>`
- **Snippet**: A short description from the meta description or first paragraph
- **Full Content**: (Optional) The complete HTML content of the page
- **Created At**: Timestamp when the page was first crawled
- **Updated At**: Timestamp when the page was last updated

## Document IDs

The crawler uses MD5 hashes of URLs as document IDs. This provides:

- Consistent IDs for the same URL
- Avoidance of special character issues in URLs
- Efficient document lookups

## Update Handling

When a page is crawled multiple times, the crawler:

1. Checks if the document already exists using the URL hash
2. If it exists, preserves the original `created_at` timestamp
3. Updates the `updated_at` timestamp
4. Merges the new content with existing data

This approach maintains a history of when pages were first discovered while keeping content up to date.

## Implementation Details

### ESStorage Struct

The `ESStorage` struct in the `elasticsearch` package provides the interface between the crawler and Elasticsearch:

```go
type ESStorage struct {
    es    *elasticsearch.Client
    index string
}
```

### Key Methods

- `NewESStorage`: Creates a new storage instance with the given client and index name
- `InitializeIndex`: Creates the index with appropriate mappings if it doesn't exist
- `SavePage`: Stores a page in Elasticsearch, handling new documents and updates
- `GetPage`: Retrieves a page by URL
- `StorePageData`: Internal method that handles the details of storing and updating documents

## Using the Stored Data

Once data is stored in Elasticsearch, you can:

1. Use the Elasticsearch API to search and retrieve crawled pages
2. Build custom applications that query the data
3. Use Kibana for visualization and exploration
4. Implement advanced text analysis and processing

### Example Queries

#### Search for Pages Containing a Term

```json
GET /crawled_data/_search
{
  "query": {
    "multi_match": {
      "query": "search term",
      "fields": ["title", "snippet", "full_content"]
    }
  }
}
```

#### Get Recently Crawled Pages

```json
GET /crawled_data/_search
{
  "sort": [
    { "updated_at": { "order": "desc" }}
  ],
  "size": 10
}
```

## Performance Considerations

- **Full Content Storage**: Storing full HTML content increases storage requirements but enables more comprehensive search
- **Index Refresh**: The crawler uses `refresh=true` to make documents immediately searchable
- **Bulk Operations**: For high-volume crawling, consider implementing bulk indexing
- **Scaling**: For large datasets, configure Elasticsearch with appropriate resources and sharding
