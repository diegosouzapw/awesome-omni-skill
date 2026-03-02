---
name: api-use-external
description: Any time an external api needs to be understood or queried
---

# External API use

Includes how to query and process the current existing external APIs:
- IMDb Primary API.
- IMDb Fallback API - To be used when IMDb primary API times out or is otherwise unreachable.

## Index
- **API configuration**: 
- **API provider framework**: 
- **Docs and examples**: documenation, endpoints, and request/response examples for both IMDb Primary API and IMDb Fallback API. 

## API configuration
- All API settings such as keys, base URLs, and timeouts come from `config.py`.
- API credentials and other secrets come from `.env`.

## API provider framework

### API integration
- IMDb integration must remain active and fully operational.
    - Core implementation is in `utils/imdb_core.py` using RapidAPI-based providers.
    - Core function families include `imdb_details_fetch_core`, `imdb_search_core`, `imdb_rating_core`, `imdb_import_core`, `imdb_api_search_title`, and `imdb_api_get_details`.
- Provider configuration switches are defined in `config.py`:
    - `MEDIA_ENABLE_IMDB = True` must remain enabled.

### Base API provider
- `utils/api_provider_core.py` defines `BaseApiProvider` for all external HTTP APIs.
- Handles HTTP requests, authentication, token management, error handling, and response normalization.
- Stateless by design with no Flask context; safe to use in core modules.
- Includes retry strategy and standardized response format.
- Use this class instead of creating custom HTTP client code per API.

### Provider descriptors
- `utils/api_provider_descriptors.py` defines configuration descriptors for each external API.
- New providers should be added by defining a descriptor in this module, not by creating new ad-hoc client modules.
- Descriptors capture base URL, authentication strategy, header builders, and normalization callbacks.
- Supports bearer tokens, API keys, and static tokens.
- Includes `TokenCache` for automatic token refresh and caching where needed.

### Core vs presentation for APIs
- Core API logic belongs in `utils/*_core.py` modules that use BaseApiProvider and descriptors.
- Flask-aware presentation logic belongs in `routes/utils_*.py` modules that call core functions.
- Keep API core code free of Flask imports and request context.

## Docs and examples
- **IMDb primary API**: `utils/imdb-01-docs.md` (IMDb236 API)
- **IMDb fallback API**: `utils/imdb-02-docs.md` (IMDb8 API)

## Legacy unused but saved
**Do not use or enable but do save**:
- TVDB and TMDB are not currently enabled in the application.
    - Supporting code and tests exist but must remain disabled unless explicitly authorized.
    - Residual assets include templates under `templates/admin`, JavaScript under `static/js`, and utilities under `utils/api_tvdb` and `utils/api_tmdb`.
    - Runtime disabling is enforced in routes and config.
    - Presence does not imply enablement; do not enable or delete them without approval.
- Provider configuration switches are defined in `config.py`:
    - `MEDIA_ENABLE_TVDB = False` must stay disabled for now.
    - `MEDIA_ENABLE_TMDB = False` must stay disabled for now.