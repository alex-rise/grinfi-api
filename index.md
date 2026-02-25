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
| **Integrations** | AI Templates |

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
- Use [AI Templates](https://api.grinfi.io/openapi#tag/AI-Templates) for smart message generation.

Happy integrating!
