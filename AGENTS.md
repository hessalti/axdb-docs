# Repository Guidelines

## Project Structure & Module Organization

This repository contains the source for Percona Distribution for PostgreSQL documentation. The main content lives in `docs/` as Markdown files, with nested topic groups such as `docs/docker/`, `docs/solutions/`, and `docs/release-notes/`. Reusable Markdown fragments are stored in `snippets/`. MkDocs configuration is split across `mkdocs.yml`, `mkdocs-base.yml`, and `mkdocs-pdf.yml`. Theme overrides and PDF-specific templates live in `_resource/` and `_resourcepdf/`; CSS, JavaScript, fonts, and PDF templates are under `docs/css/`, `docs/js/`, `docs/fonts/`, and `docs/templates/`.

## Build, Test, and Development Commands

Install documentation dependencies from the repository root:

```sh
pip install -r requirements.txt
```

Build the static site and validate MkDocs configuration:

```sh
mkdocs build
```

Run a local preview server with live reload:

```sh
mkdocs serve
```

Open the preview at `http://127.0.0.1:8000`. Build output is written to `site/`; do not edit generated files there.

## Coding Style & Naming Conventions

Use Markdown for documentation pages. Keep headings sentence-style and concise, and prefer short paragraphs plus command examples where they help users complete a task. Follow existing file naming: lowercase words separated by hyphens, for example `minor-upgrade.md` or `docker-enable-pg-tde.md`. Add new pages to the appropriate `nav` section in `mkdocs.yml` when they should appear in the published navigation. Keep shared text in `snippets/` when it is reused across pages.

## Testing Guidelines

There is no separate unit test suite for this repository. Treat `mkdocs build` as the primary validation step before opening a pull request. For navigation, formatting, links, macros, and include changes, also run `mkdocs serve` and inspect the affected pages in a browser. When changing release notes, verify dates, version numbers, and links against the source issue or release material.

## Commit & Pull Request Guidelines

Git history commonly uses short imperative summaries and Jira references such as `PG-2312 - Reorganize Docker installation instructions (#957)`. When work is tied to Jira, use `PG-123 - concise description` in branch and commit names. Pull requests should describe the documentation change, link the Jira issue when applicable, list validation performed, and include screenshots for visual or layout changes.

