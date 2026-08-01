---
name: Configure push notifications (webhooks) from the Sensor Bridge
description: >-
  Set up HTTP(S) POST notifications so the Sensor Bridge pushes switchEvent and
  missingSwitchWakeupWarning JSON messages to your application instead of polling.
api: openapi/steute-technologies-gmbh-and-co-kg-sensor-bridge-openapi-original.json
operations:
- AuthController_login
- NotificationConfigController_listNotificationConfig
- NotificationConfigController_create
- NotificationConfigController_update
- NotificationConfigController_del
---

# Configure push notifications (webhooks) from the Sensor Bridge

1. **Login** — `AuthController_login` (`POST /api/v2/auth/login`); carry the JWT
   in the `JWTAuthorization` header (30-minute TTL).
2. **List existing endpoints** — `NotificationConfigController_listNotificationConfig`
   (`GET /api/v2/notification-configs`).
3. **Create an endpoint** — `NotificationConfigController_create`
   (`POST /api/v2/notification-configs`) with `name` (max 64 chars),
   `urlString` (target URL **without** the `http(s)://` prefix, custom port via
   suffix, e.g. `192.168.3.10:3000/erp/v2/incoming`), optional `description`,
   and `isEnabled`. HTTP Basic auth credentials for your receiver and
   group restrictions are supported. Expect `201`; `422` signals a validation
   failure, `400` a bad request.
4. **Receive messages** — the Sensor Bridge POSTs JSON
   (`Content-Type: application/json;charset=utf-8`) with a `messages[]` array.
   `messageType: switchEvent` carries `deviceId`, `state` (0-255 bitwise),
   `flags` (0 = actuation, 64 = broadcast, 128 = wakeup), `battState`,
   `battVoltage`, `rssi`, `counter`, `groupId` and `timestamp` (ms epoch).
   `messageType: missingSwitchWakeupWarning` fires when configured wake-ups
   fail to arrive more than twice, and repeats at wakeup-time * 2 intervals
   until the device transmits again.
5. **Maintain** — update with `NotificationConfigController_update`
   (`PUT /api/v2/notification-configs/{id}`), remove with
   `NotificationConfigController_del` (`DELETE /api/v2/notification-configs/{id}`).

Rules: respond quickly to the POST (the bridge enforces a configurable
timeout); use `flags` to separate actuations from wake-ups; schema reference in
asyncapi/steute-technologies-gmbh-and-co-kg-webhooks-asyncapi.yml.
