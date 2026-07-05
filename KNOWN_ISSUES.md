# Known issues & workarounds

Practical notes for calling the Grinfi LeadGen API, especially when building
automations (flows) programmatically. Each item has a **workaround** you can apply
today — none of these block automation building.

_Last reviewed: 2026-07-05_

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

- **Exactly one terminal `end` node**, reachable from every branch.
- **Symmetric edges** — for every `after` edge `{node_id: B}` on node A, node B has a
  matching `before` edge `{node_id: A}` with the same `branch_id`.
- **Consistent `branch_id`s** — linear steps use `1`; `rule_ab_test` uses `1`/`2`;
  `rule_filter` uses `1` (match) and `0` (else); triggers use `1`/`2`.
- **`first_common_node_id`** set on the version to the node where branches reconverge
  (or `null` for a purely linear tree). Round-trip this field when you read a version
  back and re-save it.

The API itself does not require the editor's layout metadata to run the flow — these are
editor-only invariants. If your tree runs but won't open visually, the mismatch is
almost always an asymmetric `before`/`after` edge or a missing `end` node.
