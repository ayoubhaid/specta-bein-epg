# Specta Curated EPG

## Goal

Build one EPG feed from programme data collected by independent provider adapters, restricted to the channel inventory exported by Specta. The playlist itself is never ingested by this repository: Specta exports channel metadata as XML, without stream URLs, provider credentials, or playlist URLs.

## Data flow

1. Specta exports `channel-inventory.xml` using the contract in [channel-inventory.example.xml](channel-inventory.example.xml).
2. A private delivery mechanism supplies that XML to the scheduled build. Do not commit a real inventory to this public repository.
3. The inventory validator checks required identifiers and rejects duplicate inventory IDs.
4. Provider adapters collect schedule metadata from configured sources.
5. A reviewed mapping file connects Specta inventory IDs to provider channel IDs. Automatic name matching is a suggestion only; ambiguous matches require review.
6. The merger emits a curated XMLTV guide and a Specta JSON feed. Only inventory channels with mapped programme data are emitted.
7. Validation checks XML well-formedness, time ranges, duplicate programmes, channel references, and coverage before publication.

## Identity and matching rules

- `inventoryId` is Specta's stable channel identity. It must remain stable when the stream URL changes.
- `tvgId` is retained when available and is the preferred external matching key.
- Provider channel IDs are namespaced by provider; never assume IDs are globally unique.
- Display names are not unique identifiers. Normalized names and aliases may generate candidate matches, but must not silently override explicit mappings.
- Duplicate display names with different Specta IDs remain separate inventory entries.
- Stream URLs, usernames, passwords, tokens, and playlist URLs are prohibited from the export.

## XML inventory contract

See [channel-inventory.example.xml](channel-inventory.example.xml). The XML is an inventory export, not an XMLTV schedule. It contains channel metadata only. XMLTV is a separate generated output.

Required per channel: `id` and `name`. Optional metadata: `tvg-id`, `group`, `language`, `country`, and `aliases`.

## Repository layout (target)

```
src/
  inventory/       XML parsing, validation, normalization
  providers/       one adapter per schedule source
  matching/        explicit mappings and candidate generation
  output/          XMLTV and Specta JSON serializers
  validation/      feed and coverage checks
config/
  channel-mapping.json
docs/
  curated-epg/
    channel-inventory.example.xml
    README.md
```

## Output contract

- `docs/guide.xml`: XMLTV-compatible feed.
- `docs/epg.json`: versioned Specta JSON feed.
- Both outputs must contain the same curated channel IDs and programme intervals.
- All timestamps are normalized to UTC ISO-8601 in JSON and XMLTV date-time format in XML.
- A failed or empty provider run must never replace the last known-good published feed.

## Privacy and deployment

This repository is public. Never commit the actual TREX-derived inventory if it could reveal private subscription/channel information, and never include stream URLs or credentials. Supply the generated XML to CI through a private repository/artifact or a protected secret-backed delivery mechanism. Publishing the resulting schedule feed is a separate, explicit deployment decision.

## Rollout

1. Agree the XML export contract with Specta.
2. Implement inventory parsing and validation.
3. Add explicit channel mappings and review reports.
4. Adapt the existing beIN MENA collector as the first provider adapter.
5. Add further providers only after validating their data quality and access conditions.
6. Generate, validate, and publish XMLTV and JSON atomically.
