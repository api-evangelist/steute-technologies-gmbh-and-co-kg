---
name: Monitor a nexy wireless switch fleet
description: >-
  Authenticate to the on-premise Sensor Bridge API v2, inventory switches and
  access points, and read event history and system health.
api: openapi/steute-technologies-gmbh-and-co-kg-sensor-bridge-openapi-original.json
operations:
- AuthController_login
- AuthController_refresh
- SwitchController_listSwitches
- SwitchController_getSwitch
- AccessPointController_listAccessPoints
- AccessPointController_warnings
- HistoryController_list
- SystemStatusController_listSystemStatus
---

# Monitor a nexy wireless switch fleet

The Sensor Bridge is an on-premise gateway (default device address
192.168.3.32); all calls go to `http(s)://<device>/api/v2`.

1. **Login** — `AuthController_login` (`POST /api/v2/auth/login`) with the API
   username/password (factory default user `api` / `steute_api`; changed by the
   administrator in the web UI). The response `accessToken` is a JWT valid 30
   minutes; send it on every other call in the `JWTAuthorization` header.
   Refresh before expiry via `AuthController_refresh` (`GET /api/v2/auth/refresh`).
2. **Inventory switches** — `SwitchController_listSwitches`
   (`GET /api/v2/switches`) returns every registered switch with state,
   battery (`battState`, `battVoltage`), signal (`rssi`) and group assignment.
   Drill into one device with `SwitchController_getSwitch`
   (`GET /api/v2/switches/{deviceId}`); device IDs are 6-char hex strings.
3. **Check the radio infrastructure** — `AccessPointController_listAccessPoints`
   (`GET /api/v2/access-points`) lists access points and their status;
   `AccessPointController_warnings` (`GET /api/v2/access-points/warnings/{deviceId}`)
   returns warnings for one access point.
4. **Read event history** — `HistoryController_list`
   (`GET /api/v2/history/{deviceId}`) returns switch event logs for a device.
5. **System health** — `SystemStatusController_listSystemStatus`
   (`GET /api/v2/system-status`) reports software version, load, memory and uptime.

Rules: a 403 on any call means the JWT is missing/expired (re-login) or the
credentials are wrong; 404 means an unknown `deviceId`. There is no pagination —
lists return complete arrays. No rate limits are documented, but this is a
small embedded device: poll sparingly and prefer HTTP(S) push notifications
for state changes (see the configure-webhooks skill).
