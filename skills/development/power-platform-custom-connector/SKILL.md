---
name: power-platform-custom-connector
description: Build Power Platform custom connectors (Independent Publisher and Verified Publisher) for Microsoft certification. Use when user says "create a custom connector", "build a Power Automate connector", "write apiDefinition.swagger.json", "configure apiProperties.json", "add x-ms-* extensions", "set up OAuth for a connector", "write script.csx custom code",  "create a webhook trigger connector", "prepare connector for certification PR", "add dynamic dropdowns", "configure policy templates", or "submit connector to PowerPlatformConnectors repo". Capabilities; Swagger 2.0 OpenAPI definitions, 5 auth types, 13 policy templates, C# custom code,webhook triggers, dynamic values, Copilot Studio AI extensions, certification checklists, pac connector CLI. Do NOT use for generic REST API design, Azure API Management policies, or Logic Apps built-in connectors.
license: MIT
metadata:
  author: rwilson504
  version: 1.0.0
  category: development
  tags:
    - power-platform
    - custom-connector
    - swagger
    - openapi
    - power-automate
    - power-apps
---

# Power Platform Custom Connector Creation

Build production-ready Power Platform custom connectors and submit them to the [microsoft/PowerPlatformConnectors](https://github.com/microsoft/PowerPlatformConnectors) GitHub repo. This skill covers **Independent Publisher** and **Verified Publisher** connector paths.

**References:** [OPENAPI_EXTENSIONS.md](references/OPENAPI_EXTENSIONS.md) | [AUTH_PATTERNS.md](references/AUTH_PATTERNS.md) | [POLICY_TEMPLATES.md](references/POLICY_TEMPLATES.md) | [CUSTOM_CODE.md](references/CUSTOM_CODE.md) | [WEBHOOK_TRIGGERS.md](references/WEBHOOK_TRIGGERS.md) | [CERTIFICATION.md](references/CERTIFICATION.md) | [EXAMPLES.md](references/EXAMPLES.md) | [COMMON_MISTAKES.md](references/COMMON_MISTAKES.md)

---

## First Step: Identify Publisher Type

> **IMPORTANT — Ask the user before proceeding:**
>
> Before generating any connector files or certification guidance, ask the user:
>
> *"Are you building an **Independent Publisher** connector or a **Verified Publisher** connector?"*
>
> - **Independent Publisher** — You are a community member (MVP, developer, enthusiast) wrapping a third-party API you do **not** own.
> - **Verified Publisher** — You are the service/API owner and want to publish your own connector.
>
> The answer determines: directory structure, brand color, icon requirements, OAuth support, certification workflow, submission target, and connector title format. Tailor **all** subsequent guidance to the selected publisher type.

---

## Quick Reference: Independent Publisher vs Verified Publisher

| Aspect | Independent Publisher | Verified Publisher |
|--------|----------------------|-------------------|
| **Who** | Community member (MVP, developer) — does NOT own the API | Service owner — owns the underlying API |
| **Brand color** | **Must** be `#da3b01` | Custom brand color allowed |
| **PR target** | `independent-publisher-connectors/` on `dev` branch | `certified-connectors/` on `dev` branch |
| **Certification** | Free, reviewed by MS Connector Certification Team | Free, via ISV Studio registration |
| **Connector tier** | Premium (automatic for external APIs) | Premium (automatic for external APIs) |
| **PR requirements** | Screenshots of 3 unique operations working in a Flow | Thorough API testing documentation |
| **OAuth redirect** | Per-connector redirect URI (mandatory since Feb 2024) | Per-connector redirect URI (mandatory since Feb 2024) |
| **OAuth support** | **Not currently supported** for certification | Fully supported (AAD, Generic OAuth) |
| **Connector title** | Service name only (≤30 chars) | Service name only (≤30 chars) |
| **Icon** | Generic icon assigned automatically | Custom icon required (100×100 to 230×230 px, PNG, non-white/non-default background) |
| **Submission target** | GitHub PR to `microsoft/PowerPlatformConnectors` | Microsoft Partner Center |
| **Identity verification** | Verified credentials via OneVet + Microsoft Authenticator | Partner Center account |
| **Deployment timeline** | ~15 business days after approval (Friday deployments) | ~15 business days after approval (Friday deployments) |
| **Intro artifact** | `intro.md` included in PR | `intro.md` packaged in submission zip |

**Decision rule:** If you own the API/service, go **Verified Publisher**. If you're wrapping a third-party API you don't own, go **Independent Publisher**.

---

## Getting Started

### Required Files

Every connector directory must contain:

| File | Required | Purpose |
|------|----------|---------|
| `apiDefinition.swagger.json` | **Yes** | OpenAPI 2.0 (Swagger) definition — operations, parameters, responses, schemas |
| `apiProperties.json` | **Yes** | Connection parameters, brand color, auth config, policy templates |
| `readme.md` | **Yes** | Description, prerequisites, supported operations, credentials guide |
| `icon.png` | Optional | Connector icon displayed in the designer |
| `script.csx` | Optional | C# custom code for request/response transformation |

### Directory Structure

```
PowerPlatformConnectors/
├── independent-publisher-connectors/
│   └── YourConnector/
│       ├── apiDefinition.swagger.json
│       ├── apiProperties.json
│       ├── readme.md
│       ├── icon.png          (optional)
│       └── script.csx        (optional)
├── certified-connectors/       (verified publishers)
├── custom-connectors/          (samples only)
├── schemas/                    (JSON Schema for validation)
│   ├── apiDefinition.swagger.schema.json
│   └── paconn-apiProperties.schema.json
└── templates/                  (starter templates)
```

### Scaffold a New Connector

```bash
# Fork and clone the repo
git clone https://github.com/<your-fork>/PowerPlatformConnectors.git
cd PowerPlatformConnectors
git checkout dev

# Create connector directory
mkdir independent-publisher-connectors/YourConnector
cd independent-publisher-connectors/YourConnector

# Create required files (see EXAMPLES.md for complete templates)
```

---

## Swagger Definition Structure (apiDefinition.swagger.json)

**Critical:** Must be **OpenAPI 2.0** (Swagger). OpenAPI 3.0 is NOT supported.

If your source API provides an OpenAPI 3.0 definition, convert it before importing:

```bash
# Option 1: api-spec-converter CLI (requires Node.js)
npm install -g api-spec-converter
api-spec-converter --from=openapi_3 --to=swagger_2 openapi3.yaml > apiDefinition.swagger.json

# Option 2: Apimatic Transform (https://www.apimatic.io/transformer)
# Upload your 3.0 file → select Swagger 2.0 → download
```

After conversion, manually verify the output — automated tools may not handle all Power Platform–specific extensions correctly.

```json
{
  "swagger": "2.0",
  "info": {
    "version": "1.0.0",
    "title": "My Service",
    "description": "Short description (30-500 chars). No words 'API', 'Connector', or Power Platform product names.",
    "contact": {
      "name": "Your Name",
      "url": "https://github.com/yourusername",
      "email": "you@example.com"
    }
  },
  "host": "api.myservice.com",
  "basePath": "/v1",
  "schemes": ["https"],
  "consumes": ["application/json"],
  "produces": ["application/json"],
  "securityDefinitions": {},
  "security": [],
  "paths": {},
  "definitions": {},
  "x-ms-connector-metadata": [
    { "propertyName": "Website", "propertyValue": "https://myservice.com" },
    { "propertyName": "Privacy policy", "propertyValue": "https://myservice.com/privacy" },
    { "propertyName": "Categories", "propertyValue": "AI;Business Intelligence" }
  ]
}
```

**Key rules:**
- `title` — **Maximum 30 characters**. Cannot include the words "API", "Connector", "Copilot Studio", or any Power Platform product names. Must end with an alphanumeric character (no trailing punctuation, spaces, or special chars). Must be unique and distinguishable from existing connector titles. Note: the `(Independent Publisher)` suffix goes on the **PR title**, not the connector title in the OpenAPI definition
- `description` — Must be **30-500 characters** for certification quality and must stay within the platform import hard limit of **1000 characters**. Use plain text only (no HTML tags). Cannot contain "API", "Copilot Studio", or Power Platform product names. Must be free of grammatical and spelling errors. Should concisely describe the main purpose and value of the connector
- `contact` — Include `name`, `url`, and `email` with a valid email address
- `x-ms-connector-metadata` — **Required** array with Website, Privacy policy, and Categories. The `Categories` value must be a semicolon-delimited string from these allowed values: `AI`, `Business Management`, `Business Intelligence`, `Collaboration`, `Commerce`, `Communication`, `Content and Files`, `Data`, `Finance`, `Human Resources`, `Internet of Things`, `IT Operations`, `Lifestyle and Entertainment`, `Marketing`, `Productivity`, `Sales and CRM`, `Security`, `Social Media`, `Website`
- `consumes` / `produces` — Always explicitly set to `["application/json"]` for JSON APIs. Do not omit these fields even if the API only handles JSON — being explicit prevents content-type mismatches
- `schemes` — Must include `"https"` (HTTP not allowed for production connectors)
- `host` — **Production host URL only**. Staging, dev, and test host URLs are not allowed. Base hostname only, no path segments

**File formatting:**
- Use soft tabs with **4 spaces** for indentation — no hard tabs
- Remove trailing whitespace from all lines
- Structure the file in this order: swagger version → info → host/schemes → consumes/produces → paths → definitions → parameters

---

## Operations (Paths)

Define actions and triggers in the `paths` object:

```json
"paths": {
  "/items": {
    "get": {
      "operationId": "ListItems",
      "summary": "List all items",
      "description": "Retrieves a list of all items from the service.",
      "x-ms-visibility": "important",
      "parameters": [
        {
          "name": "status",
          "in": "query",
          "type": "string",
          "x-ms-summary": "Status",
          "description": "Filter items by their current status.",
          "x-ms-visibility": "advanced"
        }
      ],
      "responses": {
        "200": {
          "description": "Success",
          "schema": { "$ref": "#/definitions/ItemList" }
        },
        "400": {
          "description": "The request was invalid or malformed."
        },
        "404": {
          "description": "The requested resource was not found."
        }
      }
    },
    "post": {
      "operationId": "CreateItem",
      "summary": "Create an item",
      "description": "Creates a new item in the service.",
      "parameters": [
        {
          "name": "body",
          "in": "body",
          "required": true,
          "schema": { "$ref": "#/definitions/CreateItemRequest" }
        }
      ],
      "responses": {
        "201": {
          "description": "Created",
          "schema": { "$ref": "#/definitions/Item" }
        }
      }
    }
  }
}
```

**Key rules:**
- `operationId` — **Must** be PascalCase (capitalize every word), unique across all operations. Remove all non-alpha characters — no hyphens, underscores, or spaces (e.g., `get_user-info` → `GetUserInfo`)
- `summary` — **Required** for every operation. Must be sentence case, **80 characters or fewer**, and must **end with an alphanumeric character** (no trailing punctuation, spaces, or special characters). Must contain only alphanumeric characters or parentheses — **no slashes (`/`)**. As a naming convention: start with **"List"** when the operation returns multiple records, **"Get"** when it returns a single record. For triggers, use the format: **"When a [event]"** (e.g., "When a task is created")
- `summary` and `description` **must not have the same text** — the description should provide additional information beyond the summary
- `description` — **Required** for every operation and parameter. Must be a **full, descriptive sentence ending in punctuation**. Must not contain URLs or HTML markup. Must be in English and free of grammatical or spelling errors
- **Input description UX style** — Parameter descriptions are shown as helper text in Power Automate/Power Apps. Keep them concise and action-oriented: avoid repetitive prefixes like "Filter the response by"; prefer short forms like "Location code (ISO-3).", "Reference period start (inclusive).", or "Select from dropdown." for long enum lists

### Description Style Examples (Parameters)

Use this quick mapping when rewriting parameter helper text for designer UX.

| Pattern | Avoid | Prefer |
|---|---|---|
| Generic filter prefix | `Filter the response by location code.` | `Location code (ISO-3).` |
| Date lower bound | `Filter entries to include rows where the reference period overlaps with or extends beyond this date...` | `Reference period start (inclusive).` |
| Date upper bound | `Filter entries to include rows where the reference period overlaps with or begins prior to this date...` | `Reference period end (inclusive).` |
| Boolean flag | `The has_hrp flag. The has_hrp flag indicates whether...` | `Whether the location has a Humanitarian Response Plan (HRP).` |
| Enum (short list) | `... available values are described in documentation.` | `Allowed values: csv, json.` |
| Enum (long list/dropdown) | `Allowed values: value1, value2, value3, ...` | `Select from dropdown.` |
| Non-clickable references | `See https://... for details.` | `See location metadata.` |

- `x-ms-summary` — **Required** on every parameter and schema property. Use Title Case, matching the parameter `name` but without hyphens or underscores (e.g., `name: "form_name"` → `x-ms-summary: "Form Name"`)
- `x-ms-visibility` — Controls visibility: `"important"` (always shown), `"advanced"` (hidden under menu), `"internal"` (hidden from user)
- **Response schemas** — Each operation should have only **one response with a schema**, which should be the `2XX` success response (200 or 201). The `default` response should **NOT** have a schema definition — schemas belong on expected success responses only. For error responses (`4xx`, `5xx`), provide meaningful descriptions but **remove the schema property**. Empty response schemas are not allowed (except when dynamic). Empty operations are not allowed — every operation must have at least one response
- **Boolean filters** — Parameters that represent true/false flags (for example `has_hrp`, `in_gho`, `is_hxl`, and origin/asylum variants) should use `"type": "boolean"` instead of `"string"`
- **Path parameters** — All path parameters (e.g., `/items/{itemId}`) **must** include `"x-ms-url-encoding": "single"` and **must** be marked `"required": true`
- **Reserved names** — A parameter cannot be named `connectionId` (reserved by the platform)
- **Swagger 2.0 parameter typing** — Every non-body parameter (`in: query`, `header`, `path`, `formData`) must include a `type` field. Missing `type` frequently causes APIM import failures such as `JSON is valid against no schemas from 'oneOf'`
- **GET operations** — Cannot have body or form data parameters
- **`collectionFormat: "multi"` is NOT supported** — The Custom Connector wizard rejects array parameters with `"collectionFormat": "multi"`. Workaround: change the parameter type from `array` to `string`, accept comma-separated values, and use custom code (`script.csx`) to split them into repeated query parameters. See [CUSTOM_CODE.md](references/CUSTOM_CODE.md) Pattern 5 and [COMMON_MISTAKES.md](references/COMMON_MISTAKES.md) entry #33
- **Remove empty properties** from operations and parameters unless they are explicitly required

**Recommended preflight lint before `pac connector create` / `pac connector update`:**
- Confirm every non-body parameter has a `type`
- Confirm every action/operation has a non-empty `description`
- Confirm path parameters are `required: true` with `x-ms-url-encoding: "single"`
- Confirm each operation has exactly one success response schema
- Confirm every `definitions.*.properties.*` entry has an explicit schema discriminator (`type`, `$ref`, `enum`, `anyOf`, `oneOf`, or `allOf`) so properties are never ambiguous during WADL conversion
- Confirm every array schema has explicit `items` typing/ref metadata (no `"items": {}` placeholders)
- Confirm no `readOnly: true` schema properties are listed in any definition `required` array
- Confirm there are no empty parameter/response/schema objects such as `{}` after automated transforms
- Confirm boolean-like flags (`has_*`, `in_*`, `is_*`) are typed as `boolean` where semantically correct
- Confirm parameter descriptions are concise helper text (no repetitive "Filter the response by" boilerplate)
- Parse-check the file before deploy (`python -c "import json;json.load(open('apiDefinition.swagger.json',encoding='utf-8'))"`) to catch malformed JSON and trailing/extra data
- Check for missing operation descriptions (`python -c "import json; s=json.load(open('apiDefinition.swagger.json',encoding='utf-8')); missing=[(m.upper(),p,o.get('operationId')) for p,ms in s.get('paths',{}).items() for m,o in ms.items() if isinstance(o,dict) and not (isinstance(o.get('description'),str) and o.get('description').strip())]; print('missing_description_count',len(missing)); [print(x) for x in missing[:200]]"`)
- If converting from OpenAPI 3.x, run a second pass to ensure nullable fields were translated into valid Swagger 2.0 schemas (avoid leaving properties with only `description`/`title`)
- Confirm nullable primitives use Swagger 2.0-compatible patterns (`type` + `x-nullable: true`) and avoid OpenAPI 3 style unions like `"type": ["integer", "null"]`

---

## Definitions (Schemas)

Define reusable data models in the `definitions` object. These are referenced via `$ref` in operation parameters and responses.

**Key rules:**
- All objects under `properties` **must** include both a `description` and `title` property, even when nested — **unless** the object contains a `$ref` property
- Every property schema must include an explicit schema discriminator (`type`, `$ref`, `enum`, `anyOf`, `oneOf`, or `allOf`); do not leave property definitions as metadata-only objects
- `title` — Must be in **Title Case**. Must not contain URLs
- `description` — Must be a **full sentence with proper punctuation**. Must not contain URLs
- Keep all other existing properties of the definition intact
- **Remove `default` values** from objects inside `properties` (defaults belong on operation parameters, not schema definitions)
- If a `number` or `integer` property's description mentions minimum or maximum values, add explicit `minimum` and/or `maximum` properties to the schema
- **Nullable fields (Swagger 2.0)** — Represent nullable scalar properties with `x-nullable: true` on the property schema. If upstream data can omit or null a field, remove that field from response `required` arrays. Do not use array `type` unions such as `["integer", "null"]`

```json
"definitions": {
  "Task": {
    "type": "object",
    "properties": {
      "id": {
        "type": "string",
        "title": "Task ID",
        "description": "The unique identifier for the task.",
        "x-ms-summary": "Task ID"
      },
      "name": {
        "type": "string",
        "title": "Task Name",
        "description": "The display name of the task.",
        "x-ms-summary": "Task Name"
      },
      "priority": {
        "type": "integer",
        "title": "Priority",
        "description": "The priority level of the task, from 1 (highest) to 5 (lowest).",
        "x-ms-summary": "Priority",
        "minimum": 1,
        "maximum": 5
      },
      "project": {
        "$ref": "#/definitions/Project"
      }
    }
  }
}
```

Note: The `project` property uses `$ref`, so it does **not** need its own `title` or `description`.

---

## OpenAPI Extensions Quick Reference

| Extension | Purpose | Applies To |
|-----------|---------|------------|
| `summary` | Action title in designer | Operations |
| `x-ms-summary` | Display name for entity | Parameters, schema properties |
| `description` | Verbose explanation | Operations, parameters, schemas |
| `x-ms-visibility` | `important` / `advanced` / `internal` | Operations, parameters, schemas |
| `x-ms-trigger` | `"single"` or `"batch"` — marks as trigger | Operations |
| `x-ms-trigger-hint` | Hint text for triggers | Operations |
| `x-ms-notification-content` | Webhook payload schema | Resources (path level) |
| `x-ms-notification-url` | Marks field as callback URL | Parameters |
| `x-ms-dynamic-values` | Populate dropdown from API call | Parameters |
| `x-ms-dynamic-list` | Enhanced dynamic dropdown (unambiguous refs) | Parameters |
| `x-ms-dynamic-schema` | Dynamic schema from API call | Parameters, responses |
| `x-ms-dynamic-properties` | Enhanced dynamic schema (unambiguous refs) | Parameters, responses |
| `x-ms-capabilities` | Test connection, chunk transfer | Connectors, operations |
| `x-ms-api-annotation` | Versioning: family, revision, replacement | Operations |
| `x-ms-url-encoding` | `"double"` or `"single"` for path params | Path parameters |
| `x-ms-connector-metadata` | Website, privacy policy, categories | Root level |
| `x-ms-no-generic-test` | Skip generic testing | Operations |
| `x-ms-safe-operation` | Mark POST as non-modifying | Operations |

**Copilot Studio / AI extensions:**

| Extension | Purpose |
|-----------|---------|
| `x-ms-name-for-model` | LLM-friendly operation name (snake_case) |
| `x-ms-description-for-model` | LLM-friendly usage description |
| `x-ms-media-kind` | `"image"` or `"audio"` for media operations |

**Extended format types:** `date-no-tz`, `email`, `html`, `uri`, `uuid`

See [OPENAPI_EXTENSIONS.md](references/OPENAPI_EXTENSIONS.md) for detailed examples of each extension.

### Dynamic Dropdown Pattern (Metadata Endpoints)

For connectors with metadata operations (for example, location/sector/org catalogs), use `x-ms-dynamic-values` so users can select valid values from dropdowns.

```json
{
  "name": "location_code",
  "in": "query",
  "type": "string",
  "required": false,
  "x-ms-summary": "Location Code",
  "description": "Filter the response by location code.",
  "x-ms-dynamic-values": {
    "operationId": "GetLocationApiVMetadataLocationGet",
    "parameters": {
      "output_format": "json",
      "limit": 10000,
      "offset": 0
    },
    "value-collection": "data",
    "value-path": "code",
    "value-title": "name"
  }
}
```

**Guidance:**
- Map frequently reused filters (location/admin1/admin2/sector/org/org-type/commodity/market) to metadata operations
- Use `value-collection` to match the response payload root (commonly `data`)
- Avoid applying dynamic-values on the same metadata source action itself
- Provide stable defaults for source operation parameters (`output_format`, `limit`, `offset`)

---

## Authentication Quick Reference

Configure auth in `apiProperties.json` under `connectionParameters` (or `connectionParameterSets` for multi-auth):

| Auth Type | `type` value | Identity Provider | Example |
|-----------|-------------|-------------------|---------|
| **API Key** | `securestring` | N/A | Most Independent Publisher connectors |
| **OAuth 2.0 (AAD)** | `oauthSetting` | `aad` | Azure services (Key Vault, Graph) |
| **OAuth 2.0 (Generic)** | `oauthSetting` | `oauth2` / `oauth2generic` | GitHub, Slack, Spotify |
| **Basic Auth** | `securestring` (x2) | N/A | Username + password |
| **Multi-Auth** | `connectionParameterSets` | Mixed | Multiple auth options for one connector |

**Note:** Multi-auth connectors use `connectionParameterSets` instead of `connectionParameters` and are **not supported in the Custom Connector Wizard** — use the `pac connector` or `paconn` CLI.

**API Key example (apiProperties.json):**

```json
{
  "properties": {
    "connectionParameters": {
      "api_key": {
        "type": "securestring",
        "uiDefinition": {
          "constraints": { "clearText": false, "required": "true", "tabIndex": 2 },
          "description": "Your API key from the service dashboard",
          "displayName": "API Key",
          "tooltip": "Provide your API key"
        }
      }
    },
    "iconBrandColor": "#da3b01",
    "capabilities": [],
    "publisher": "Your Name",
    "stackOwner": "Service Company Name"
  }
}
```

**Critical:** Independent Publisher `iconBrandColor` **must** be `"#da3b01"`.

See [AUTH_PATTERNS.md](references/AUTH_PATTERNS.md) for OAuth 2.0, AAD, and Basic Auth patterns.

---

## Policy Templates

Policy templates transform requests/responses without custom code. Defined in `apiProperties.json`:

| Template ID | Purpose |
|------------|---------|
| `dynamichosturl` | Route to dynamic host URL from connection params |
| `setheader` | Inject request/response headers |
| `setqueryparameter` | Add default query parameters |
| `routerequesttoendpoint` | Redirect request to different path |
| `setproperty` | Set body property values |
| `pollingtrigger` | Configure polling trigger behavior |
| `updatenextlink` | Fix pagination nextLink routing |

```json
"policyTemplateInstances": [
  {
    "templateId": "setheader",
    "title": "Set Content-Type Header",
    "parameters": {
      "x-ms-apimTemplateParameter.name": "Content-Type",
      "x-ms-apimTemplateParameter.value": "application/json",
      "x-ms-apimTemplateParameter.existsAction": "override",
      "x-ms-apimTemplateParameter.newValue": "application/json"
    }
  }
]
```

See [POLICY_TEMPLATES.md](references/POLICY_TEMPLATES.md) for all templates with examples.

---

## Custom Code (script.csx)

C# code for request/response transformation when policy templates aren't sufficient.

```csharp
public class Script : ScriptBase
{
    public override async Task<HttpResponseMessage> ExecuteAsync()
    {
        if (this.Context.OperationId == "MyOperation")
            return await HandleMyOperation().ConfigureAwait(false);

        // Forward all other requests unchanged
        return await this.Context.SendAsync(
            this.Context.Request, this.CancellationToken
        ).ConfigureAwait(false);
    }
}
```

**Constraints:** .NET Standard 2.0 | 2-minute timeout | 1 MB max file size | One script per connector

### Common Pattern: Build Base64 Identifier from Connection Fields

When the upstream API expects a computed identifier (for example `base64("app_name:email")`), collect user-friendly fields in `connectionParameters` and derive the required value in `script.csx`.

```csharp
var appName = GetHeaderValue("app_name")?.Trim();
var email = GetHeaderValue("email")?.Trim();

if (!string.IsNullOrWhiteSpace(appName) && !string.IsNullOrWhiteSpace(email))
{
  var appIdentifier = Convert.ToBase64String(Encoding.UTF8.GetBytes($"{appName}:{email}"));
  // Inject into query/header as required by the API
  // Remove raw app_name/email headers before forwarding
}
```

**Guidance:**
- Use UTF-8 before Base64 encoding
- Trim values before encoding
- Remove raw source headers after deriving the computed identifier
- If accepted by the API, set both query and header forms for compatibility
- Prefer deterministic header injection via `policyTemplateInstances` + `setheader` using `@connectionParameters('...')`, then consume those headers in `script.csx`

Example `apiProperties.json` policy mapping:

```json
"policyTemplateInstances": [
  {
    "templateId": "setheader",
    "title": "app_name",
    "parameters": {
      "x-ms-apimTemplateParameter.name": "app_name",
      "x-ms-apimTemplateParameter.value": "@connectionParameters('app_name')",
      "x-ms-apimTemplateParameter.existsAction": "override",
      "x-ms-apimTemplate-policySection": "Request"
    }
  }
]
```

See [CUSTOM_CODE.md](references/CUSTOM_CODE.md) for full reference with examples.

---

## Webhook Triggers

Mark an operation as a webhook trigger with `x-ms-trigger` and define the notification payload:

```json
"/hooks": {
  "x-ms-notification-content": {
    "schema": { "$ref": "#/definitions/WebhookPayload" }
  },
  "post": {
    "operationId": "WebhookTrigger",
    "summary": "When an event occurs",
    "x-ms-trigger": "single",
    "x-ms-trigger-hint": "To see it work, create a new item in the service.",
    "parameters": [
      {
        "name": "body",
        "in": "body",
        "schema": {
          "type": "object",
          "properties": {
            "callbackUrl": {
              "type": "string",
              "x-ms-notification-url": true,
              "x-ms-visibility": "internal"
            }
          }
        }
      }
    ],
    "responses": { "201": { "description": "Created" } }
  }
}
```

**Requirements:** Must also define a DELETE operation so the platform can unregister webhooks. The API must return a `Location` header in the 201 response pointing to the webhook resource.

See [WEBHOOK_TRIGGERS.md](references/WEBHOOK_TRIGGERS.md) for complete patterns.

---

## Certification & Submission

See [CERTIFICATION.md](references/CERTIFICATION.md) for full step-by-step certification workflows, packaging instructions, and submission details for both publisher types.

### Connector Requirements (Both Publisher Types)

**Connector Title:**
- ≤30 characters, no restricted words (`API`, `Connector`, `Copilot Studio`)
- The `(Independent Publisher)` suffix goes on the **PR title**, not the connector title in the OpenAPI definition

**Connector Description:** 30–500 characters, ≤1000 total, no HTML tags.

**Icon (Verified Publisher only):** Custom icon required — 100×100 to 230×230 px, PNG, non-white/non-default background.

**OAuth:** Currently **unsupported** for Independent Publisher certification.

### Certification Process Summary

| Step | Independent Publisher | Verified Publisher |
|------|----------------------|-------------------|
| **1** | Verify connector doesn't already exist | Prepare connector artifacts + custom icon |
| **2** | Submit proposal PR (`dev` branch) | Run Solution Checker |
| **3** | Build connector after approval | Create `intro.md` artifact |
| **4** | Submit artifacts to same PR | Package solution zip + intro.md |
| **5** | Complete OneVet identity verification | Validate with ConnectorPackageValidator.ps1 |
| **6** | Certification review → deploy (~15 biz days, Fridays) | Upload to blob (SAS URL) → submit via Partner Center (~15 biz days, Fridays) |

**Office Hours:** Tuesdays 3:30–4:30 PM UTC with the Connector Certification Team.

### PR Checklist

- [ ] All files in correct directory (`independent-publisher-connectors/` or `certified-connectors/`)
- [ ] PR targets `dev` branch (never `master`)
- [ ] `apiDefinition.swagger.json` passes swagger validation (use Solution Checker)
- [ ] `apiDefinition.swagger.json` is valid JSON (no trailing/extra data) and contains no empty `{}` parameter/response/schema placeholders
- [ ] `apiProperties.json` matches schema
- [ ] `readme.md` / `intro.md` follows template (Publisher, Prerequisites, Operations, Credentials, Known Issues)
- [ ] No secrets or real API keys in any file
- [ ] Connector title is ≤30 characters, no restricted words ("API", "Connector", "Copilot Studio")
- [ ] Connector description is 30-500 characters
- [ ] Connector description is <= 1000 characters and contains no HTML tags
- [ ] Host URL is a **production** URL (no staging/dev/test URLs)
- [ ] Response schemas provided on success responses only (not on `default` response)
- [ ] All `definitions.*.properties.*` entries have explicit schema type/ref metadata (no ambiguous properties during OpenAPI→WADL conversion)
- [ ] `x-ms-connector-metadata` array present with Website, Privacy policy, Categories
- [ ] All summaries are ≤80 chars, end with alphanumeric, no slashes
- [ ] Every action/operation has a non-empty `description`
- [ ] All descriptions are full sentences ending in punctuation, no URLs
- [ ] Summary and description text are **not identical** for any operation or parameter
- [ ] All path parameters have `required: true` and `x-ms-url-encoding: "single"`
- [ ] All `operationId` values are PascalCase with no non-alphanumeric characters
- [ ] For OAuth connectors: `"redirectMode": "GlobalPerConnector"` (not `"Global"`)
- [ ] For Independent Publisher: `iconBrandColor` is `"#da3b01"`
- [ ] For Independent Publisher with OAuth: detailed instructions for creating the OAuth app
- [ ] At least **10 successful calls per operation** tested
- [ ] Screenshots of 3+ unique operations working in a Flow (Independent Publisher)
- [ ] All strings are in English, free of spelling/grammar errors
- [ ] JSON uses 4-space indentation, no trailing whitespace
- [ ] Package validated with [ConnectorPackageValidator.ps1](https://github.com/microsoft/PowerPlatformConnectors/blob/dev/scripts/ConnectorPackageValidator.ps1)

### CLI Deployment

Two CLI tools can deploy custom connectors. **Power Platform CLI (`pac`)** is the modern, recommended tool. **`paconn`** is the legacy Python-based tool still used in many existing guides.

| Feature | `pac connector` (Power Platform CLI) | `paconn` (Legacy Python CLI) |
|---------|--------------------------------------|------------------------------|
| **Install** | `winget install Microsoft.PowerPlatformCLI` or [VS Code extension](https://marketplace.visualstudio.com/items?itemName=microsoft-IsvExpTools.powerplatform-vscode) | `pip install paconn` |
| **Auth** | `pac auth create` (interactive, service principal, device code) | `paconn login` (device code only) |
| **Solution-aware** | Yes — `--solution-unique-name` flag | No |
| **Scaffold** | `pac connector init` generates starter files | N/A |
| **List connectors** | `pac connector list` | N/A |
| **Validate swagger** | N/A (use ConnectorPackageValidator.ps1) | `paconn validate --api-def ...` |
| **Status** | Actively maintained | Maintenance mode |

#### Power Platform CLI (`pac connector`)

```bash
# Install (Windows)
winget install Microsoft.PowerPlatformCLI

# Or install via dotnet
dotnet tool install --global Microsoft.PowerApps.CLI.Tool

# Authenticate
pac auth create --environment <environment-url-or-id>

# Scaffold a new connector (generates starter apiProperties.json)
pac connector init \
  --connection-template OAuthAAD \
  --generate-script-file \
  --generate-settings-file \
  --outputDirectory MyConnector

# List connectors in current environment
pac connector list

# Create a connector
pac connector create \
  --api-definition-file apiDefinition.swagger.json \
  --api-properties-file apiProperties.json

# Create with icon, custom code, and add to a solution
pac connector create \
  --api-definition-file apiDefinition.swagger.json \
  --api-properties-file apiProperties.json \
  --icon-file icon.png \
  --script-file script.csx \
  --solution-unique-name MySolution

# Update an existing connector
pac connector update \
  --api-definition-file apiDefinition.swagger.json \
  --api-properties-file apiProperties.json \
  --connector-id <connector-id>

# If connector uses custom code, ALWAYS include script file on update
pac connector update \
  --api-definition-file apiDefinition.swagger.json \
  --api-properties-file apiProperties.json \
  --script-file script.csx \
  --connector-id <connector-id>

# Download a connector's files
pac connector download \
  --connector-id <connector-id> \
  --outputDirectory ./MyConnector
```

#### paconn CLI (Legacy)

```bash
# Install the CLI
pip install paconn

# Authenticate
paconn login

# Create a connector
paconn create --api-def apiDefinition.swagger.json --api-prop apiProperties.json

# With custom code and icon
paconn create --api-def apiDefinition.swagger.json --api-prop apiProperties.json \
  --script script.csx --icon icon.png

# Update an existing connector
paconn update --api-def apiDefinition.swagger.json --api-prop apiProperties.json \
  --connector-id <connector-id>

# Validate swagger definition
paconn validate --api-def apiDefinition.swagger.json

# Download connector files
paconn download -e <environment-id> -c <connector-id>
```

> **Tip:** Both CLIs support a `settings.json` file to store environment, connector ID, and file paths — avoiding repetitive flags on every command. When using `paconn`, always download first as a backup before running `update`.

> **Reliability note:** `pac connector update` with `--script-file` can intermittently fail with `CustomScriptProvisioningFailed` (often surfaced as upstream 502). This is commonly transient; retry the same command after a short delay before changing connector files.

### README Template

```markdown
# Service Name
Short description of the service (2-3 sentences).

## Publisher: Your Name

## Prerequisites
- An account with [Service Name](https://service.com)
- An API key (obtained from Settings > API Keys)

## Supported Operations
### List Items
Retrieves all items from your account.

### Create Item
Creates a new item with the specified properties.

## Obtaining Credentials
Step-by-step instructions for getting API credentials.

## Known Issues and Limitations
- Rate limited to 100 requests per minute
- Maximum 1000 items per response

## Deployment Instructions
Run one of the following commands:
\`pac connector create --api-definition-file apiDefinition.swagger.json --api-properties-file apiProperties.json\`
or (legacy):
\`paconn create --api-def apiDefinition.swagger.json --api-prop apiProperties.json\`
```

---

## Pagination Support

For connectors to leverage Power Platform's **built-in paging**, the API must return responses following this pattern:

```json
{
  "nextLink": "https://api.example.com/items?page=2",
  "value": [
    { "id": "1", "name": "Item 1" },
    { "id": "2", "name": "Item 2" }
  ]
}
```

**Requirements:**
- `value` — Array of result items (required on every page)
- `nextLink` — Full URI to the next page (present only when more pages exist; omit on the final page)
- Return HTTP **200** for all paginated responses

When the last page is reached, omit `nextLink` entirely — Power Platform stops paging automatically.

**Important runtime caveat (custom code):**
- Do **not** rely on `script.csx` to reshape pagination payloads for built-in auto paging.
- In practice, follow-up requests triggered internally by platform auto-pagination may bypass custom response transformation logic.
- If page 1 is transformed to include `value`/`nextLink` but the upstream API returns a different shape on subsequent pages (for example `data`/`next`), built-in pagination can still fail.

**Recommended approach:**
- Prefer native upstream pagination contracts for built-in paging (`value` + `nextLink` on every page).
- If upstream payloads are non-standard and cannot be changed, implement manual pagination in the flow (Do Until) or create a dedicated custom action that fetches/aggregates pages explicitly.

**Pagination decision tree (use this before implementing):**
1. **Can the upstream API return `value` + `nextLink` on every page?**
  - **Yes** → Use built-in connector pagination.
  - **No** → Continue to step 2.
2. **Can `updatenextlink` policy safely map the API's paging model?**
  - **Yes** → Use `updatenextlink` + built-in pagination.
  - **No** → Continue to step 3.
3. **Need all records in a single action output?**
  - **Yes** → Build a dedicated custom "fetch all pages" action.
  - **No** → Use manual pagination in Flow (`Do Until`) and process page-by-page.

**Rule of thumb:** If pagination correctness depends on response reshaping in `script.csx`, do not use built-in auto-pagination as the primary design.

**If the API uses non-standard pagination** (e.g., `page`/`limit` query parameters, cursor-based, or offset-based), you have two options:
1. Use the `updatenextlink` **policy template** to rewrite the pagination URL into the `nextLink` format Power Platform expects
2. Build pagination logic in a **Power Automate flow** using a Do Until loop that increments the page parameter until no more results are returned

Add `limit` and `page` parameters to operations that support pagination:

```json
{
  "name": "limit",
  "in": "query",
  "type": "integer",
  "required": false,
  "x-ms-summary": "Page Size",
  "description": "The number of items to return per page."
},
{
  "name": "page",
  "in": "query",
  "type": "integer",
  "required": false,
  "x-ms-summary": "Page Number",
  "description": "The page number of results to retrieve."
}
```

---

## Using AI to Accelerate Development

Leverage AI assistants to generate boilerplate OpenAPI extensions and documentation. This is especially valuable when an API has many operations or parameters that each need `x-ms-summary`, `description`, and `title` attributes.

**Generating OpenAPI extensions prompt:**

> Acting as a Power Platform developer, I would like your assistance in writing a custom connector. I will provide each path for the API. Include the following:
> - A `summary` and `description` attribute for each path
> - A `description` and `x-ms-summary` attribute for each path parameter and response property; the `x-ms-summary` should read like a title for the name field
> - A `title`, `description`, and `x-ms-summary` attribute for each response property; the `title` and `x-ms-summary` will be the same
> - If the `name` attribute is used in the `description`, then update the description to use the new `title` attribute
>
> Please update the file with those additional attributes and provide it back to me. Here is the first path:
> ```json
> <paste path here>
> ```

**Generating README prompt:**

> Acting as a technical writer, create a README.MD file for the custom connector. Below is the template — do not deviate from it. When generating the operations, include all input attributes and use the friendly names.
>
> TEMPLATE:
> ```
> # {Connector Title}
> {Description from the OpenAPI info.description}
>
> ## Publisher: {Your Name}
>
> ## Prerequisites
> {How to get an account and API credentials}
>
> ## Supported Operations
> ### {Operation Summary}
> {Operation description}
> - **Inputs:** `{Param x-ms-summary}`: {param description}
> - **Outputs:** `{Property title}`: {property description}
>
> ## Obtaining Credentials
> {Step-by-step instructions}
>
> ## Known Issues and Limitations
> {Current limitations or "Currently, no known issues or limitations exist."}
> ```
>
> OPENAPI FILE:
> ```json
> <paste full apiDefinition.swagger.json>
> ```

**Tips:**
- Process paths individually for APIs with many operations — large files may exceed context limits
- Always review and validate AI output against the coding standards before submission
- AI works best when the source API has comprehensive documentation

---

## Testing and Debugging

After importing the connector into Power Platform via the Custom Connector Wizard or CLI:

1. **Test every operation** in the connector's Test tab — run at least **10 successful calls per operation**
2. **Use the Swagger Editor toggle** in the custom connector editor for quick inline edits to fix validation errors
3. **Common schema validation fixes:**
   - **Remove `required` arrays from response schemas** — if the API doesn't always return every field, the `required` constraint causes validation failures. Keep `required` on request body schemas but remove from response schemas
   - **Fix type mismatches** — if the API returns a string where the schema says integer (or vice versa), update the schema to match actual API behavior
   - **Remove empty schema properties** — empty `properties: {}` on responses can cause issues
4. **Re-test after every change** — iterate until all operations pass cleanly
5. **Create test flows** in Power Automate using 3+ unique operations to verify end-to-end behavior and capture screenshots for PR submission

### Post-Deploy Verification (Recommended)

After `pac connector create` or `pac connector update`:
1. Download the deployed connector files using `pac connector download --connector-id <id>`
2. Perform semantic JSON comparison between local and downloaded files
3. Focus on functional parity for:
  - OpenAPI paths/parameters/responses
  - `x-ms-dynamic-values` mappings
  - `connectionParameters` and script wiring
4. Treat these as common platform-managed non-functional differences unless behavior changed:
  - `policyTemplateInstances` may be auto-added
  - `publisher` or `stackOwner` may differ/appear blank in downloaded `apiProperties.json`

---

## Best Practices

**Do:**
- Use `x-ms-summary` on every parameter and schema property (controls designer labels)
- Use `x-ms-visibility: "internal"` for parameters with fixed values (e.g., `api-version`)
- Provide default values for all `internal` required parameters
- Include complete response schemas on **success responses only** (enables dynamic content in flows)
- Use `$ref` definitions and `$ref` parameters for reusable schemas and shared parameters
- Test with the custom connector wizard before submitting PR — run **at least 10 successful calls per operation**
- Run **Solution Checker** and **ConnectorPackageValidator.ps1** before submitting
- Use policy templates before reaching for custom code
- Use PascalCase for `operationId` — remove all non-alpha characters
- Keep `summary` to 80 characters or fewer, sentence case, ending with an alphanumeric character
- Ensure summaries contain only **alphanumeric characters or parentheses** — no slashes
- Start summaries with **"List"** (returns many) or **"Get"** (returns one); for triggers: **"When a [event]"**
- Make sure `summary` and `description` have **different text** on every operation/parameter
- Write `description` as full sentences ending in punctuation — no URLs
- Keep parameter helper text concise for designer UX; avoid repetitive boilerplate and long non-clickable references
- Add `title` (Title Case) and `description` (full sentence) to all definition properties (skip for `$ref`)
- Add `"x-ms-url-encoding": "single"` and `"required": true` to all path parameters
- Add `minimum`/`maximum` when number/integer ranges are known
- Use `type: "boolean"` for true/false inputs instead of `string` flags
- For nullable response primitives, use `x-nullable: true` and remove from `required` when null/omitted is possible
- Ensure all text is in English and free of grammatical/spelling errors
- Capitalize abbreviations to avoid translation issues
- Use `"redirectMode": "GlobalPerConnector"` for all OAuth connectors
- Use **4-space soft tabs** and remove trailing whitespace in JSON files
- Use only **production** host URLs (no staging, dev, or test URLs)
- Explicitly set `consumes` and `produces` to `["application/json"]` — don't rely on defaults
- Back up connector files using source control before running `update` commands
- For APIs with multiple cloud endpoints (e.g., commercial, GCC, GCC High, DoD), use the `dynamichosturl` policy template to let users select the correct endpoint at connection time

**Don't:**
- Use OpenAPI 3.0 — must be Swagger 2.0
- Include real API keys or secrets in files (use dummy values: `<<Enter your API key>>`)
- Submit PR to `master` branch (always target `dev`)
- Leave `x-ms-connector-metadata` empty or missing
- Use wrong brand color for Independent Publisher (must be `#da3b01`)
- Skip response schemas (breaks dynamic content in Power Automate)
- Put schemas on the `default` response — schemas belong on explicit success responses (200/201) only
- Exceed 1 MB for the OpenAPI definition file
- Put schemas on error responses (4xx/5xx) — descriptions only
- Leave `default` values on definition properties (only on operation parameters)
- Include empty properties unless they are required
- Put URLs in `title` or `description` fields
- Name any parameter `connectionId` (reserved by the platform)
- Include body or form data parameters on GET operations
- Use `"redirectMode": "Global"` — must be `"GlobalPerConnector"` (mandatory since Feb 2024)
- Exceed 30 characters for connector title
- Include `required` arrays on response schemas — they cause validation failures when the API omits optional fields
- Use OpenAPI 3 nullable union syntax in Swagger 2.0 files (for example `"type": ["integer", "null"]`)

**Known limitation:** When using `paconn`, the `stackOwner` property in `apiProperties.json` prevents `paconn update` from working. Workaround: maintain two versions of your apiProperties — one with `stackOwner` for certification submission and one without for local environment updates via `paconn`. The `pac connector` CLI does not have this limitation.

See [COMMON_MISTAKES.md](references/COMMON_MISTAKES.md) for a full error catalog with fixes.

---

## Session Learnings

Use this section as a lightweight changelog for practical field learnings that should influence future connector work.

### 2026-02-18: HDX Connector Session

- **Swagger 2.0 strictness:** Non-body parameters without `type` caused APIM import failures (`JSON is valid against no schemas from 'oneOf'`); adding explicit `type` resolved deployment.
- **Metadata-driven UX:** Mapping metadata endpoints to `x-ms-dynamic-values` significantly improved usability by enabling dropdowns for common filter parameters.
- **Computed auth pattern:** Collecting `app_name` + `email` as connection parameters and computing `app_identifier` in `script.csx` (UTF-8 Base64) produced a cleaner auth experience.
- **Deterministic parameter flow:** For custom code that reads connection values from headers, explicitly map with `policyTemplateInstances` + `setheader` (`@connectionParameters('app_name')`, `@connectionParameters('email')`) instead of relying on implicit runtime behavior.
- **Input normalization:** Trimming connection values before encoding prevented malformed identifiers from accidental whitespace.
- **Operational reliability:** `pac connector update` with custom code can intermittently fail with `CustomScriptProvisioningFailed`/502; retrying the same command after a short delay often succeeds.
- **Deployment discipline:** When a connector has `script.csx`, include `--script-file script.csx` on every `pac connector update`; otherwise code changes might not be published with schema updates.
- **Verification approach:** Post-deploy semantic comparison is more reliable than byte-for-byte diff because downloaded files may include platform-managed differences.
- **WADL hardening:** In addition to property typing, Power Apps WADL conversion also requires typed array `items` and rejects `readOnly` properties in `required` arrays.

### 2026-02-25: HDX Connector UX & Nullability Session

- **Input description UX:** Power Automate surfaces parameter `description` as helper text; concise descriptions materially improve usability. Prefer short guidance over repeated phrases like "Filter the response by...".
- **No dead links in helper text:** URLs/HTML anchors in parameter descriptions are not useful in designer helper text; remove them from input descriptions.
- **Enum helper strategy:** For short enums, include values inline (`Allowed values: ...`). For long enums shown as dropdowns, use concise text such as "Select from dropdown.".
- **Boolean correctness:** Converted boolean-like query flags (`has_hrp`, `in_gho`, `origin_*`, `asylum_*`, `is_hxl`) from `string` to `boolean` to align schema with runtime semantics.
- **Swagger 2.0 nullable safety:** Avoid `type` arrays (for example `["integer", "null"]`) because they can pass some validators but fail connector import/parsing; use `x-nullable: true` with scalar `type` instead.
- **Required vs nullable:** If upstream API returns `null` or omits fields, remove those fields from response `required` arrays rather than coercing values (for example, avoid null→0 data mutation unless explicitly desired).

### 2026-02-26: HDX Auto-Pagination Session

- **Auto-pagination + custom code caveat:** Built-in platform auto-pagination may execute follow-up page calls outside custom `script.csx` response-shaping assumptions.
- **Design implication:** Do not depend on custom code to retrofit page 2+ into `value`/`nextLink` for built-in pagination.
- **Preferred fallback:** Use manual pagination (Do Until) or a dedicated "fetch all pages" action when the upstream API cannot natively emit Power Platform paging shape on every page.

---

## Troubleshooting

| Symptom | Likely Cause | Fix |
|---------|-------------|-----|
| `JSON is valid against no schemas from 'oneOf'` | Non-body parameter missing `type` | Add `type: "string"` (or appropriate type) to every query/header/path/formData parameter |
| `DefaultResponseHasSchema` | Schema on `default` response | Move schema to the `200`/`201` response; remove schema from `default` |
| `CustomScriptProvisioningFailed` / 502 | Transient deployment error | Retry the same `pac connector update` command after a short delay |
| Dynamic content missing in Power Automate | No response schema on success response | Add a complete schema to the `200` response |
| `PathParameterMustBeRequired` | Path parameter missing `required: true` | Add `"required": true` and `"x-ms-url-encoding": "single"` |
| `ConnectionIdParameterNotAllowed` | Parameter named `connectionId` | Rename the parameter (e.g., `connId`) |
| `The 'collectionFormat' value 'Multi' is not supported` | Array parameter with `collectionFormat: "multi"` | Change to `type: "string"`, accept comma-separated values, split in `script.csx` |
| `An error occured while converting OpenAPI file to WADL file` | Ambiguous definition properties (missing `type`/`$ref`) | Ensure every `definitions.*.properties.*` has an explicit schema discriminator |
| Properties show raw technical names in designer | Missing `x-ms-summary` or `title` | Add `x-ms-summary` to parameters; add `title` and `description` to definition properties |
| Double-encoded path parameters | Missing `x-ms-url-encoding` | Add `"x-ms-url-encoding": "single"` to all path parameters |
| OAuth connector certification rejected | Using `redirectMode: "Global"` | Change to `"redirectMode": "GlobalPerConnector"` |
| Connector title rejected | Title > 30 chars or contains "API"/"Connector" | Shorten title and remove restricted words |

For the full error catalog with detailed fixes and code examples, see [COMMON_MISTAKES.md](references/COMMON_MISTAKES.md).

---

## Related Files

- [OPENAPI_EXTENSIONS.md](references/OPENAPI_EXTENSIONS.md) — Detailed reference for all `x-ms-*` extensions with examples
- [AUTH_PATTERNS.md](references/AUTH_PATTERNS.md) — API Key, OAuth 2.0 (AAD/Generic), Basic Auth, and Multi-Auth patterns
- [POLICY_TEMPLATES.md](references/POLICY_TEMPLATES.md) — All policy template IDs with configuration examples
- [CUSTOM_CODE.md](references/CUSTOM_CODE.md) — C# `script.csx` patterns, ScriptBase class, supported namespaces
- [WEBHOOK_TRIGGERS.md](references/WEBHOOK_TRIGGERS.md) — Webhook trigger registration, notification, and deletion patterns
- [CERTIFICATION.md](references/CERTIFICATION.md) — Full certification workflows for Independent and Verified Publishers, packaging, and submission
- [EXAMPLES.md](references/EXAMPLES.md) — Complete working connector examples for common scenarios
- [COMMON_MISTAKES.md](references/COMMON_MISTAKES.md) — Error catalog with fixes and validation tips
