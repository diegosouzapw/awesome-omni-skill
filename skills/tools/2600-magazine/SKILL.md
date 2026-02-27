---
name: 2600-magazine
description: Query and explore the 2600: The Hacker Quarterly magazine archive (1984-present) via DuckDB. Provides structured access to 168+ issues covering hacker culture, security, privacy, telephony, and digital rights without loading full content into context.
---

# 2600: The Hacker Quarterly - Archive Skill

## What is 2600 Magazine?

**2600: The Hacker Quarterly** (ISSN 0749-3851) is the longest-running hacker publication, founded in 1984 by Emmanuel Goldstein (Eric Corley) in Middle Island, New York. Named after the 2600Hz tone that phone phreakers used to control AT&T's long-distance switching systems, it has published continuously for 40+ years as a quarterly magazine.

### Cultural Significance
- **The hacker community's primary publication** — sometimes called "the hacker's bible"
- Advocates the **hacker ethic**: information freedom, opposition to corporate control, exposing vulnerabilities as consumer advocacy
- Founded partly in response to Corley's 1983 arrest for alleged hacking (charges dropped)
- The pen name "Emmanuel Goldstein" references the resistance leader in Orwell's *1984*

### Key Legal Case
The MPAA sued 2600 in 2000 (*Universal City Studios v. Reimerdes*) for publishing/linking to DeCSS (DVD decryption software). Landmark DMCA/free speech case.

### Associated Projects
- **Off the Hook**: Weekly radio show on WBAI (NYC), running since 1988
- **HOPE (Hackers on Planet Earth)**: Biennial hacker convention since 1994
- **2600 Meetings**: Monthly gatherings in 100+ cities (first Friday of each month)

## What This Skill Is For

Use this skill when you need to:

1. **Research hacker culture history** — find which volumes/issues covered specific topics
2. **Trace technology evolution** — phone phreaking to IoT security across 40 years
3. **Find security techniques by era** — what was cutting-edge in any given year
4. **Access archive.org PDFs** — get direct download URLs for available issues
5. **Understand hacker ethics debates** — free speech, surveillance, whistleblowing coverage
6. **Reference HOPE conference topics** — biennial conference coverage since 1994
7. **Build reading lists** — curated paths through the archive by topic

## Database Location

```
/Users/alice/.claude/skills/2600-magazine/2600.duckdb
```

## Schema (ACSet-Inspired)

The database models the magazine as a categorical structure:

```
issues ←── notable_articles ──→ topic_index
  │                                  │
  │ (archive_url)          (category, frequency)
  │
  └──→ access_methods
```

### Tables

| Table | Description |
|-------|-------------|
| `issues` | All 168+ quarterly issues (Vol 1-42, 1984-2025) |
| `topic_index` | 20 major recurring topics with category and volume range |
| `notable_articles` | Curated index of significant articles |
| `access_methods` | How to access issues (archive.org, store, API) |

### Views

| View | Description |
|------|-------------|
| `available_on_archive` | Issues with archive.org download URLs |
| `issues_by_decade` | Issue counts grouped by decade |
| `topic_coverage` | Topic spans across volumes |

## Entry Points (Query Without Loading Content)

### By Era
```sql
-- What was being hacked in the 90s?
SELECT volume, issue, year, season FROM issues WHERE year BETWEEN 1990 AND 1999;
```

### By Topic
```sql
-- All security-related topics
SELECT topic, first_volume, last_volume, frequency
FROM topic_index WHERE category = 'security';
```

### By Availability
```sql
-- Get downloadable PDF URLs
SELECT year, season, archive_url FROM available_on_archive ORDER BY year;
```

### By Decade Summary
```sql
SELECT * FROM issues_by_decade;
```

### Notable Articles
```sql
-- Famous articles with context
SELECT na.title, na.author, na.summary, i.year, t.topic
FROM notable_articles na
JOIN topic_index t ON na.topic_id = t.id
LEFT JOIN issues i ON na.volume = i.volume AND na.issue = i.issue;
```

### Direct Archive Access
```sql
-- How to bulk download
SELECT method, url_template, format, notes FROM access_methods;
```

## Topic Categories

| Category | Topics |
|----------|--------|
| **security** | Computer Security, Social Engineering, Network Security, Web Security, Lock Picking, IoT Security, AI/ML Security |
| **telephony** | Phone Phreaking, Cell Phone Security |
| **privacy** | Encryption & Cryptography, Government Surveillance |
| **politics** | Digital Rights & Free Speech, Whistleblowing & Leaks |
| **technical** | Reverse Engineering, Linux & Open Source, Wireless & RF Hacking |
| **community** | Reader Letters, Marketplace, HOPE Conference Coverage |
| **culture** | Pay Phone Photography |

## Volume-to-Year Mapping

`year = 1984 + (volume - 1)`

| Decade | Volumes | Years | Era |
|--------|---------|-------|-----|
| Early | 1-6 | 1984-1989 | Phone phreaking, early BBS |
| 1990s | 7-16 | 1990-1999 | Internet explosion, crypto wars |
| 2000s | 17-26 | 2000-2009 | Web security, DeCSS, post-9/11 surveillance |
| 2010s | 27-36 | 2010-2019 | Snowden, IoT, mobile security |
| 2020s | 37-42+ | 2020-2025 | AI security, pandemic hacking, cloud |

## Archive Access

### Internet Archive (Volumes 1-18 confirmed)
```bash
# Single issue PDF
curl -O "https://archive.org/download/2600magazine/2600_1-1.pdf"

# Bulk download via ia CLI
pip install internetarchive
ia download 2600magazine --glob='*.pdf'

# Metadata
curl "https://archive.org/metadata/2600magazine"
```

### Later Volumes (Individual Items)
Some later volumes exist as separate archive.org items (e.g., `2600-volume-36-issue-3`).

### Official Store
Current and back issues: https://store.2600.com (PDF/EPUB, $18.99/year for early issues)

## CLI Recipes

```bash
# Quick query
duckdb /Users/alice/.claude/skills/2600-magazine/2600.duckdb -c "
SELECT year, season, archive_url IS NOT NULL as available
FROM issues WHERE year = 1995;"

# Topic overview
duckdb /Users/alice/.claude/skills/2600-magazine/2600.duckdb -c "
SELECT topic, category, first_volume, last_volume FROM topic_index ORDER BY category;"

# Available PDFs count
duckdb /Users/alice/.claude/skills/2600-magazine/2600.duckdb -c "
SELECT COUNT(*) as available, COUNT(*) * 100.0 / (SELECT COUNT(*) FROM issues) as pct
FROM available_on_archive;"
```

## GF(3) Triad

```
acsets (-1)  +  duckdb-ies (0)  +  entry-point-analyzer (+1)  =  0
 schema         storage/query       navigating without
 definition     engine              loading content
```

**Skill Trit**: 0 (ERGODIC - coordination between archive structure and query access)

## Related Skills

- `acsets` — Schema definition as C-set functor
- `duckdb-ies` — DuckDB analytics patterns
- `entry-point-analyzer` — Strategic access points into large datasets
- `reverse-engineering` — Core 2600 topic area
- `webapp-testing` — Web security techniques covered in later volumes
