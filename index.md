---
title: Quick Start
layout: default
---

# Quick Start

Welcome to the **Grinfi.io Public API** Quick Start guide! This guide will help you quickly get started with the **Grinfi.io API** and understand its key functionalities.

## What You'll Learn

- How to authenticate with the API.
- How to create your first contact.
- How to send and retrieve LinkedIn messages.

---

## Authentication

To interact with the API, you must authenticate using a valid **API Key** in every request.

### Get Your API Key

To get started, generate an API key:

1. [Log in](https://leadgen.grinfi.io/) to your Grinfi.io account.
2. Navigate to the [API Keys](https://leadgen.grinfi.io/settings/api-keys) page.
3. Copy your existing key or create a new one.
4. Use the API key in the `Authorization` header:

   ```http
   Authorization: Bearer {YOUR_TOKEN}
   ```

---

## Creating a New Contact

Follow these steps to create your first contact:

1. Use the `https://leadgen.grinfi.io/leads/api/leads` endpoint.
2. Add the required headers:

   ```http
   Authorization: Bearer {YOUR_TOKEN}
   ```

3. Make a `POST` request to the `/leads` endpoint.
4. Each contact must be in some list. So go to **Lists** page, click on the three dots icon of any list and select the **"Copy List ID"** option.

### Example Request:

```http
POST /leads/api/leads HTTP/1.1
Host: leadgen.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
Content-Type: application/json

{
  "list_uuid": "uuid-uuid-uuid-uuid-example",
  "leads": [
    {
      "linkedin_id": "john-doe-123456 or ACoAAAB6GuQBOXo75numqJfM9u08uHgjOSo4p9U",
      "first_name": "John",
      "last_name": "Doe",
      "company_name": "ExampleCorp",
      "ln_id": "ACoAAAB6GuQBOXo75numqJfM9u08uHgjOSo4p9U",
      "sn_id": "ACwAAAB6GuQBEz2TMHbG4sa7Nw5wSi7cJXMQkPI",
      "linkedin": "christina-sletner-b4481a2",
      "email": "john.doe@example.com",
      "about": "Some facts about John...",
      "domain": "somecoolcompany.com",
      "headline": "Sales & Partner Manager I ERP-/Dynamics 365 Finance/OnSite 365 at Advania Norge",
      "position": "Sales & Partner Manager",
      "raw_address": "Oslo, Oslo, Norway",
      "custom_fields": {
        "Gender": "Male",
        "Connection_Message": "Hi John, I loved your recent insights on automating LinkedIn voices. I’d appreciate connecting to learn more about your approach and share ideas!",
        "First_Message": "Thanks for connection John, I was curious ..."
      }
    }
  ]
}
```

### Example Response:

```json
[
  {
    "uuid": "uuid-uuid-uuid-uuid-example",
    "team_id": 1,
    "user_id": 1,
    "sender_profile_uuid": "uuid-uuid-uuid-uuid-example",
    "list_uuid": "uuid-uuid-uuid-uuid-example",
    "data_source_uuid": "uuid-uuid-uuid-uuid-example",
    "pipeline_stage_uuid": "uuid-uuid-uuid-uuid-example",
    "company_uuid": "uuid-uuid-uuid-uuid-example",
    "name": "Christina Sletner",
    "first_name": "Christina",
    "last_name": "Sletner",
    "company_name": "Advania Norge",
    "company_ln_id": "98",
    "position": "Sales & Partner Manager",
    "headline": "Sales & Partner Manager I ERP-/Dynamics 365 Finance/OnSite 365 at Advania Norge",
    "about": "Busy times with contributing to our customers' cloud journey...",
    "linkedin_status": "active",
    "email_status": "verified",
    "created_at": "2022-12-11T10:00:00Z",
    "updated_at": "2022-12-11T11:00:00Z"
  }
]
```

---

## Managing LinkedIn Messages

### List LinkedIn Messages for a Specific Contact

Retrieve a list of LinkedIn messages using the `https://leadgen.grinfi.io/flows/api/linkedin-messages` endpoint.

### Example Request:

```http
GET /flows/api/linkedin-messages?filter[lead_uuid]=uuid-uuid-uuid-uuid-example HTTP/1.1
Host: api.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
```

### Example Response:

```json
{
  "data": [
    {
      "uuid": "uuid-uuid-uuid-uuid-example",
      "team_id": 1,
      "sender_profile_uuid": "uuid-uuid-uuid-uuid-example",
      "linkedin_account_uuid": "uuid-uuid-uuid-uuid-example",
      "linkedin_conversation_uuid": "uuid-uuid-uuid-uuid-example",
      "lead_uuid": "uuid-uuid-uuid-uuid-example",
      "message_hash": null,
      "text": "Hello John, how are you doing?",
      "attachments": [],
      "type": "outbox",
      "status": "new",
      "sent_at": "2022-05-20T20:49:39.000000Z"
    }
  ],
  "limit": 10,
  "offset": 0,
  "total": 100,
  "has_more": true
}
```

---

## Sending a LinkedIn Message

Send a LinkedIn message using the `https://leadgen.grinfi.io/flows/api/linkedin-messages` endpoint.

### Example Request:

```http
POST /flows/api/linkedin-messages HTTP/1.1
Host: api.grinfi.io
Authorization: Bearer {YOUR_TOKEN}
Content-Type: application/json

{
  "sender_profile_uuid": "uuid-uuid-uuid-uuid-example",
  "lead_uuid": "uuid-uuid-uuid-uuid-example",
  "text": "Hello John, how are you?"
}
```

### Example Response:

```json
{
  "uuid": "uuid-uuid-uuid-uuid-example",
  "team_id": 1,
  "sender_profile_uuid": "uuid-uuid-uuid-uuid-example",
  "linkedin_account_uuid": "uuid-uuid-uuid-uuid-example",
  "linkedin_conversation_uuid": "uuid-uuid-uuid-uuid-example",
  "lead_uuid": "uuid-uuid-uuid-uuid-example",
  "text": "Hello John, how are you?",
  "status": "new",
  "sent_at": "2022-05-20T20:49:39.000000Z"
}
```

---

## Next Steps

Once you've completed this Quick Start guide:

- Explore the full [API Reference](https://grinfi-api.redocly.app/openapi) for more details.
- Experiment with additional features to create sender profiles and start-stop automations.

Happy integrating!