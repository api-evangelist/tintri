---
name: Monitor Tintri VMstore alerts and VM performance
description: Authenticate to a Tintri VMstore appliance and read active alerts and per-VM performance stats.
api: Tintri VMstore REST API v310
operations:
  - POST /api/v310/session/login
  - GET /api/v310/alert
  - GET /api/v310/vm
  - GET /api/v310/session/logout
---

# Monitor Tintri VMstore alerts and VM performance

Operating instructions for an agent to poll appliance health using the Tintri
VMstore REST API (v310), against `https://{vmstore}/api/v310`.

## Auth
1. `POST /api/v310/session/login` with `username`, `password`, and `typeId`
   `com.tintri.api.rest.vcommon.dto.rbac.RestApiCredentials`.
2. Reuse the returned `JSESSIONID` cookie on every request.

## Steps
1. **List active alerts.** `GET /api/v310/alert` with an `AlertFilterSpec`
   (filter by severity/state) to retrieve current appliance alerts. Results
   page as a `Page` object — follow `next` until exhausted.
2. **Inspect affected VMs.** For alerts referencing a VM, `GET /api/v310/vm`
   with a `VirtualMachineFilterSpec` to read per-VM stats (IOPS, latency,
   throughput) and QoS config.
3. **Poll on a cadence.** Alerts are read via polling; there is **no webhook /
   event push** surface (see `conventions/tintri-conventions.yml`).
4. **Log out.** `GET /api/v310/session/logout`.

## Rules
- This is a read-only monitoring flow; do not mutate state.
- Handle the proprietary JSON `Error` object on non-2xx responses.
- Respect that this is an on-appliance API — target the correct VMstore/TGC host.
