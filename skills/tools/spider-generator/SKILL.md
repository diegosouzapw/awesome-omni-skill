---
name: spider-generator
description: Generate Scrapy spiders with best practices when creating new spiders, crawlers, or implementing scraping patterns. Automatically scaffolds spiders based on target website type and requirements.
allowed-tools: Read, Write, Grep
---

You are a Scrapy spider generation expert. You create well-structured, efficient spiders following Scrapy best practices with proper error handling, rate limiting, and data extraction patterns.

## Spider Types and When to Use Them

### 1. Basic Spider (scrapy.Spider)

**Use for**:
- Simple single-page scraping
- Static content extraction
- Following simple link patterns
- API endpoints returning HTML

**Template**:

```python
import scrapy
from typing import Iterator


class BasicSpider(scrapy.Spider):
    """
    Spider for scraping [DESCRIPTION].

    Usage:
        scrapy crawl basic_spider
        scrapy crawl basic_spider -a category=electronics
    """

    name = "basic_spider"
    allowed_domains = ["example.com"]
    start_urls = ["https://example.com/products"]

    # Custom settings
    custom_settings = {
        'DOWNLOAD_DELAY': 1,
        'CONCURRENT_REQUESTS_PER_DOMAIN': 2,
        'ROBOTSTXT_OBEY': True,
        'USER_AGENT': 'MyBot/1.0 (+http://www.example.com/bot)',
    }

    def __init__(self, category: str = None, *args, **kwargs):
        """Initialize spider with optional arguments."""
        super().__init__(*args, **kwargs)
        self.category = category
        if category:
            self.start_urls = [f"https://example.com/products/{category}"]

    def parse(self, response) -> Iterator[scrapy.Request | dict]:
        """
        Parse main page and extract data.

        @url https://example.com/products
        @returns items 10 100
        @returns requests 0 0
        """
        # Extract items
        for item in response.css('div.product'):
            yield {
                'title': item.css('h2.title::text').get(),
                'price': item.css('span.price::text').get(),
                'url': response.urljoin(item.css('a::attr(href)').get()),
                'image': item.css('img::attr(src)').get(),
                'description': item.css('p.desc::text').get(),
                'category': self.category,
            }

        # Follow pagination
        next_page = response.css('a.next::attr(href)').get()
        if next_page:
            yield response.follow(next_page, callback=self.parse)
```

### 2. CrawlSpider (Rules-Based)

**Use for**:
- Complex site navigation
- Multiple URL patterns
- Deep crawling with rules
- Hierarchical content structures

**Template**:

```python
import scrapy
from scrapy.spiders import CrawlSpider, Rule
from scrapy.linkextractors import LinkExtractor
from typing import Iterator


class MyCrawlSpider(CrawlSpider):
    """
    Crawl spider for [DESCRIPTION].

    This spider follows links based on defined rules and extracts
    data from matching pages.
    """

    name = "crawl_spider"
    allowed_domains = ["example.com"]
    start_urls = ["https://example.com"]

    # Custom settings
    custom_settings = {
        'DEPTH_LIMIT': 3,
        'DOWNLOAD_DELAY': 1,
        'CONCURRENT_REQUESTS': 16,
        'AUTOTHROTTLE_ENABLED': True,
        'AUTOTHROTTLE_START_DELAY': 1,
        'AUTOTHROTTLE_TARGET_CONCURRENCY': 2.0,
    }

    # Define crawling rules
    rules = (
        # Extract and follow category links
        Rule(
            LinkExtractor(restrict_css='nav.categories a'),
            callback='parse_category',
            follow=True
        ),

        # Extract product links, don't follow
        Rule(
            LinkExtractor(restrict_css='div.product a.detail'),
            callback='parse_product',
            follow=False
        ),

        # Follow pagination
        Rule(
            LinkExtractor(restrict_css='a.next-page'),
            follow=True
        ),
    )

    def parse_category(self, response) -> dict:
        """Parse category page."""
        return {
            'type': 'category',
            'name': response.css('h1::text').get(),
            'url': response.url,
            'product_count': len(response.css('div.product')),
        }

    def parse_product(self, response) -> dict:
        """Parse product detail page."""
        return {
            'type': 'product',
            'title': response.css('h1.title::text').get(),
            'price': response.css('span.price::text').get(),
            'sku': response.css('span.sku::text').get(),
            'description': response.css('div.description::text').getall(),
            'images': response.css('div.gallery img::attr(src)').getall(),
            'specs': {
                spec.css('dt::text').get(): spec.css('dd::text').get()
                for spec in response.css('dl.specs div')
            },
            'availability': response.css('span.stock::text').get(),
            'url': response.url,
        }
```

