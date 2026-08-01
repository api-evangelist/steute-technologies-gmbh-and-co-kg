---
name: Manage nexy switch configuration groups
description: >-
  Create and tune configuration groups (wake-up intervals, cycle/debounce
  times, sensor thresholds) and assign switches to them on the Sensor Bridge.
api: openapi/steute-technologies-gmbh-and-co-kg-sensor-bridge-openapi-original.json
operations:
- AuthController_login
- SwitchGroupController_listSwitchGroups
- SwitchGroupController_create
- SwitchGroupController_update
- SwitchGroupController_deleteGroup
- SwitchController_update
---

# Manage nexy switch configuration groups

1. **Login** — `AuthController_login` (`POST /api/v2/auth/login`); JWT in the
   `JWTAuthorization` header.
2. **List groups** — `SwitchGroupController_listSwitchGroups`
   (`GET /api/v2/switch-groups`). Groups carry `wakeupHours`/`wakeupMin`
   (wake-up monitoring interval; default 15 minutes for new groups since
   Sensor Bridge 2.6.0), cycle/debounce factors (125 ms units) and
   sensor-specific thresholds (`sdsThresholdDist`, `ldsThresholdDist`, ...).
3. **Create a group** — `SwitchGroupController_create`
   (`POST /api/v2/switch-groups`); expect `201`, `422` on validation failure.
4. **Tune a group** — `SwitchGroupController_update`
   (`PUT /api/v2/switch-groups/{id}`); `404` for an unknown id.
5. **Assign a switch** — `SwitchController_update`
   (`PUT /api/v2/switches/{deviceId}`) setting `groupId` (plus per-device
   `description`, `parameterOne`, `parameterTwo` used in notification payloads).
6. **Remove a group** — `SwitchGroupController_deleteGroup`
   (`DELETE /api/v2/switch-groups/{id}`).

Rules: PUT updates are idempotent by id, but there is no idempotency-key
contract for POST creates — check for an existing group by name via the list
call before creating. Wake-up interval changes drive the
missingSwitchWakeupWarning monitoring described in the configure-webhooks skill.
