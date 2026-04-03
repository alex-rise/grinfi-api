---
title: Quick Start
layout: default
---

# Quick Start

Welcome to the **Grinfi.io Public API**! This guide will help you authenticate, make your first API calls, and understand the key areas of the platform.

## What You'll Learn

- How to authenticate with the API.
- How to create and manage contacts.
- How to send LinkedIn messages and emails.
- How to work with automations, tasks, and webhooks.

---

## Authentication

Every API request requires a valid **API Key** in the `Authorization` header.

### Get Your API Key

1. [Log in](https://leadgen.grinfi.io/) to your Grinfi.io account.
2. Navigate to the [API Keys](https://leadgen.grinfi.io/settings/api-keys) page.
3. Copy your existing key or create a new one.

### Required Header

Include in every request:

```http
Authorization: Bearer {YOUR_TOKEN}
```

---

## API Overview

The API is organized into four main areas:

| Area | What it covers |
|------|---------------|
| **CRM** | Contacts, Companies, Lists, Tags, Pipeline Stages, Custom Fields, Notes |
| **Outreach** | Automations (flows), Tasks, Sender Profiles |
| **Messaging** | LinkedIn Messages, Emails, Mailboxes |
| **Integrations** | Webhooks |

---

## Creating a New Contact

Each contact must belong to a list. First, copy a list UUID from the **Lists** page (click the three-dot menu on any list and select **Copy List ID**).

### Example Request

```http
POST /leads/api/leads HTTP/1.1
Host: leadgen.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
Content-Type: application/json

{
  "list_uuid": "uuid-uuid-uuid-uuid-example",
  "leads": [
    {
      "linkedin_id": "john-doe-123456",
      "first_name": "John",
      "last_name": "Doe",
      "company_name": "ExampleCorp",
      "email": "john.doe@example.com",
      "position": "Sales Manager",
      "raw_address": "Oslo, Norway"
    }
  ]
}
```

### Example Response

```json
[
  {
    "uuid": "uuid-uuid-uuid-uuid-example",
    "first_name": "John",
    "last_name": "Doe",
    "company_name": "ExampleCorp",
    "position": "Sales Manager",
    "linkedin_status": "active",
    "email_status": "verified",
    "created_at": "2024-01-15T10:00:00Z"
  }
]
```

---

## Searching Contacts

Use the powerful filter system to find contacts by any field.

```http
POST /leads/api/leads/search HTTP/1.1
Host: leadgen.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
Content-Type: application/json

{
  "filter": {
    "company_name": "ExampleCorp"
  },
  "limit": 20,
  "offset": 0
}
```

Filter supports scalar values (exact match), arrays (IN), objects with operators (`>=`, `<=`, `>`, `<`, `=`, `!=`), and special values `is_null` / `is_not_null`.

---

## Sending a LinkedIn Message

Send a LinkedIn message using a sender profile.

```http
POST /flows/api/linkedin-messages HTTP/1.1
Host: leadgen.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
Content-Type: application/json

{
  "sender_profile_uuid": "uuid-uuid-uuid-uuid-example",
  "lead_uuid": "uuid-uuid-uuid-uuid-example",
  "text": "Hello John, how are you?"
}
```

---

## Managing Automations

Start and stop outreach automations programmatically.

```http
PUT /flows/api/flows/{flowUuid}/start HTTP/1.1
Host: leadgen.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
```

---

## Pagination

All list endpoints support pagination with `limit` and `offset` parameters. Responses include `total` and `has_more` fields.

```json
{
  "data": [...],
  "limit": 20,
  "offset": 0,
  "total": 150,
  "has_more": true
}
```

---

## Next Steps

- Explore the full [API Reference](https://api.grinfi.io/openapi) for all endpoints.
- Manage your [Automations](https://api.grinfi.io/openapi#tag/Automations) programmatically.

---

## Checking LinkedIn Connection Status

To check whether a contact has accepted a LinkedIn connection request, query the **Activities** endpoint with a JSON filter:

```http
GET /leads/api/activities?filter={"lead_uuid":"LEAD_UUID","type":"linkedin_connection_request_accepted"}&limit=1 HTTP/1.1
Host: leadgen.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
```

**Connected** — `total` is `1` (or more). The `payload` contains `sender_profile_uuid` (which sender profile is connected) and `connected_at` (timestamp).

```json
{
  "data": [
    {
      "type": "linkedin_connection_request_accepted",
      "lead_uuid": "a1b2c3d4-...",
      "payload": {
        "sender_profile_uuid": "e5f6a7b8-...",
        "connected_at": "2026-03-04T09:25:42.000000Z"
      },
      "created_at": "2026-03-04T09:25:42.000000Z"
    }
  ],
  "total": 1
}
```

**Not connected** — `total` is `0`. Either no connection request was sent, or it hasn't been accepted yet.

> **Note:** The `filter` parameter accepts a JSON string, not separate query parameters.

---

## Working with Custom Fields

Custom fields (e.g. `msg1`, `msg2`, `msg3`) are **not** standard contact fields.
The `PUT /leads/api/leads/{uuid}` endpoint silently ignores any field it does not
recognize, so custom values sent there will be lost without an error.

### Option A — Set each custom field separately

Use `POST /leads/api/custom-field-values` for each custom field.
You need the `custom_field_uuid` (get it from `GET /leads/api/custom-fields`).

```http
POST /leads/api/custom-field-values HTTP/1.1
Host: leadgen.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
Content-Type: application/json

{
  "custom_field_uuid": "aaaaaaaa-bbbb-cccc-dddd-eeeeeeeeeeee",
  "object_type": "lead",
  "object_uuid": "LEAD_UUID",
  "value": "Hello world"
}
```

Repeat for each custom field you need to set.

### Option B — Upsert with custom fields in one call

`POST /leads/api/leads/upsert` supports a `custom_fields` object, so you can
create or update a contact together with its custom fields in a single request:

```http
POST /leads/api/leads/upsert HTTP/1.1
Host: leadgen.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
Content-Type: application/json

{
  "lead": {
    "linkedin_id": "john-doe-123456",
    "first_name": "John",
    "last_name": "Doe"
  },
  "list_uuid": "LIST_UUID",
  "custom_fields": {
    "msg1": "First message",
    "msg2": "Second message"
  },
  "update_if_exists": true
}
```

> **Important:** All requests must use `Content-Type: application/json`.
> Sending data as `application/x-www-form-urlencoded` will cause validation errors.

---

Happy integrating!
