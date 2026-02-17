---
name: toppreise-agent
description: Query Swiss product price comparison data from toppreise.ch using the toppreise-cli command-line tool. Use when the user asks about product prices in Switzerland, comparing prices across Swiss shops, finding the cheapest offer for electronics, appliances, or any consumer product available in Switzerland. Triggers include questions like "What's the cheapest iPhone 16 in Switzerland?", "Compare prices for Samsung Galaxy S25", "Find me a cheap laptop in Swiss shops", "How much does a PS5 cost in Switzerland?", or "Which Swiss shop has the best price for AirPods?".
---

# toppreise-agent

Use the `toppreise-cli` binary to query toppreise.ch — Switzerland's largest price comparison website. Plain HTTP requests, no browser needed. Results are cached for 7 days. Every result includes a `Data from:` timestamp — use `--no-cache` if the data is stale.

## Commands

### Search

```bash
toppreise-cli search "<query>" [--limit <n>] [--sort <method>] [--category <slug>]
```

- `--limit`: max results (default 20)
- `--sort`: `relevance` (default), `price-asc`, `price-desc`, `rating`, `popularity`
- `--category`: filter by category slug

Output: Markdown list with product name, price range, offer count, product ID, URL.

### Product details

```bash
toppreise-cli product <id-or-url> [--section <name>]
```

Accepts a numeric product ID (e.g., `779730`), prefixed ID (`p779730`), or full toppreise.ch URL.

`--section` options: `overview`, `offers`, `description`, `specs`

Output: Full Markdown with overview (brand, price range, category), offer table (shop, price, shipping, total, rating), description, and specs.

### Categories

```bash
toppreise-cli categories [--flat] [--depth <n>]
```

Lists product categories with subcategories (parsed from the mega menu). Use `--flat` for a flat list. Use `--depth` to limit nesting depth (e.g., `--depth 1` for top-level only, `--depth 2` for one level of subcategories).

### Global flags

- `--lang <code>`: Language — `en` (default), `de`, `fr`
- `--no-cache`: Bypass cache (still writes to cache)
- `--delay <ms>`: Rate limit delay between requests
- `--debug`: Enable debug logging

## Workflows

### Find the cheapest product

1. Search with `--sort price-asc` to find the lowest prices
2. Get details on top candidates: `toppreise-cli product <id>`
3. Compare offers, shipping costs, and shop ratings
4. Recommend the best value option

```bash
toppreise-cli search "iphone 16" --sort price-asc --limit 10
toppreise-cli product 779730 --section offers
```

### Compare products across shops

1. Get product details for each product
2. Compare price ranges, offer counts, and available shops
3. Look at shipping costs and shop ratings for the best total deal

```bash
toppreise-cli product 779730 --section offers
toppreise-cli product 779728 --section offers
```

### Check specific product info

Use `--section` to fetch only what's needed:

```bash
toppreise-cli product 779730 --section overview     # brand, price range, category
toppreise-cli product 779730 --section offers        # shop comparison table
toppreise-cli product 779730 --section description   # product features
toppreise-cli product 779730 --section specs         # technical specifications
```

### Browse categories

```bash
toppreise-cli categories                             # full category tree with subcategories
toppreise-cli categories --depth 1                   # top-level categories only
toppreise-cli categories --depth 2                   # top-level + one level of subcategories
toppreise-cli categories --flat                      # flat list with IDs
```

### Localized results

```bash
toppreise-cli search "iphone" --lang de              # German
toppreise-cli search "iphone" --lang fr              # French
```

## Tips

- All prices are in CHF (Swiss Francs)
- Shop ratings are on a 6-point scale (not 5)
- The offer table shows both product price and total price (incl. shipping) — always compare totals
- Some shops have free shipping, others charge extra — this can change which offer is cheapest
- Use `--no-cache` if you need the absolute latest prices
- Search queries work best with specific product names (e.g., "iPhone 16 256GB" not just "phone")
- Product IDs can be found in search results or extracted from toppreise.ch URLs (the `-p{digits}` pattern)
- When recommending, mention the shop name, total price (incl. shipping), shop rating, and availability
