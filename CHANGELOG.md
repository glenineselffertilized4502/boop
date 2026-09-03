# Changelog

All notable changes to Boop are recorded here. The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/); versions follow [Semantic Versioning](https://semver.org/). The server, web UI and iOS app share one version number.

## [Unreleased]

## [1.3.0] — 2026-08-30

### Added
- **Sentry SDK compatibility.** Point any server-side Sentry SDK's DSN at Boop (`https://boop_proj_...@your-boop-host/1`) and it reports straight into that project with no code changes. `POST /api/{id}/envelope/` accepts Sentry envelopes (gzip/deflate included), authenticated via `X-Sentry-Auth` or `?sentry_key=`. Each Sentry event becomes a Boop event: `Type: value` title, culprit and top stack frame in the body, levels mapped (`fatal→critical`, `info`/`debug→info`), `source: "sentry"`, and grouping fingerprints (explicit ones, or exception type plus top in-app frame) so silence rules work per error group. Transactions and sessions are accepted and ignored. Keep the DSN server-side: its public key is a write-capable project API key. Contributed by [@ashwin47](https://github.com/ashwin47).
- **Pre-built binaries.** Tagged releases now attach static, CGO-free binaries for Linux, macOS and Windows (amd64/arm64) with the web UI embedded, plus a `checksums.txt`. Tests run before anything is published (`.github/workflows/release.yml`).
- **Laravel client.** [`laravel-boop`](https://github.com/solutionforest/laravel-boop), a community package by Solution Forest, is listed under Integrations.

### Changed
- iOS: cleaner app icons.

## [1.2.0] — 2026-08-29

### Added
- **Grouping by fingerprint.** `GET /api/v1/events?grouped=true` collapses events that share a `fingerprint` within a project into their latest occurrence, annotated with `group: {count, first_seen, last_seen}`. The web inbox and the iOS app show one row per fingerprint (`KeyError ×47 · First seen 09:31 · Last seen 10:42`) with a **Group repeats** toggle; opening a grouped row lists the occurrences (web: `/groups/:project/:fingerprint`; iOS: a pushed screen). Filters apply inside a group; fingerprints never merge across projects.
- **Actions on events.** `POST /api/v1/events` accepts `actions: [{label, url}]` (up to 3; label ≤ 40 chars; absolute URL, `javascript:`/`data:`/`file:` refused). Actions render as buttons in the web and iOS event detail and as buttons on the push notification. New `BoopNotificationService` extension target in the iOS project registers the buttons per notification; tapping one opens its URL.
- **Copy for an agent (iOS).** The share button on an event is now a menu: **Copy** (plain text), **Copy as Markdown** (sectioned: exception, environment, stack trace, context, breadcrumbs, data, links) and **Share**.
- **MCP endpoint (read-only).** `/mcp` speaks the Model Context Protocol over Streamable HTTP using the official Go SDK. Tools: `list_projects`, `list_events`, `search_events`, `get_event`, `get_event_group`. Auth: `BOOP_MCP_TOKEN` bearer, a device credential, or the admin login; project keys are refused. **Settings → MCP** switch (`mcp_enabled`) turns the endpoint off.
- `GET /api/v1/events` filters: `fingerprint`, `since`, `until` (RFC 3339, on `created_at`).
- `BOOP_MCP_TOKEN` configuration variable (16+ characters).
- `GET /api/v1/settings` reports `mcp_enabled` and `mcp_token_set`.

### Changed
- Projects page: each project is a compact row with its settings button inline.
- Inbox: level chips share one width and titles/bodies truncate before the level column.
- Database migration `0003_actions_groups`: adds `events.actions_json` and an index on `(project_id, fingerprint, created_at)`. Applied automatically on start.
- iOS: the project file is regenerated from `project.yml` and now contains the notification service extension; its bundle id must be `<app bundle id>.NotificationService`.

## [1.1.0] — 2026-08-28

### Added
- **Silences.** Rules that stop matching events (by fingerprint, title or source, per project or global) from being pushed while still storing them. Create from an event's page or **Settings → Silences**; filter the inbox to silenced events; unsilence and push on demand. `GET/POST/DELETE /api/v1/silences`, `POST /api/v1/events/:id/unsilence`, `silenced` filter on `GET /api/v1/events`.
- README: integrations section covering `boop_ex`, `boop_error_tracker` and `@boop/node`.

## [1.0.0] — 2026-08-23

### Added
- Go server with SQLite storage, embedded Svelte web UI, direct APNs delivery (ES256 token auth), project API keys, device pairing by QR, redaction of sensitive keys, admin login (`BOOP_ADMIN_USER`/`BOOP_ADMIN_PASSWORD`), setup wizard and test notifications.
- Event retention managed from Settings and persisted; `BOOP_RETENTION_DAYS` overrides it on every start when set.
- iOS app (SwiftUI, iOS 26): inbox with filters and cursor pagination, rich event detail (exception, stacktrace, tags, context, breadcrumbs, raw JSON), notification tap-through, Keychain-stored device credential.
- Docker image, compose file and a Dokploy-specific compose file; architecture diagram (`docs/architecture`); `integration-llms.md` client spec.

[Unreleased]: https://github.com/chrisgreg/boop/compare/v1.3.0...HEAD
[1.3.0]: https://github.com/chrisgreg/boop/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/chrisgreg/boop/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/chrisgreg/boop/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/chrisgreg/boop/releases/tag/v1.0.0
