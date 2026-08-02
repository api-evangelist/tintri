---
name: Snapshot a VM on Tintri VMstore
description: Authenticate to a Tintri VMstore appliance and take a snapshot of a virtual machine.
api: Tintri VMstore REST API v310
operations:
  - POST /api/v310/session/login
  - GET /api/v310/vm
  - POST /api/v310/snapshot
  - GET /api/v310/session/logout
---

# Snapshot a VM on Tintri VMstore

Operating instructions for an agent to snapshot a VM using the Tintri VMstore
REST API (v310). All calls are made against a specific appliance host
`https://{vmstore}/api/v310`.

## Auth
1. `POST /api/v310/session/login` with a JSON body containing `username`,
   `password`, and `typeId` set to
   `com.tintri.api.rest.vcommon.dto.rbac.RestApiCredentials`.
2. Capture the returned `JSESSIONID` cookie and send it on every subsequent
   request. There is no OAuth token — the session cookie is the credential.

## Steps
1. **Find the VM.** `GET /api/v310/vm` with a `VirtualMachineFilterSpec`
   (filter by name or uuid) to resolve the target VM's `uuid`.
2. **Create the snapshot.** `POST /api/v310/snapshot` with a request that
   references the VM `uuid`, a snapshot name, and the snapshot type
   (e.g. crash-consistent). The API is **not idempotent** — do not blindly
   retry a create; on a timeout, re-list snapshots to check whether one was made.
3. **Confirm.** `GET /api/v310/snapshot` filtered by the VM `uuid` to verify the
   snapshot exists.
4. **Log out.** `GET /api/v310/session/logout` to close the session.

## Rules
- Errors return a proprietary JSON `Error` object (code + message), not RFC 9457
  problem+json; branch on the HTTP status and the `code` field.
- See `conventions/tintri-conventions.yml` for pagination (Page objects) and
  filter-spec usage, and `authentication/tintri-authentication.yml` for the
  session model.
