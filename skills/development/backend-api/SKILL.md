---
name: backend-api
description: Manage items, projects, and references in Senticor Project via REST API
user-invokable: false
---

# Backend API for Senticor Project

Du hast vollen Zugriff auf die Senticor Project Backend-API.
Verwende `exec` mit `curl`, um Items zu lesen, erstellen und aktualisieren.

## Authentifizierung

Der aktuelle Token liegt in `/runtime/token`. Verwende ihn so in jedem Request:

```
Authorization: Bearer $(cat /runtime/token)
```

Die Backend-URL liegt in der Umgebungsvariable `COPILOT_BACKEND_URL`.
Weitere Umgebungsvariablen:
- `COPILOT_FRONTEND_URL` — Frontend-URL (z.B. fuer Kontext)
- `COPILOT_STORYBOOK_URL` — Storybook mit Produkt-, Design- und Engineering-Dokumentation

## Umgebung erkennen

Pruefe deine Umgebungsvariablen:

```bash
exec echo "Backend: $COPILOT_BACKEND_URL | Frontend: $COPILOT_FRONTEND_URL | Storybook: $COPILOT_STORYBOOK_URL"
```

## API-Dokumentation entdecken

Die vollstaendige OpenAPI-Spezifikation ist verfuegbar unter:

```bash
exec curl -s "$COPILOT_BACKEND_URL/openapi.json" | head -c 5000
```

Verwende sie, um alle verfuegbaren Endpoints, Parameter und Modelle zu entdecken.
Wichtige Pfade: `/items`, `/items/sync`, `/items/{item_id}`, `/files`, `/search`,
`/imports/nirvana/*`, `/imports/native/*`, `/imports/jobs`, `/email/*`, `/calendar/*`.

Fuer Import/Export-Details: Lies `/workspace/skills/import-export/SKILL.md`.
Fuer E-Mail/Kalender-Details: Lies `/workspace/skills/email-calendar/SKILL.md`.

## Storybook (Produktdokumentation)

Das Storybook enthaelt die gesamte Produkt-, Design- und Engineering-Dokumentation:

```bash
exec curl -s "$COPILOT_STORYBOOK_URL/index.json" | head -c 3000
```

Dort findest du Informationen zu:
- Produktvision und Methodik
- Design-System und Paperclip-Prinzipien
- Datenmodell und Architektur
- Verfuegbare UI-Komponenten

## Items lesen

```bash
exec curl -s "$COPILOT_BACKEND_URL/items?limit=200" \
  -H "Authorization: Bearer $(cat /runtime/token)"
```

Antwort: Array von Items mit `item_id`, `canonical_id`, `item` (JSON-LD mit `name`, `@type`, `additionalProperty`).

Den Bucket findest du in `item.additionalProperty` unter `propertyID: "app:bucket"`.

Einzelnes Item:

```bash
exec curl -s "$COPILOT_BACKEND_URL/items/<item_id>" \
  -H "Authorization: Bearer $(cat /runtime/token)"
```

## Item aktualisieren (Bucket aendern, umbenennen)

```bash
exec curl -s -X PATCH "$COPILOT_BACKEND_URL/items/<item_id>" \
  -H "Authorization: Bearer $(cat /runtime/token)" \
  -H "Content-Type: application/json" \
  -H "X-Agent: openclaw" \
  -d '{
    "source": "openclaw",
    "item": {
      "additionalProperty": [
        {"@type": "PropertyValue", "propertyID": "app:bucket", "value": "next"}
      ]
    }
  }'
```

Gueltige Buckets: `inbox`, `next`, `waiting`, `calendar`, `someday`, `reference`

Name aendern:

```bash
exec curl -s -X PATCH "$COPILOT_BACKEND_URL/items/<item_id>" \
  -H "Authorization: Bearer $(cat /runtime/token)" \
  -H "Content-Type: application/json" \
  -H "X-Agent: openclaw" \
  -d '{
    "source": "openclaw",
    "item": {
      "name": "Neuer Name"
    }
  }'
```

## Wichtig: Payload-Format fuer neue Items

Alle POST-Requests an `/items` erwarten dieses Format:

```json
{
  "source": "openclaw",
  "item": { ... JSON-LD ... }
}
```

