# Source Implementation - Scraper Package

## Adding a New Source

### 1. Create Scraper Class
```typescript
// src/sources/newsource.ts
export class NewSourceScraper implements MangaSource {
  constructor(private options: ScraperOptions = {}) {}
  
  @withRateLimit({ maxRequests: 10, perMilliseconds: 1000 })
  @withRetry({ maxRetries: 3 })
  async search(query: string): Promise<MangaSearchResult[]> {
    // Implementation
  }
  
  // Implement other methods...
}
```

### 2. Add to Factory
```typescript
// src/factory.ts
case 'newsource':
  return new NewSourceScraper(options)
```

### 3. Test Thoroughly
- Test search with various queries
- Test pagination
- Test error handling
- Test rate limiting

## Source-Specific Patterns

### HTML Parsing (Cheerio)
```typescript
const $ = cheerio.load(html)
const results = $('.manga-item').map((i, el) => ({
  id: $(el).attr('data-id'),
  title: $(el).find('.title').text(),
})).get()
```

### API Calls
```typescript
const response = await axios.get(url, {
  timeout: this.options.timeout,
  headers: { 'User-Agent': 'KazeNoManga/1.0' }
})
```
