# URL Deduplication

The Quillium-Crawler implements an efficient URL deduplication system to avoid crawling the same pages multiple times. This document explains how the deduplication works and how it's configured.

## Overview

Deduplication is essential for web crawlers to:

- Prevent infinite loops in cyclical link structures
- Avoid wasting resources on already visited pages
- Reduce load on target websites
- Improve crawling efficiency

The Quillium-Crawler uses a bloom filter for efficient URL deduplication.

## Bloom Filter Implementation

A bloom filter is a space-efficient probabilistic data structure used to test whether an element is a member of a set. It's particularly well-suited for URL deduplication because:

- It has constant-time lookups regardless of the number of elements
- It uses significantly less memory than storing the complete URL list
- It never produces false negatives (though it can produce false positives)

### How It Works

1. When a URL is discovered, the crawler checks if it's in the bloom filter
2. If it's not in the filter, the URL is added to the crawl queue and marked as seen
3. If it's already in the filter, the URL is skipped

## Configuration

The bloom filter is automatically configured based on the expected number of URLs to be crawled:

```go
// Calculate optimal bloom filter size based on expected visits
expectedURLs := 10000 // Default minimum size
if options.MaxVisits != nil && *options.MaxVisits > 0 {
    expectedURLs = *options.MaxVisits * 10 // Estimate 10x the max visits as potential URLs
}

// Create bloom filter with 1% false positive rate
bloomSize := dedup.CalculateOptimalSize(expectedURLs, 0.01)
hashFuncs := dedup.CalculateOptimalHashFunctions(bloomSize, expectedURLs)
```

The crawler automatically:

- Sets a minimum expected URL count of 10,000
- Estimates the total number of URLs as 10 times the maximum visit count
- Configures the bloom filter for a 1% false positive rate
- Calculates the optimal filter size and number of hash functions

## Implementation Details

### Bloom Filter Structure

The bloom filter is implemented in the `dedup` package:

```go
type BloomFilter struct {
    bits       []uint64
    size       uint64
    hashFuncs  uint64
    setBits    uint64
    mutex      sync.RWMutex
}
```

### Key Methods

- `NewBloomFilter`: Creates a new bloom filter with the specified size and hash function count
- `Add`: Adds a URL to the filter
- `Contains`: Checks if a URL is in the filter
- `CalculateOptimalSize`: Determines the optimal filter size based on expected elements and false positive rate
- `CalculateOptimalHashFunctions`: Calculates the optimal number of hash functions

### Integration with Crawler

The bloom filter is integrated into the crawler's link handling process:

```go
func (c *Crawler) handleLink(e *colly.HTMLElement) {
    link := e.Attr("href")
    absURL := e.Request.AbsoluteURL(link)

    // Skip if URL doesn't match allowed patterns
    if !c.isAllowedURL(absURL) {
        return
    }

    // Skip URLs we've already seen (deduplication using bloom filter)
    if c.urlFilter.Contains(absURL) {
        return
    }

    // Mark URL as seen before visiting
    c.urlFilter.Add(absURL)

    c.Collector.Visit(absURL)
}
```

## Performance Considerations

### Memory Usage

The bloom filter is very memory-efficient. For example, with default settings:

- A filter for 100,000 URLs uses approximately 120 KB of memory
- A filter for 1,000,000 URLs uses approximately 1.2 MB of memory

This is significantly less than storing the complete list of URLs.

### False Positives

Bloom filters can produce false positives (incorrectly indicating a URL has been seen). The crawler configures the filter for a 1% false positive rate, which means:

- About 1 in 100 new URLs might be incorrectly identified as duplicates
- This is an acceptable trade-off for the memory efficiency gained
- The false positive rate increases as the filter fills up

### No Removal Support

Standard bloom filters don't support element removal. This is acceptable for the crawler because:

- URLs are only added, never removed during a crawl session
- The filter is recreated for each new crawl session

## Best Practices

1. **Set Appropriate Max Visits**: Configure `CRAWLER_MAX_VISITS` based on your expected crawl size to optimize the bloom filter
2. **Monitor Memory Usage**: For very large crawls, be aware of the memory used by the bloom filter
3. **Consider False Positives**: If complete coverage is critical, be aware of the small false positive rate
4. **Restart for Fresh Crawls**: For completely fresh crawls of the same site, restart the crawler to reset the bloom filter
