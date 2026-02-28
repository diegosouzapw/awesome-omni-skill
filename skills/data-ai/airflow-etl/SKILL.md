---
name: airflow-etl
description: Generate Apache Airflow ETL pipelines for government websites and document sources. Explores websites to find downloadable documents, verifies commercial use licenses, and creates complete Airflow DAG assets with daily scheduling. Use when user wants to create ETL pipelines, scrape government documents, or automate document collection workflows.
---

# Airflow ETL Pipeline Generator

Generate production-ready Apache Airflow ETL pipelines that automatically discover, download, and transform documents from government websites and other data sources into structured markdown files.

## Workflow

### Phase 1: Website Exploration and Discovery

1. **Initial Analysis**:
   - Use WebFetch to explore the provided website URL
   - Identify document sections (downloads, archives, publications, meetings, etc.)
   - Look for API endpoints, RSS feeds, or structured data sources
   - Note pagination patterns and document organization

2. **License Verification**:
   - Search for license information (Creative Commons, Open Government License, etc.)
   - Look for terms of use or copyright notices
   - Check for explicit commercial use permissions
   - If unclear, ask user about license status

3. **Document Inventory**:
   - Identify document types (PDF, DOC, DOCX, etc.)
   - Understand the URL patterns for documents
   - Determine how to detect new documents
   - Note any metadata available (dates, categories, titles)

4. **User Confirmation**:
   - Present findings in a clear summary
   - Show example document URLs
   - Describe the discovered structure
   - Ask user to confirm this is the correct data source

### Phase 2: Generate Airflow Pipeline Assets

Create a complete, production-ready Airflow project structure:

```
airflow_pipelines/
├── dags/
│   └── [source_name]_etl_dag.py
├── operators/
│   ├── __init__.py
│   ├── document_scraper.py
│   └── document_converter.py
├── utils/
│   ├── __init__.py
│   ├── license_checker.py
│   └── file_manager.py
├── config/
│   └── [source_name]_config.yaml
├── requirements.txt
└── README.md
```

#### File Generation Requirements:

**1. DAG File** (`dags/[source_name]_etl_dag.py`):
- Daily schedule (adjustable)
- Clear task dependencies
- Error handling and retries
- Sensor for checking new documents
- Download task
- Conversion task to markdown
- File organization task
- Use Airflow best practices (XComs, task groups, dynamic task generation)

**2. Document Scraper** (`operators/document_scraper.py`):
- BeautifulSoup or Scrapy for web scraping
- Request handling with retries
- Respect robots.txt
- User-agent configuration
- Rate limiting
- Checksum/hash tracking to avoid re-downloading
- State management for incremental updates

**3. Document Converter** (`operators/document_converter.py`):
- Support for PDF, DOC, DOCX conversion to markdown
- Use libraries like pypandoc, pdfplumber, or python-docx
- Preserve document structure (headings, lists, tables)
- Extract metadata
- Handle encoding issues
- Clean and normalize output

**4. License Checker** (`utils/license_checker.py`):
- Validate license information
- Check for commercial use permission
- Log license status
- Skip non-compliant documents

**5. File Manager** (`utils/file_manager.py`):
- Create meaningful directory structure
- Organize by date, category, or document type
- Generate consistent filenames
- Handle duplicates
- Maintain index of processed documents

**6. Configuration** (`config/[source_name]_config.yaml`):
```yaml
source:
  name: "Source Name"
  url: "https://example.com"
  document_section: "/documents"

schedule:
  interval: "0 0 * * *"  # Daily at midnight

storage:
  base_path: "/data/documents"
  structure: "year/month/category"

scraping:
  rate_limit: 1  # requests per second
  user_agent: "ETL Pipeline Bot"
  retry_attempts: 3

conversion:
  format: "markdown"
  preserve_structure: true
  extract_metadata: true
```

**7. Requirements** (`requirements.txt`):
```
apache-airflow>=2.7.0
beautifulsoup4>=4.12.0
requests>=2.31.0
pypandoc>=1.12
pdfplumber>=0.10.0
python-docx>=1.0.0
pyyaml>=6.0
lxml>=4.9.0
```

**8. Documentation** (`README.md`):
- Pipeline overview
- Setup instructions
- Configuration guide
- Airflow connection requirements
- Monitoring and troubleshooting
- Example usage

### Phase 3: Implementation Notes

**Important Considerations**:
- Include comprehensive error handling
- Log all operations for debugging
- Add data quality checks
- Implement idempotency (safe to re-run)
- Use Airflow variables for sensitive config
- Add email/Slack alerts for failures
- Document the directory structure created

**Code Quality**:
- Follow PEP 8 style guidelines
- Add docstrings to all functions
- Include type hints
- Write modular, reusable code
- Add comments for complex logic

**Testing Recommendations** (optional):
- Suggest basic unit tests for utilities
- Recommend integration testing approach
- Provide example test cases

### Phase 4: Delivery

1. Generate all files using Write tool
2. Provide summary of created assets
3. Explain how to deploy to Airflow:
   - Copy files to Airflow home directory
   - Install requirements
   - Enable the DAG in Airflow UI
   - Configure connections if needed
4. Suggest next steps (testing, scheduling, monitoring)

## Examples

### Example 1: German Bundestag Documents
```
User: "Create an ETL pipeline for https://www.bundestag.de/digitales to collect committee meeting documents"

Skill Response:
- Explores the digital committee section
- Finds document sections (agendas, protocols, reports)
- Checks copyright notice
- Confirms findings with user
- Generates complete Airflow pipeline
- Creates scraper for committee documents
- Sets up markdown conversion
- Organizes by committee and date
```

### Example 2: EU Open Data Portal
```
User: "Build an Airflow pipeline for EU legislation documents from data.europa.eu"

Skill Response:
- Discovers API endpoints
- Verifies open data license
- Generates API-based scraper
- Creates pipeline with API operators
- Includes rate limiting
- Organizes by document type and year
```

## Key Success Criteria

- Pipeline runs successfully in Airflow
- Documents are correctly downloaded
- Markdown conversion preserves structure
- File organization is logical and scalable
- License compliance is enforced
- New documents are detected automatically
- Pipeline is idempotent and fault-tolerant

## Tips for Users

- Provide the main URL of the data source
- Mention any specific document types needed
- Specify preferred organization structure
- Note any special requirements (date ranges, categories)
- Test with a small sample before full deployment
