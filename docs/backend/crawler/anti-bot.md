# Anti-Bot Measures

The Quillium-Crawler includes sophisticated anti-bot measures to avoid detection and blocking by websites. This document explains the available techniques and how to configure them.

## Overview

Many websites implement bot detection systems to prevent automated crawling. These systems look for patterns that distinguish bots from human users, such as:

- Consistent request patterns and timing
- Missing or unchanging HTTP headers
- Lack of cookie handling
- Unusual user agent strings

The Quillium-Crawler implements several techniques to mimic human browsing behavior and avoid these detection mechanisms.

## Available Anti-Bot Techniques

### User Agent Rotation

User agent strings identify the browser and operating system making a request. The crawler can rotate through a list of common user agents to appear as different browsers.

```
CRAWLER_ENABLE_USER_AGENT_ROTATION=true
CRAWLER_CUSTOM_USER_AGENTS=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/91.0.4472.124 Safari/537.36,Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/605.1.15 (KHTML, like Gecko) Version/14.1.1 Safari/605.1.15
```

When enabled, the crawler will:

- Use a different user agent for each request
- Rotate through a list of common browser user agents
- Include any custom user agents specified in the configuration

### Header Randomization

HTTP headers provide information about the request and the client. The crawler can randomize these headers to appear more like a real browser.

```
CRAWLER_ENABLE_HEADER_RANDOMIZATION=true
```

When enabled, the crawler will randomize headers including:

- `Accept-Language`
- `Accept`
- `Accept-Encoding`
- `Referer` (based on the previous page)

### Cookie Handling

Websites often use cookies to track users and detect bots. The crawler can handle cookies like a real browser.

```
CRAWLER_ENABLE_COOKIE_HANDLING=true
```

When enabled, the crawler will:

- Store cookies received from websites
- Send appropriate cookies with subsequent requests
- Handle cookie expiration and domain restrictions

### Sophisticated Delays

Bots often make requests at regular intervals, which is easy to detect. The crawler can use more human-like timing patterns.

```
CRAWLER_ENABLE_SOPHISTICATED_DELAYS=true
CRAWLER_RANDOM_DELAY_FACTOR=1.5
```

When enabled, the crawler will:

- Vary the delay between requests
- Use a non-uniform distribution for more realistic timing
- Apply different delays for different domains

The `CRAWLER_RANDOM_DELAY_FACTOR` controls the variability of delays. Higher values create more randomness.

### Accept-Language Customization

The `Accept-Language` header tells websites which languages the user prefers. The crawler can customize this header.

```
CRAWLER_CUSTOM_ACCEPT_LANGUAGES=en-US,en;q=0.9,fr;q=0.8
```

When specified, the crawler will rotate through these language preferences.

## Proxy Rotation

Websites can block IP addresses that make too many requests. The crawler can rotate through a list of proxies to distribute requests across different IP addresses.

```
CRAWLER_PROXIES=http://user1:pass1@proxy1.example.com:8080,http://user2:pass2@proxy2.example.com:8080
```

When configured, the crawler will:

- Distribute requests across all specified proxies
- Handle proxy authentication
- Continue working if some proxies fail

## Implementation Details

The anti-bot measures are implemented in the `ApplyAntiBotMeasures` function in the crawler package. This function configures the Colly collector with appropriate callbacks and extensions based on the provided configuration.

```go
func ApplyAntiBotMeasures(collector *colly.Collector, config *AntiBotConfig) error {
    // Implementation details...
}
```

The function applies the selected anti-bot measures based on the configuration flags, creating a more human-like browsing pattern.

## Best Practices

1. **Start Conservative**: Enable only the measures you need to avoid detection
2. **Respect Rate Limits**: Even with anti-bot measures, respect website rate limits
3. **Monitor Blocks**: Watch for signs that your crawler is being blocked and adjust accordingly
4. **Combine Techniques**: Use multiple anti-bot measures together for best results
5. **Test Different Settings**: Different websites may require different anti-bot configurations

## Ethical Considerations

While these anti-bot measures can help avoid detection, they should be used responsibly:

- Respect websites' terms of service
- Consider using the `CRAWLER_RESPECT_ROBOTS_TXT=true` setting
- Avoid excessive requests that could impact website performance
- Only crawl publicly available content that you have permission to access
