---
name: Argentine Invoice Processing System
description: Complete invoice processing system for Argentine utility bills with OCR, classification, and automated organization
version: 1.0.0
author: Santiago Braida
tags:
  - invoice-processing
  - document-automation
  - ocr
  - utilities
  - argentina
  - file-organization
categories:
  - document-processing
  - automation
prerequisites:
  - .NET 9.0 Runtime
  - Tesseract OCR with Spanish support
  - macOS or Linux environment
related_skills:
  - invoice-processing
  - ocr-troubleshooting
  - service-identification
  - date-extraction
  - testing-procedures
  - deployment-guide
---

# Argentine Invoice Processing System

## Overview

This skill enables automated processing, classification, and organization of Argentine utility invoices (electricity, water, gas, municipal services, etc.). The system extracts text from PDFs and images using OCR, identifies service providers, extracts dates, and organizes files into a structured year/month hierarchy with standardized naming.

## What This Skill Does

- **OCR Processing**: Extract text from PDFs, images (JPG, PNG) using Tesseract
- **Service Classification**: Identify service providers (AYSA, Edenor, Metrogas, ARBA, etc.)
- **Date Extraction**: Parse due dates from Spanish-language invoices
- **File Naming**: Apply consistent naming convention: `{service}_{YYYY-MM-DD}_{payment}.ext`
- **Organization**: Create year/month folder structure (e.g., `2025/03_marzo/`)
- **Parallel Processing**: Handle multiple files concurrently

## When to Use This Skill

Use this skill when you need to:

- Process and organize utility invoices
- Extract dates from Spanish-language documents
- Classify Argentine service providers
- Troubleshoot OCR issues with Spanish text
- Deploy or maintain the invoice processing system
- Add new service providers or rules
- Test invoice processing functionality

## Quick Start

### Basic Usage

```bash
# Run with default configuration
dotnet run --project FileContentRenamer/FileContentRenamer.csproj

# Process specific directory
dotnet run --project FileContentRenamer -- /path/to/invoices
```

### Key Configuration

```json
{
  "AppConfig": {
    "BasePath": "/Users/santibraida/Downloads/__comprobantes/servicios",
    "FileExtensions": [".pdf", ".jpg", ".png"],
    "TesseractLanguage": "spa+eng",
    "MaxDegreeOfParallelism": 4
  }
}
```

## Detailed Documentation

This skill is organized into specialized sub-skills for different aspects of the system:

### 1. **Invoice Processing** ([invoice-processing.md](invoice-processing.md))

Core workflow and service provider directory. **Start here** for system overview.

**Covers**:

- Complete processing workflow
- Service provider catalog (AYSA, Edenor, Metrogas, etc.)
- File naming conventions
- Folder organization structure
- Configuration reference

**When to use**: Understanding the overall system, adding new providers, configuring the app

### 2. **OCR Troubleshooting** ([ocr-troubleshooting.md](ocr-troubleshooting.md))

Diagnose and fix OCR issues with Tesseract.

**Covers**:

- Common OCR problems and solutions
- Spanish character recognition
- Image quality requirements
- Performance optimization
- Tesseract configuration

**When to use**: OCR producing garbled text, missing content, or slow performance

### 3. **Service Identification** ([service-identification.md](service-identification.md))

Detailed patterns for each service provider.

**Covers**:

- Provider-specific keywords and identifiers
- Invoice characteristics and formats
- Validation rules and amount ranges
- Edge cases and conflicts
- Seasonal patterns

**When to use**: Adding new providers, debugging misclassification, understanding provider specifics

### 4. **Date Extraction** ([date-extraction.md](date-extraction.md))

Comprehensive date parsing patterns and validation.

**Covers**:

- All regex patterns with priority order
- Argentine date format handling
- OCR error correction for dates
- Multiple date scenarios
- Validation and edge cases

**When to use**: Date extraction issues, adding new date patterns, understanding parsing logic

### 5. **Testing Procedures** ([testing-procedures.md](testing-procedures.md))

Complete testing guide from unit tests to production validation.

