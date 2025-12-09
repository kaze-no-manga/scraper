# @kaze/scraper

> Multi-source manga scraper with unified interface for Kaze no Manga

## Overview

This package provides stateless scrapers for multiple manga sources (MangaPark, OmegaScans, etc.) with a common interface, rate limiting, retry logic, and error handling.

## Features

- 🔌 **Unified Interface**: Common API across all sources
- 🔄 **Stateless**: No internal state, pure functions
- ⏱️ **Rate Limiting**: Configurable per source
- 🔁 **Retry Logic**: Automatic retry with exponential backoff
- ⏰ **Timeout Handling**: Configurable timeout per source
- 🛡️ **Error Handling**: Graceful degradation
- 📦 **Extensible**: Easy to add new sources

## Installation

```bash
npm install @kaze/scraper
```

## Usage

### Basic Usage

```typescript
import { MangaParkScraper, OmegaScansScraper } from '@kaze/scraper'

// Initialize scraper
const scraper = new MangaParkScraper()

// Search manga
const results = await scraper.search('one piece')
console.log(results) // Array of Manga

// Get manga details
const manga = await scraper.getManga('manga-id')
console.log(manga) // MangaDetail

// Get chapters
const chapters = await scraper.getChapters('manga-id')
console.log(chapters) // Array of Chapter

// Get chapter images
const images = await scraper.getChapterImages('chapter-id')
console.log(images) // Array of image URLs
```

### With Configuration

```typescript
import { MangaParkScraper } from '@kaze/scraper'

const scraper = new MangaParkScraper({
  timeout: 10000,           // 10 seconds
  maxRetries: 3,
  rateLimit: {
    maxRequests: 10,
    perMilliseconds: 1000   // 10 requests per second
  }
})
```

### Using the Factory

```typescript
import { createScraper } from '@kaze/scraper'

const scraper = createScraper('mangapark', {
  timeout: 5000,
  maxRetries: 2
})

const results = await scraper.search('naruto')
```

## Common Interface

All scrapers implement the `MangaSource` interface:

```typescript
interface MangaSource {
  /**
   * Search manga by query
   */
  search(query: string): Promise<MangaSearchResult[]>
  
  /**
   * Get manga details by ID
   */
  getManga(id: string): Promise<MangaDetail>
  
  /**
   * Get all chapters for a manga
   */
  getChapters(mangaId: string): Promise<Chapter[]>
  
  /**
   * Get image URLs for a chapter
   */
  getChapterImages(chapterId: string): Promise<string[]>
}
```

### Types

```typescript
interface MangaSearchResult {
  id: string                    // Source-specific ID
  title: string
  coverImage?: string
  latestChapter?: number
  status?: 'ongoing' | 'completed'
}

interface MangaDetail {
  id: string
  title: string
  altTitles: string[]
  description?: string
  coverImage?: string
  status: 'ongoing' | 'completed' | 'hiatus' | 'cancelled'
  genres: string[]
  authors: string[]
  year?: number
  totalChapters?: number
}

interface Chapter {
  id: string                    // Source-specific ID
  number: number                // Supports decimals (5.5)
  title?: string
  releaseDate?: Date
  url: string
}
```

## Supported Sources

### MangaPark

```typescript
import { MangaParkScraper } from '@kaze/scraper'

const scraper = new MangaParkScraper()
```

**Features:**
- Search by title
- Manga details with metadata
- Chapter list with release dates
- High-quality images

**Rate Limit:** 10 requests/second (default)

### OmegaScans

```typescript
import { OmegaScansScraper } from '@kaze/scraper'

const scraper = new OmegaScansScraper()
```

**Features:**
- Search by title
- Manga details
- Chapter list
- Image URLs

**Rate Limit:** 5 requests/second (default)

## Configuration

### Scraper Options

```typescript
interface ScraperOptions {
  timeout?: number              // Request timeout in ms (default: 10000)
  maxRetries?: number           // Max retry attempts (default: 3)
  rateLimit?: {
    maxRequests: number         // Max requests
    perMilliseconds: number     // Time window
  }
  headers?: Record<string, string>  // Custom headers
}
```

### Rate Limiting

Rate limiting is handled automatically using a decorator:

```typescript
import { withRateLimit } from '@kaze/scraper/decorators'

class MyScraper implements MangaSource {
  @withRateLimit({ maxRequests: 10, perMilliseconds: 1000 })
  async search(query: string) {
    // Implementation
  }
}
```