### 3. Playwright/Selenium Spider (JavaScript Rendering)

**Use for**:
- JavaScript-heavy websites
- Dynamic content loading
- AJAX requests
- Pages requiring interaction (clicks, scrolls)
- SPAs (Single Page Applications)

**Template**:

```python
import scrapy
from typing import Iterator
import json


class PlaywrightSpider(scrapy.Spider):
    """
    Spider using Playwright for JavaScript rendering.

    Requires: scrapy-playwright
    Install: pip install scrapy-playwright
    """

    name = "playwright_spider"
    allowed_domains = ["example.com"]

    custom_settings = {
        'DOWNLOAD_HANDLERS': {
            "http": "scrapy_playwright.handler.ScrapyPlaywrightDownloadHandler",
            "https": "scrapy_playwright.handler.ScrapyPlaywrightDownloadHandler",
        },
        'TWISTED_REACTOR': "twisted.internet.asyncioreactor.AsyncioSelectorReactor",
        'PLAYWRIGHT_BROWSER_TYPE': 'chromium',
        'PLAYWRIGHT_LAUNCH_OPTIONS': {
            'headless': True,
            'timeout': 60000,
        },
        'CONCURRENT_REQUESTS': 4,  # Lower for browser-based scraping
        'DOWNLOAD_DELAY': 2,
    }

    def start_requests(self) -> Iterator[scrapy.Request]:
        """Start requests with Playwright meta."""
        urls = [
            "https://example.com/dynamic-page",
        ]

        for url in urls:
            yield scrapy.Request(
                url,
                meta={
                    'playwright': True,
                    'playwright_include_page': True,
                    'playwright_page_methods': [
                        # Wait for element
                        ('wait_for_selector', 'div.content'),
                        # Scroll to bottom
                        ('evaluate', 'window.scrollTo(0, document.body.scrollHeight)'),
                        # Wait for network idle
                        ('wait_for_load_state', 'networkidle'),
                    ],
                },
                callback=self.parse,
                errback=self.errback_close_page,
            )

    async def parse(self, response) -> Iterator[dict]:
        """Parse page with Playwright."""
        page = response.meta["playwright_page"]

        try:
            # Wait for dynamic content
            await page.wait_for_selector('div.product', timeout=10000)

            # Optional: Click "Load More" button
            load_more = await page.query_selector('button.load-more')
            if load_more:
                await load_more.click()
                await page.wait_for_timeout(2000)

            # Get updated content
            content = await page.content()
            updated_response = response.replace(body=content.encode('utf-8'))

            # Extract data from updated response
            for product in updated_response.css('div.product'):
                yield {
                    'title': product.css('h2::text').get(),
                    'price': product.css('span.price::text').get(),
                    'url': product.css('a::attr(href)').get(),
                }

            # Optional: Extract data from JavaScript variables
            data = await page.evaluate('''() => {
                return window.__INITIAL_STATE__ || {};
            }''')

            if data:
                yield {'js_data': data}

        finally:
            await page.close()

    async def errback_close_page(self, failure):
        """Close page on error."""
        page = failure.request.meta.get("playwright_page")
        if page:
            await page.close()
        self.logger.error(f"Error processing {failure.request.url}: {failure}")
```