Jedes Item braucht eine `@id` als URN. Erzeuge sie mit UUID:
- Aktionen: `urn:app:action:<uuid>`
- Referenzen: `urn:app:reference:<uuid>`
- Projekte: `urn:app:project:<uuid>`

Verwende `$(cat /proc/sys/kernel/random/uuid)` fuer UUIDs.

## Aktion erstellen

```bash
exec curl -s -X POST "$COPILOT_BACKEND_URL/items" \
  -H "Authorization: Bearer $(cat /runtime/token)" \
  -H "Content-Type: application/json" \
  -H "X-Agent: openclaw" \
  -d '{
    "source": "openclaw",
    "item": {
      "@id": "urn:app:action:'"$(cat /proc/sys/kernel/random/uuid)"'",
      "@type": "Action",
      "_schemaVersion": 2,
      "name": "Aktion-Name",
      "additionalProperty": [
        {"@type": "PropertyValue", "propertyID": "app:bucket", "value": "next"},
        {"@type": "PropertyValue", "propertyID": "app:rawCapture", "value": "Aktion-Name"}
      ]
    }
  }'
```

Gueltige Buckets: `inbox`, `next`, `waiting`, `calendar`, `someday`

## Referenz erstellen

```bash
exec curl -s -X POST "$COPILOT_BACKEND_URL/items" \
  -H "Authorization: Bearer $(cat /runtime/token)" \
  -H "Content-Type: application/json" \
  -H "X-Agent: openclaw" \
  -d '{
    "source": "openclaw",
    "item": {
      "@id": "urn:app:reference:'"$(cat /proc/sys/kernel/random/uuid)"'",
      "@type": "CreativeWork",
      "_schemaVersion": 2,
      "name": "Referenz-Name",
      "description": "Optionale Beschreibung",
      "additionalProperty": [
        {"@type": "PropertyValue", "propertyID": "app:bucket", "value": "reference"}
      ]
    }
  }'
```

## Projekt erstellen (Projekt + Aktionen)

Erstelle zuerst das Projekt, dann die Aktionen mit `app:projectRefs`:

```bash
# 1. Projekt erstellen
exec curl -s -X POST "$COPILOT_BACKEND_URL/items" \
  -H "Authorization: Bearer $(cat /runtime/token)" \
  -H "Content-Type: application/json" \
  -H "X-Agent: openclaw" \
  -d '{
    "source": "openclaw",
    "item": {
      "@id": "urn:app:project:'"$(cat /proc/sys/kernel/random/uuid)"'",
      "@type": "Project",
      "_schemaVersion": 2,
      "name": "Projekt-Name",
      "additionalProperty": [
        {"@type": "PropertyValue", "propertyID": "app:bucket", "value": "project"},
        {"@type": "PropertyValue", "propertyID": "app:desiredOutcome", "value": "Gewuenschtes Ergebnis"},
        {"@type": "PropertyValue", "propertyID": "app:projectStatus", "value": "active"}
      ]
    }
  }'

# 2. canonical_id aus der Antwort notieren, dann Aktionen erstellen:
exec curl -s -X POST "$COPILOT_BACKEND_URL/items" \
  -H "Authorization: Bearer $(cat /runtime/token)" \
  -H "Content-Type: application/json" \
  -H "X-Agent: openclaw" \
  -d '{
    "source": "openclaw",
    "item": {
      "@id": "urn:app:action:'"$(cat /proc/sys/kernel/random/uuid)"'",
      "@type": "Action",
      "_schemaVersion": 2,
      "name": "Schritt 1",
      "additionalProperty": [
        {"@type": "PropertyValue", "propertyID": "app:bucket", "value": "next"},
        {"@type": "PropertyValue", "propertyID": "app:rawCapture", "value": "Schritt 1"},
        {"@type": "PropertyValue", "propertyID": "app:projectRefs", "value": ["<canonical_id_from_project>"]}
      ]
    }
  }'
```

## Antwortformat

Erfolgreiche Erstellung liefert:

```json
{
  "item_id": "uuid",
  "canonical_id": "urn:app:...",
  "source": "openclaw",
  "item": { ... },
  "created_at": "...",
  "updated_at": "..."
}
```