### Retry Logic

Retry logic with exponential backoff:

```typescript
import { withRetry } from '@kaze/scraper/decorators'

class MyScraper implements MangaSource {
  @withRetry({ maxRetries: 3, backoff: 'exponential' })
  async getManga(id: string) {
    // Implementation
  }
}
```

## Error Handling

All scrapers throw typed errors:

```typescript
import { ScraperError, RateLimitError, TimeoutError } from '@kaze/scraper'

try {
  const results = await scraper.search('one piece')
} catch (error) {
  if (error instanceof RateLimitError) {
    console.error('Rate limit exceeded, retry after:', error.retryAfter)
  } else if (error instanceof TimeoutError) {
    console.error('Request timed out')
  } else if (error instanceof ScraperError) {
    console.error('Scraper error:', error.message)
  }
}
```

## Adding a New Source

1. Create a new file in `src/sources/`:

```typescript
// src/sources/newsource.ts
import { MangaSource, MangaSearchResult, MangaDetail, Chapter } from '../types'
import { withRateLimit, withRetry } from '../decorators'

export class NewSourceScraper implements MangaSource {
  constructor(private options: ScraperOptions = {}) {}
  
  @withRateLimit({ maxRequests: 10, perMilliseconds: 1000 })
  @withRetry({ maxRetries: 3 })
  async search(query: string): Promise<MangaSearchResult[]> {
    // Implementation
  }
  
  @withRetry({ maxRetries: 3 })
  async getManga(id: string): Promise<MangaDetail> {
    // Implementation
  }
  
  @withRetry({ maxRetries: 3 })
  async getChapters(mangaId: string): Promise<Chapter[]> {
    // Implementation
  }
  
  @withRetry({ maxRetries: 3 })
  async getChapterImages(chapterId: string): Promise<string[]> {
    // Implementation
  }
}
```

2. Export in `src/index.ts`:

```typescript
export { NewSourceScraper } from './sources/newsource'
```

3. Add to factory:

```typescript
// src/factory.ts
export function createScraper(source: string, options?: ScraperOptions): MangaSource {
  switch (source) {
    case 'mangapark':
      return new MangaParkScraper(options)
    case 'omegascans':
      return new OmegaScansScraper(options)
    case 'newsource':
      return new NewSourceScraper(options)
    default:
      throw new Error(`Unknown source: ${source}`)
  }
}
```

## Package Structure

```
scraper/
├── src/
│   ├── sources/
│   │   ├── mangapark.ts
│   │   ├── omegascans.ts
│   │   └── index.ts
│   ├── decorators/
│   │   ├── rate-limit.ts
│   │   ├── retry.ts
│   │   ├── timeout.ts
│   │   └── index.ts
│   ├── errors/
│   │   ├── scraper-error.ts
│   │   ├── rate-limit-error.ts
│   │   ├── timeout-error.ts
│   │   └── index.ts
│   ├── types/
│   │   └── index.ts
│   ├── factory.ts
│   └── index.ts
├── tests/
│   ├── sources/
│   │   ├── mangapark.test.ts
│   │   └── omegascans.test.ts
│   └── decorators/
│       ├── rate-limit.test.ts
│       └── retry.test.ts
├── package.json
└── README.md
```

## Development

```bash
# Install dependencies
npm install

# Build package
npm run build

# Run tests
npm test

# Run tests in watch mode
npm run test:watch

# Lint
npm run lint

# Publish (private)
npm publish
```

## Testing

```bash
# Run all tests
npm test

# Test specific source
npm test -- mangapark

# Test with coverage
npm run test:coverage
```

## Best Practices

1. **Always use decorators** for rate limiting and retry logic
2. **Handle errors gracefully** - don't let one source failure break the system
3. **Respect source rate limits** - be a good citizen
4. **Cache responses** when appropriate (handled by backend, not scraper)
5. **Log errors** for debugging and monitoring

## Versioning

This package follows [Semantic Versioning](https://semver.org/):

- **Major**: Breaking changes to interface or behavior
- **Minor**: New sources or features
- **Patch**: Bug fixes or improvements

## License

MIT License - see [LICENSE](LICENSE) for details.

---

**Part of the [Kaze no Manga](https://github.com/kaze-no-manga) project**