### 4. API Spider

**Use for**:
- REST APIs
- GraphQL endpoints
- JSON/XML responses
- Paginated API results

**Template**:

```python
import scrapy
import json
from typing import Iterator, Optional
from urllib.parse import urlencode


class APISpider(scrapy.Spider):
    """
    Spider for scraping data from API endpoints.
    """

    name = "api_spider"
    allowed_domains = ["api.example.com"]

    # API configuration
    api_base = "https://api.example.com/v1"
    api_key = None  # Set via spider argument or settings

    custom_settings = {
        'DOWNLOAD_DELAY': 0.5,
        'CONCURRENT_REQUESTS': 10,
        'RETRY_TIMES': 3,
        'HTTPERROR_ALLOWED_CODES': [400, 404],  # Handle specific errors
    }

    def __init__(self, api_key: Optional[str] = None, *args, **kwargs):
        """Initialize with API key."""
        super().__init__(*args, **kwargs)
        self.api_key = api_key or self.settings.get('API_KEY')

        if not self.api_key:
            raise ValueError("API key required. Pass via -a api_key=XXX")

    def start_requests(self) -> Iterator[scrapy.Request]:
        """Start API requests."""
        # Initial request
        yield self.make_api_request(
            endpoint="/products",
            params={'page': 1, 'limit': 100},
            callback=self.parse_products,
        )

    def make_api_request(
        self,
        endpoint: str,
        params: Optional[dict] = None,
        method: str = 'GET',
        callback=None,
        **kwargs
    ) -> scrapy.Request:
        """Create API request with authentication."""
        url = f"{self.api_base}{endpoint}"

        if params:
            url = f"{url}?{urlencode(params)}"

        headers = {
            'Authorization': f'Bearer {self.api_key}',
            'Accept': 'application/json',
            'User-Agent': 'MyBot/1.0',
        }

        return scrapy.Request(
            url,
            method=method,
            headers=headers,
            callback=callback or self.parse,
            errback=self.handle_error,
            **kwargs
        )

    def parse_products(self, response) -> Iterator[scrapy.Request | dict]:
        """Parse API response."""
        try:
            data = json.loads(response.text)
        except json.JSONDecodeError as e:
            self.logger.error(f"Invalid JSON: {e}")
            return

        # Extract items
        for item in data.get('results', []):
            yield {
                'id': item.get('id'),
                'name': item.get('name'),
                'price': item.get('price'),
                'category': item.get('category'),
                'url': f"https://example.com/products/{item.get('id')}",
            }

        # Handle pagination
        pagination = data.get('pagination', {})
        next_page = pagination.get('next_page')

        if next_page:
            yield self.make_api_request(
                endpoint="/products",
                params={'page': next_page, 'limit': 100},
                callback=self.parse_products,
            )

    def handle_error(self, failure):
        """Handle request errors."""
        self.logger.error(f"Request failed: {failure.request.url}")

        if failure.value.response:
            self.logger.error(f"Status: {failure.value.response.status}")
            self.logger.error(f"Body: {failure.value.response.text}")
```

## Best Practices for Spider Development

### 1. Error Handling

```python
def parse(self, response):
    """Parse with comprehensive error handling."""
    try:
        # Check response status
        if response.status != 200:
            self.logger.warning(f"Non-200 status: {response.status}")
            return

        # Validate expected content
        if not response.css('div.product'):
            self.logger.warning(f"No products found on {response.url}")
            return

        for product in response.css('div.product'):
            # Safe extraction with defaults
            item = {
                'title': product.css('h2::text').get('Unknown'),
                'price': self.parse_price(product.css('span.price::text').get()),
                'url': response.urljoin(product.css('a::attr(href)').get()),
            }

            # Validate required fields
            if item['title'] != 'Unknown' and item['url']:
                yield item
            else:
                self.logger.warning(f"Incomplete item: {item}")

    except Exception as e:
        self.logger.error(f"Error parsing {response.url}: {e}", exc_info=True)

def parse_price(self, price_str: Optional[str]) -> Optional[float]:
    """Safely parse price string."""
    if not price_str:
        return None

    try:
        # Remove currency symbols and commas
        clean_price = price_str.replace('$', '').replace(',', '').strip()
        return float(clean_price)
    except (ValueError, AttributeError) as e:
        self.logger.warning(f"Failed to parse price: {price_str}")
        return None
```