**Covers**:

- Unit, integration, and E2E testing
- Test data management
- Manual testing procedures
- Performance and regression testing
- CI/CD integration

**When to use**: Writing tests, validating changes, testing new invoice types, quality assurance

### 6. **Deployment Guide** ([deployment-guide.md](deployment-guide.md))

Production deployment and maintenance handbook.

**Covers**:

- Installation and setup
- Production configuration
- Monitoring and health checks
- Backup and recovery
- Troubleshooting common issues
- Security considerations

**When to use**: Deploying to production, setting up scheduled runs, maintenance, troubleshooting

## Common Workflows

### Adding a New Service Provider

1. Collect 3-5 sample invoices
2. Read [service-identification.md](service-identification.md) for pattern analysis
3. Update `appsettings.json` with new rule
4. Test with samples (see [testing-procedures.md](testing-procedures.md))
5. Validate results and adjust keywords

### Troubleshooting OCR Issues

1. Check [ocr-troubleshooting.md](ocr-troubleshooting.md) for your specific issue
2. Verify Tesseract configuration and language packs
3. Test image quality requirements
4. Apply preprocessing if needed
5. Check logs for detailed error messages

### Fixing Date Extraction

1. Review [date-extraction.md](date-extraction.md) for pattern priority
2. Check if date format is supported
3. Apply OCR error correction patterns
4. Add new regex pattern if needed
5. Validate with unit tests

### Deploying to Production

1. Follow [deployment-guide.md](deployment-guide.md) installation steps
2. Configure production settings
3. Set up monitoring and logging
4. Test with sample invoices
5. Schedule automated runs (cron/launchd)

## Architecture Overview

```text
FileContentRenamer/
├── Program.cs                  # Entry point, configuration loading
├── Configuration/
│   └── ServiceConfiguration.cs # Dependency injection setup
├── Models/
│   ├── AppConfig.cs           # Configuration model
│   └── NamingRule.cs          # Service provider rules
└── Services/
    ├── FileService.cs         # Main orchestrator
    ├── PdfProcessor.cs        # PDF text extraction
    ├── ImageProcessor.cs      # OCR with Tesseract
    ├── TextProcessor.cs       # Plain text handling
    ├── DateExtractor.cs       # Date parsing
    ├── FilenameGenerator.cs   # Naming rules application
    ├── DirectoryOrganizer.cs  # Folder structure creation
    └── FileValidator.cs       # File validation
```

## Supported Service Providers

| Provider                | Type               | Code                  |
| ----------------------- | ------------------ | --------------------- |
| AYSA                    | Water & Sanitation | `aysa`                |
| Edenor                  | Electricity        | `edenor`              |
| Metrogas                | Natural Gas        | `metrogas`            |
| Municipality of Quilmes | Municipal Taxes    | `municipal_quilmes`   |
| ARBA Inmobiliario       | Property Tax       | `arba_inmobiliario`   |
| ARBA Automotor          | Vehicle Tax        | `arba_automotor`      |
| Personal/Flow           | Mobile/Internet    | `personal`            |
| Quilmes High School     | School Tuition     | `high_school_cuota`   |
| Quilmes High School     | School Lunch       | `high_school_comedor` |
| Gloria                  | Domestic Service   | `gloria`              |

See [service-identification.md](service-identification.md) for detailed information on each provider.

## Key Features

### Intelligent Date Extraction

Handles multiple date formats with priority order:

1. Due date (abbreviated): `Vto.:DD/MM/YYYY`
2. Due date (full): `vencimiento DD/MM/YYYY`
3. Spanish format: `DD de MONTH de YYYY`
4. Generic: `DD/MM/YYYY`

### OCR with Error Correction

- Automatic correction of common OCR mistakes (O→0, I→1, S→5)
- Support for Spanish characters (ñ, á, é, í, ó, ú)
- Multi-language support (Spanish + English)

### Flexible Organization

Files organized into year/month structure:

