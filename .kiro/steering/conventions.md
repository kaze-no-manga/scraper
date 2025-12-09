# Conventions - Scraper Package

## File Naming
- Source scrapers: `{source-name}.ts` (e.g., `mangapark.ts`)
- Decorators: `{feature}.ts` (e.g., `rate-limit.ts`)

## Class Naming
- Scrapers: `{SourceName}Scraper` (e.g., `MangaParkScraper`)
- Implement `MangaSource` interface

## Error Handling
- Throw typed errors: `ScraperError`, `RateLimitError`, `TimeoutError`
- Include source name in error messages
