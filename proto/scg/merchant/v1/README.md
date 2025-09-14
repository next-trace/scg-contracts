Purpose: Canonical merchant lifecycle events for identity/merchant integration.

Topic: scg.merchant.lifecycle.v1
Partition key: merchant_uuid

Field semantics:
- event_id: UUID v7 unique per event
- occurred_at: event occurrence time (UTC)
- correlation_id/causation_id: trace linkage IDs (optional)
- source_service: producer service name
- actor_sub: OIDC subject of actor or "system"
- attributes: non-PII metadata map
- merchant_uuid: primary entity key
- merchant_key: optional stable business key
- name/status/reason: current merchant details and rationale for changes

Envelope standard:
- Use meta (proto.scg.shared.v1.EventEnvelope) going forward.
- Inline envelope fields (event_id, occurred_at, correlation_id, causation_id, source_service, actor_sub, attributes) are deprecated on canonical events.
- The transitional meta_common (proto.scg.common.v1.EventMeta) field is also deprecated.
- Removal of deprecated fields will occur in a future major version; continue to populate meta for forward compatibility.

PII policy:
- Canonical merchant events avoid PII; prefer stable IDs (merchant_uuid, merchant_key where applicable).
- Free-text reason/status/name fields are labels; avoid including PII in attributes or labels.
