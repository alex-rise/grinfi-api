# Known issues & workarounds

Practical notes for calling the Grinfi LeadGen API, especially when building
automations (flows) programmatically. Each item has a **workaround** you can apply
today — none of these block automation building.

_Last reviewed: 2026-07-22_

---

## 1. `validate-node` requires `before` and `after` (even when empty)

**Endpoint:** `POST /flows/api/flow-versions/validate-node`

Sending a node without the `before` and `after` edge arrays returns a **500** instead
of a validation error. This is independent of the node `type` — it happens for every
type when those fields are missing.

**Workaround:** always include `before` and `after`, using `[]` when the node has no
edges yet. Send the full node object (the same shape you would store in a flow version):

```json
{
  "id": 1,
  "before": [],
  "after": [],
  "type": "linkedin_send_connection_request",
  "automation": "auto",
  "payload": { "template": "Hi {{first_name}}!", "fallback_send": true, "template_uuid": null },
  "delay_in_seconds": 0
}
```

A valid node returns `204 No Content`.

> Note: the node config goes in `payload` with `automation` set to the string `"auto"`.
> The `flow-node` schema in the OpenAPI spec lists the `payload` shape for all node types.

---

## 2. Flow-lead filters reject unknown field names with a 500

**Endpoint:** `POST /flows/api/flows-leads/list`

Passing a filter field that doesn't exist (e.g. `flow_uuids`) returns a **500** instead
of a validation error.

**Workaround:** use the documented filter field names. The flow filter is `flow_uuid`
(singular, array-valued):

```json
{ "filter": { "flow_uuid": ["3a4ac30b-1321-449c-9b37-e9fe3174e00e"] }, "limit": 20 }
```

Supported `filter` keys: `flow_uuid`, `lead_uuid`, `status` (all arrays).

---

## 3. Flows built via API may not open in the web editor

Flows created through `POST /flows/api/flows/{flowUuid}/flow-versions` run correctly, but
the visual editor (Automations page) enforces extra layout invariants beyond what
`validate-node` checks. If a programmatically built flow fails to open in the editor,
check that the node tree satisfies these:

- **`first_common_node_id` must be set** whenever the flow branches. In editor-created
  flows this field is populated in virtually every version (211/213 sampled) — it points
  to the node where the branches reconverge. Sending `null` (or omitting it) on a
  branching tree is the most common reason a valid, runnable flow won't render. Use `null`
  only for a purely linear tree. Always round-trip this field when you read a version back
  and re-save it.
- **Symmetric edges** — for every `after` edge `{node_id: B}` on node A, node B has a
  matching `before` edge `{node_id: A}` with the same `branch_id`.
- **Consistent `branch_id`s** — linear steps use `1`; `rule_ab_test` uses `1`/`2`;
  `rule_filter` uses `1` (match) and `0` (else); triggers use `1`/`2`.
- **Every branch ends in an `end` node.** A flow may have several `end` nodes (one per
  branch tail), and branches may reconverge into a shared `end` — both are valid. What
  matters is that no branch is left dangling.

**No node coordinates are needed** — nodes carry no x/y or layout fields; the editor
derives the layout from the `before`/`after` graph plus `first_common_node_id`. These are
editor-only invariants; the API runs the flow without them. If a tree runs but won't open
visually, the cause is almost always a missing/`null` `first_common_node_id` on a
branching flow, or an asymmetric `before`/`after` edge.

---

## 4. `createFlowVersion` rejects an empty `contact_sources` array

**Endpoint:** `POST /flows/api/flows/{flowUuid}/flow-versions`

Sending `contact_sources: []` returns a **422** — `"The contact sources field is
required."` (verified on prod 2026-07-22). There is no way to save a version with a
truly empty audience list.

**Workaround:** always send at least one source. For a version with no audience yet,
send the same default source the backend itself auto-creates with every new flow shell
(see below) — `sender_profiles: []` means no senders are attached, so nothing runs:

```json
{
  "id": 1,
  "name": null,
  "rotation_strategy": "fair",
  "sender_profiles": [],
  "after_id": 3,
  "mass_actions_filter": null,
  "filter_tree_format": null
}
```

`after_id` points at the tree's **entry node** (the first step leads execute — the node
with no incoming `before` edges).

**Related quirk (useful, not a bug):** `POST /flows/api/flows` auto-creates an initial
flow version for the new shell — a single `end` node plus exactly this default contact
source (`after_id` pointing at the `end` node). A fresh draft therefore already has a
`flow_version_uuid` before you save anything; your first `createFlowVersion` adds a
second version that replaces it.

---

## 5. Creating an AI Template requires `is_public`

**Endpoint:** `POST /flows/api/ai-templates`

Omitting `is_public` returns a **500** (a backend type error — `is_public` reaches the
domain constructor as `null`) instead of a validation error.

**Workaround:** always send `is_public` as an integer — `1` (team-visible) or `0`
(private). Minimal working create body: `{ "name": "...", "type": "message", "is_public": 1 }`.
Valid `type` values: `message`, `connection_note`, `email`, `post_comment`.

---

## 6. Deleted AI Templates still show up in the API (read path ignores soft-delete)

**Endpoint:** `DELETE /flows/api/ai-templates/{uuid}` (+ `List` / `Get`)

`DELETE` returns `204` and the template **does** get removed — it disappears from the web
app. But the API read path does not filter it out: after deletion the template still
resolves on `GET /flows/api/ai-templates/{uuid}` and still appears in `List AI Templates`
(verified 2026-07-22). Deletion looks like a soft-delete that only the UI honors.

**Workaround:** treat a template as deleted as soon as `DELETE` returns `204`. Do **not**
use `List`/`GET` to confirm removal — they will still show the deleted record. If you need
a reliable "is this alive?" check, track deletions on your side until the backend filters
soft-deleted rows from the read endpoints. `POST` (create) and `PUT` (update) work normally.

---

## 7. Flat query filters are silently ignored — use `filter[...]`

**Endpoints:** `GET /flows/api/tasks`, `GET /flows/api/linkedin-conversations`, and other
list endpoints backed by a `*Filter` class.

Passing a filter as a flat query param (`?type=`, `?status=`, `?flow_uuid=`) is **silently
ignored** — you get unfiltered results with a `200`, which is easy to mistake for real data.
Measured on `/flows/api/tasks`: `?type=linkedin_send_connection_request` returned the same
`total` (1,245,527) as no filter at all; `filter[type]=…` returned 254,654.

**Workaround:** always use the bracket form, URL-encoded:

```
GET /flows/api/tasks?limit=1
  &filter%5Bflow_uuid%5D=<uuid>
  &filter%5Btype%5D=linkedin_send_connection_request
  &filter%5Bstatus%5D=closed
  &filter%5Bexecuted_at%5D%5B%3E%3D%5D=2026-07-22
  &filter%5Bexecuted_at%5D%5B%3C%3D%5D=2026-07-23
```

Notes:
- Date ranges use comparison operators **inside** the field: `filter[executed_at][>=]`,
  `filter[executed_at][<=]`. Range-style field names (`executed_at_from`, `date_start`, …)
  are not valid and return a 500 (see §2 — unknown filter keys throw instead of 422).
- Read `total` with `limit=1` to count without paging.
- **Narrow your query.** Unfiltered or broad reporting queries over large task volumes can
  return `504 Gateway Timeout`; adding `filter[flow_uuid]` keeps them ~1s.