```text
servicios/
└── 2025/
    ├── 03_marzo/
    │   └── aysa_2025-03-21_santander.pdf
    └── 08_agosto/
        └── gloria_2025-08-08_mercadopago.jpeg
```

### Parallel Processing

Configurable parallelism for faster processing of large batches while maintaining file safety with proper locking.

## Configuration Reference

### Key Settings

| Setting                      | Description            | Default                    |
| ---------------------------- | ---------------------- | -------------------------- |
| `BasePath`                   | Root directory to scan | `.`                        |
| `FileExtensions`             | File types to process  | `[".pdf", ".jpg", ".png"]` |
| `IncludeSubdirectories`      | Scan subdirectories    | `true`                     |
| `TesseractLanguage`          | OCR languages          | `"spa+eng"`                |
| `MaxDegreeOfParallelism`     | Concurrent files       | `4`                        |
| `ForceReprocessAlreadyNamed` | Reprocess named files  | `false`                    |

See [invoice-processing.md](invoice-processing.md) for complete configuration documentation.

## Logging

Logs are written to:

- Console: Real-time processing status
- File: `logs/app{YYYYMMDD}.log` (daily rotation)

### Log Levels

- **Information**: Normal processing flow
- **Warning**: Skipped files, no content found
- **Error**: Processing failures, OCR errors
- **Debug**: Detailed extraction and matching info

## Performance

Typical performance (4-core system):

- **PDF**: ~1-2 seconds per file
- **Image (OCR)**: ~3-5 seconds per file
- **Large batch** (100 files): ~5-8 minutes

See [deployment-guide.md](deployment-guide.md) for optimization tips.

## Troubleshooting Quick Reference

| Issue                    | See                                                    | Quick Fix                                       |
| ------------------------ | ------------------------------------------------------ | ----------------------------------------------- |
| Garbled OCR text         | [ocr-troubleshooting.md](ocr-troubleshooting.md)       | Check image quality, verify `spa` language pack |
| Wrong service identified | [service-identification.md](service-identification.md) | Add more keywords, check priority               |
| Date not found           | [date-extraction.md](date-extraction.md)               | Verify date format, check OCR quality           |
| Files in wrong folder    | [invoice-processing.md](invoice-processing.md)         | Check `BasePath` configuration                  |
| High memory usage        | [deployment-guide.md](deployment-guide.md)             | Reduce `MaxDegreeOfParallelism`                 |

## Testing

Run tests:

```bash
dotnet test FileContentRenamer.Tests/
```

See [testing-procedures.md](testing-procedures.md) for comprehensive testing guide.

## Updates and Maintenance

- **Weekly**: Review logs for errors
- **Monthly**: Update service provider rules if needed
- **Quarterly**: Review and archive old invoices
- **Yearly**: Update dependencies and .NET runtime

See [deployment-guide.md](deployment-guide.md) for maintenance procedures.

## Version History

- **v1.0.0** (2025-11-06): Initial skill documentation
  - Complete invoice processing system
  - Support for 10 service providers
  - OCR with Tesseract
  - Automated organization

## Support

- **Repository**: <https://github.com/santibraida/comprobantes>
- **Issues**: Create GitHub issue for bugs/features
- **Documentation**: See `.skills/` directory

## Related Technologies

- **.NET 9.0**: Application framework
- **Tesseract OCR**: Text extraction from images
- **Serilog**: Structured logging
- **xUnit**: Unit testing framework
- **ImageMagick**: Image preprocessing (optional)

## Best Practices

1. **Always backup** before processing
2. **Test with samples** before bulk processing
3. **Monitor logs** for errors and warnings
4. **Keep configuration** in version control (except production secrets)
5. **Update skills documentation** when adding features

## Getting Help

1. Check the relevant detailed skill file for your issue
2. Review logs for error messages
3. Search existing GitHub issues
4. Create new issue with:
   - Sample invoice (redacted)
   - Log excerpt
   - Configuration used
   - Expected vs actual behavior

---

**Start with [invoice-processing.md](invoice-processing.md) for the complete system overview, then dive into specific skill files as needed.**
