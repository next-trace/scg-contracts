Purpose: Canonical identity lifecycle and ACL delta events for Identity Service consumers.

Topic: scg.identity.lifecycle.v1
Partition key:
- User lifecycle: user_uuid (secondary key tenant_uuid where applicable)
- ACL deltas: tenant_uuid

Field semantics:
- event_id: UUID v7 unique per event
- occurred_at: event occurrence time (UTC)
- correlation_id/causation_id: trace linkage IDs (optional)
- source_service: producer service name
- actor_sub: OIDC subject of actor or "system"
- attributes: non-PII metadata map
- user_uuid, tenant_uuid, role_uuid: primary entity keys
- role_name: human-readable role name (not a key)

Envelope standard:
- Use meta (proto.scg.shared.v1.EventEnvelope) going forward.
- Inline envelope fields (event_id, occurred_at, correlation_id, causation_id, source_service, actor_sub, attributes) are deprecated on canonical events.
- The transitional meta_common (proto.scg.common.v1.EventMeta) field is also deprecated.
- Removal of deprecated fields will occur in a future major version; continue to populate meta for forward compatibility.

PII policy:
- Canonical identity events do not include PII. Prefer stable identifiers (user_uuid, tenant_uuid, role_uuid, sub).
- Operational/legacy events may contain limited PII or labels; they remain for internal usage and will be reviewed in a future major version.
