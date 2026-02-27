---
name: github-repo-skill
description: Guide for creating new GitHub repos and best practice for existing GitHub repos, applicable to both code and non-code projects
license: CC-0
---

# github-repo-skill

## overview

To create and maintain high-quality repos that conform to Mungall group / BBOP best practice, use this skill. Use this skill regardless of whether the repo is for code or non-code (ontology, linkml schemas, curated content, analyses, websites). Use this skill for both new repos, for migrating legacy repos, or for ongoing maintenance.

---

# Principles

## Follow existing copier templates

The Mungall group favors the use of [copier](https://copier.readthedocs.io/) and blesses the following templates:

* For LinkML schemas: https://github.com/linkml/linkml-project-copier
* For code: https://github.com/monarch-initiative/monarch-project-copier
* For ontologies: https://github.com/INCATools/ontology-development-kit (uses bespoke framework, not copier)

These should always be used for new repos. Pre-existing repos should try and follow these or migrate towards them.

Additionally the group uses additional drop-in templates for AI integrations:

* https://github.com/ai4curation/github-ai-integrations

## Favored tools

These are included in the templates above but some general over-arching preferences:

* modern python dev stack: `uv`, `ruff` (currently `mypy` for typing but we may switch to  https://docs.astral.sh/ty/)
* for builds, both `just` and `make` are favored, with `just` favored for non-pipeline cases

## Engineering best practice

* pydantic or pydantic generated from LinkML for data models and data access objects (dataclasses are fine for engine objects)
* always make sure function/method parameters and return objects are typed. Use mypy or equivalent to test.
* testing:
   * follow TDD, use pytest-style tests, `@pytest.mark.parametrize` is good for combinatorial testing
   * always use doctests: make them informative for humans but also serving as additional tests
   * ensure unit tests and tests that depend on external APIs, infrastructure etc are separated (e.g. `pytest.mark.integration`)
   * for testing external APIs or services, use vcrpy
   * do not create mock tests unless explicitly requested
   * for data-oriented projects, yaml, tsvs, etc can go in `tests/input` or smilar
   * for schemas, follow the linkml copier template, and ensure schemas and example test data is validated
   * for ontologies, follow ODK best practice and ensure ontologies are comprehensively axiomatized to allow for reasoner-based checking
* jupyter notebooks are good for documentation, dashboards, and analysis, but ensure that core logic is separated out and has unit tests
* CLI:
   * Every library should have a fully featured CLI
   * typer is favored, but click is also good.
   * CLIs, APIs, and MCPs should be shims on top of core logic
   * have separate test for both core logic and CLIs.
   * Use idiomatic options and clig conventions. Group standards: `-i/--input`, `-o/--output` (default stdout), `-f/--format` (input format), `-O/--output-format`, `-v/-vv`
   * When testing Typer/Rich CLIs, set `NO_COLOR=1` and `TERM=dumb` env vars to avoid ANSI escape codes breaking string assertions in CI.
* Exceptions
   * In general you should not need to worry about catching exceptions, although for a well-polished CLI some catching in the CLI layer is fine
   * IMPORTANT: better to fail fast and know there is a problem than to defensively catch and carry on as if everything is OK (general principle: transparency)

## Dependency management

* `uv add` to add new dependencies (or `uv add --dev` or similar for dev dependencies)
* libraries should allow somewhat relaxed dependencies to avoid diamond dependency problems. applications and infra can pin more tightly.

## Git and GitHub Practices

* always work on branches, commit early and often, make PRs early
* in general, one PR = one issue (avoid mixing orthogonal concerns). Always reference issues in commits/PR messages
* use `gh` on command line for operations like finding issues, creating PRs
* all the group copier templates include extensive github actions for ensuring PRs are high quality
* github repo should have adequate metadata, links to docs, tags

## Source of truth

* always have a clear source of truth (SoT) for all content, with github favored
* where projects dictate SoT is google docs/sheets, use https://rclone.org/ to sync

## Documentation

* markdown is always favored, but older sites may use sphinx
* Follow [Diátaxis framework](https://diataxis.fr/): tutorial, how-to, reference, explanation
* Use examples extensively - examples can double as tests
* frameworks: mkdocs is generally favored due to simplicity but sphinx is ok for older projects
* Every project must have a comprehensive up to date README.md (or the README.md can point to site generated from mkdocs)
* jupyter notebooks can serve as combined integration tests/docs, use mkdocs-jupyter, for CLI examples, use `%%bash`
* Formatting tips: lists should be preceded by a blank line to avoid formatting issues with mkdocs
