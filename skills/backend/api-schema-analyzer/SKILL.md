---
name: api-schema-analyzer
description: Analyze OpenAPI and Postman schemas for MCP tool generation. Use when analyzing API specifications, extracting endpoint information, generating tool signatures, or when user mentions OpenAPI, Swagger, API schema, endpoint analysis.
allowed-tools: Bash, Read, Write
---

# API Schema Analyzer

This skill analyzes OpenAPI/Swagger and Postman collection schemas to extract endpoint information for generating MCP tools.

## Instructions

### Analyzing OpenAPI Schemas

1. **Load OpenAPI Spec**
   - Use script: `scripts/analyze-openapi.py <openapi.json|yaml>`
   - Extracts: endpoints, methods, parameters, request/response schemas

2. **Generate Tool Signatures**
   - Use script: `scripts/generate-tool-signatures.py <openapi.json> --lang=python|typescript`
   - Creates: Function signatures with type hints from schema

### Analyzing Postman Collections

1. **Parse Collection**
   - Use script: `scripts/analyze-postman.py <collection.json>`
   - Extracts: requests, parameters, headers, auth requirements

2. **Map to MCP Tools**
   - Use script: `scripts/map-to-mcp-tools.py <collection.json> --output=tools.json`
   - Creates: MCP tool definitions with parameter mapping

## Available Scripts

- **`scripts/analyze-openapi.py`** - Parse OpenAPI specs (v2, v3)
- **`scripts/analyze-postman.py`** - Parse Postman collections
- **`scripts/generate-tool-signatures.py`** - Generate function signatures
- **`scripts/map-to-mcp-tools.py`** - Map API endpoints to MCP tools
- **`scripts/extract-schemas.sh`** - Extract request/response schemas

## Examples

**Example 1: Analyze OpenAPI Spec**
```bash
# Extract all endpoints and parameters
./scripts/analyze-openapi.py api-spec.json

# Generate Python tool signatures
./scripts/generate-tool-signatures.py api-spec.json --lang=python
```

**Example 2: Map Postman to MCP**
```bash
# Analyze Postman collection
./scripts/analyze-postman.py my-api.json

# Generate MCP tool mappings
./scripts/map-to-mcp-tools.py my-api.json --output=mcp-tools.json
```

## Requirements

- Python 3.7+ with `pyyaml`, `jsonschema` packages
- Valid OpenAPI v2/v3 spec or Postman collection
- jq for JSON processing

## Success Criteria

- ✅ Schema parsed successfully
- ✅ All endpoints extracted with full details
- ✅ Parameter types correctly identified
- ✅ Tool signatures generated with proper types
