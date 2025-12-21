# scg-contracts — SupplyChainGuard Domain Contracts

SCG Contracts is the single source of truth for SupplyChainGuard’s domain contracts. It contains canonical, versioned Protocol Buffers for value objects, models, enums, and domain events across all bounded contexts. Services import the generated code to exchange messages over gRPC, REST, and event buses while preserving a ubiquitous language.

---

## Overview: Domains & Bounded Contexts

All contracts live under proto/scg and are organized by bounded context. Each context typically contains enums.proto, models.proto, and events.proto.

- shared/v1 — Common types, enums, error, event envelope
- partner/v1 — Partners (suppliers, customers, carriers)
- unit/v1 — Units/SKUs/batches
- consignment/v1 — Consignments, shipments, orders
- compliance/v1 — Regulations, certifications, attestations
- custody/v1 — Chain of custody, ownership transitions
- digital_twin/v1 — Sensor/IoT digital twins
- event_log/v1 — Event audit and history
- identity/v1 — Identities, roles, permissions, sessions
- inventory/v1 — Stock, locations, movements
- logistics/v1 — Routes, carriers, drivers, transport
- onboarding/v1 — Onboarding, verification
- provenance/v1 — Origin, trace history
- risk/v1 — Risk models and events
- security/v1 — Security incidents, policies
- user/v1 — User models and events
- warehouse/v1 — Warehouses, storage areas
- billing/v1 — Billing models and events

Note: The current version for these packages is v1. Breaking changes require a new vN directory. See Versioning for details.

---

## Build / Generate

This repo is Buf-native. Use Buf to lint, check breaking changes, and generate code. The helper script ./scg also wraps the common tasks.

Prerequisites
- Go 1.25+
- Tools: Buf CLI and Go protobuf plugins
  - Fast path: ./scg install-tools (installs buf, golangci-lint, govulncheck, gosec, goimports, protoc-gen-go, protoc-gen-go-grpc)
  - Manual installs (if you prefer):
    - go install github.com/bufbuild/buf/cmd/buf@latest
    - go install google.golang.org/protobuf/cmd/protoc-gen-go@latest
    - go install google.golang.org/grpc/cmd/protoc-gen-go-grpc@latest

Preferred: Buf
- Generate Go code into gen/go
  - buf generate
  - or: ./scg proto-gen
- Lint contracts
  - buf lint
- Breaking-change check (compares against main)
  - buf breaking --against '.git#branch=main'

Alternative: protoc using Buf export
If you must run protoc directly, export the module via Buf and then invoke protoc against the exported sources.

- Export the module
  - buf export . -o build/proto
- Generate Go code
  - protoc \
    -I build/proto \
    --go_out=gen/go --go_opt=paths=source_relative \
    --go-grpc_out=gen/go --go-grpc_opt=paths=source_relative \
    $(find build/proto/proto -name "*.proto" -print)

The generated Go packages appear under gen/go/proto/scg/... matching option go_package in the .proto files.

---

## Canonical import paths and aliasing

- go_package format (canonical): github.com/next-trace/scg-contracts/gen/go/proto/<domain>/v1;{alias}
  - Examples: identityv1, tenantv1, sharedv1, userv1, partnerv1, inventoryv1, etc.
- Package policy: we keep package names as proto.scg.<domain>.v1; lint is configured to allow DIRECTORY_SAME_PACKAGE to avoid moving files.
- Generated Go imports follow the alias after the semicolon; consumers import the aliased package path under gen/go.

Dependency locking for deterministic builds
- This repository checks in buf.lock to pin transitive dependencies (e.g., googleapis). Run buf dep update only when you intentionally want to bump dependency versions.
- Generation steps (buf generate) are pinned to specific plugin versions via buf.gen.yaml.

## Usage Example (Go)

Quickstart (so your IDE can resolve imports)

```sh
# in an empty folder for the example program
go mod init example.com/scg-consumer
# pull the contracts module
go get github.com/next-trace/scg-contracts@latest
```

Import and use generated contracts

```go
package main

import (
    "fmt"

    identityv1 "github.com/next-trace/scg-contracts/gen/go/proto/scg/identity/v1"
)

func main() {
    u := &identityv1.Identity{
        Uuid:     "3b1a1b14-5b7e-4d2a-8a3f-4a0b1cf0f001",
        Username: "alice",
        Email:    "alice@example.com",
        // optional fields can be left zero-valued
    }
    fmt.Printf("Identity: %s (%s)\n", u.GetUsername(), u.GetUuid())
}
```

The import paths reflect this repo’s module (github.com/next-trace/scg-contracts) and the generated code layout (gen/go/proto/scg/...).

---

## Versioning

This project follows [Semantic Versioning](https://semver.org/) (`MAJOR.MINOR.PATCH`):

- **MAJOR**: Breaking changes to the public API
- **MINOR**: Backward-compatible new features
- **PATCH**: Backward-compatible bug fixes

Consumers should always pin to a specific tag (e.g. `v1.2.3`) to avoid accidental breaking changes.

---

## Testing & Quality

Continuous Integration

- GitHub Actions: scg-contracts-ci
  - [![CI](https://github.com/next-trace/scg-contracts/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/next-trace/scg-contracts/actions/workflows/ci.yml)

Buf Checks

- Lint: buf lint
- Breaking change detection: buf breaking --against '.git#branch=main'
- Registry: [buf.build/next-trace/scg-contracts](https://buf.build/next-trace/scg-contracts)

Local helper
- ./scg install-tools — installs buf, protoc plugins, and quality tools
- ./scg proto-gen — runs: buf lint, buf breaking (on main), and buf generate
- ./scg ci — runs: build, test, lint, security, then proto-gen
- ./scg lint — runs golangci-lint
- ./scg lint-fix — runs golangci-lint --fix
- ./scg security — runs govulncheck and gosec
- ./scg format [file.go] — gofmt + goimports for all files or a single file
- ./scg build — go build ./...
- ./scg test — go test with race and coverage
- ./scg help — prints available commands

---

## Security

If you believe you have found a security vulnerability in these contracts:
- Please do not open a public issue with exploit details.
- Instead, report privately via GitHub Security Advisories or contact the maintainers by opening a minimal, non-sensitive issue requesting a security contact channel.

We aim to triage and respond promptly.

---

## Changelog

Release notes are available on the GitHub Releases page:
- https://github.com/next-trace/scg-contracts/releases

Significant changes are also visible in the git history. If you need a traditional CHANGELOG.md, feel free to open an issue.

---
## License & Contributing
- License: MIT
- Contributions: PRs and issues welcome; keep changes minimal and idiomatic.