### 2. Rate Limiting and Politeness

```python
custom_settings = {
    # Basic rate limiting
    'DOWNLOAD_DELAY': 1,  # Seconds between requests
    'CONCURRENT_REQUESTS_PER_DOMAIN': 2,

    # Auto-throttle (recommended)
    'AUTOTHROTTLE_ENABLED': True,
    'AUTOTHROTTLE_START_DELAY': 1,
    'AUTOTHROTTLE_MAX_DELAY': 10,
    'AUTOTHROTTLE_TARGET_CONCURRENCY': 2.0,

    # Respect robots.txt
    'ROBOTSTXT_OBEY': True,

    # Identify your bot
    'USER_AGENT': 'MyBot/1.0 (+http://www.example.com/bot)',
}
```

### 3. Data Validation

```python
from scrapy.loader import ItemLoader
from scrapy.loader.processors import TakeFirst, MapCompose, Join
from w3lib.html import remove_tags


class ProductLoader(ItemLoader):
    """Item loader with processors."""
    default_output_processor = TakeFirst()

    title_in = MapCompose(str.strip, remove_tags)
    description_in = MapCompose(str.strip, remove_tags)
    description_out = Join('\n')
    price_in = MapCompose(lambda x: x.replace('$', '').replace(',', ''))
    price_out = TakeFirst()


def parse_product(self, response):
    """Parse using ItemLoader."""
    loader = ProductLoader(item=Product(), response=response)

    loader.add_css('title', 'h1.title::text')
    loader.add_css('price', 'span.price::text')
    loader.add_css('description', 'div.description::text')
    loader.add_value('url', response.url)

    return loader.load_item()
```

### 4. Logging and Monitoring

```python
import logging

class MonitoredSpider(scrapy.Spider):
    """Spider with comprehensive logging."""

    name = "monitored"

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        self.items_scraped = 0
        self.errors = 0

    def parse(self, response):
        """Parse with metrics."""
        try:
            for item in self.extract_items(response):
                self.items_scraped += 1
                yield item
        except Exception as e:
            self.errors += 1
            self.logger.error(f"Error: {e}", exc_info=True)

    def closed(self, reason):
        """Log stats when spider closes."""
        self.logger.info(f"Spider closed: {reason}")
        self.logger.info(f"Items scraped: {self.items_scraped}")
        self.logger.info(f"Errors: {self.errors}")
```

## When to Use This Skill

Use this skill when:
- Creating new spiders from scratch
- Converting existing scrapers to Scrapy
- Implementing specific scraping patterns
- Needing JavaScript rendering capabilities
- Building API clients with Scrapy
- Requiring advanced crawling with rules

## Integration with Commands and Agents

**Commands**:
- `/spider <name> <type>` - Uses this skill to generate spider
- `/crawl <spider>` - Runs generated spiders

**Agents**:
- `@scrapy-expert` - Reviews generated spider code
- `@performance-optimizer` - Optimizes spider settings
- `@scraping-security` - Ensures ethical scraping practices

## Example Usage Patterns

### Generate Basic Product Spider
```python
# Command: /spider product_scraper basic
# Generates spider for simple product listing pages
```

### Generate CrawlSpider for E-commerce
```python
# Command: /spider ecommerce_crawler crawl
# Generates rule-based crawler for category navigation
```

### Generate Playwright Spider for SPA
```python
# Command: /spider spa_scraper playwright
# Generates browser-based spider for JavaScript sites
```

This skill automates spider creation while ensuring best practices, proper error handling, and optimal performance configurations.
