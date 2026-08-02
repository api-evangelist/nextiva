---
name: Authenticate and control a live Nextiva call
description: Mint a Nextiva JWT from basic credentials, find the agent's active workitem, and drive the call — bridge, hold, mute, send DTMF, hang up.
api: openapi/nextiva-workitem-service-openapi.yml
operations:
  - generateTokenWithAuthorities
  - refreshToken
  - fetchWorkitems
  - fetchWorkitemById
  - activeWorkitemCall
  - holdWorkitemCall
  - muteWorkitemCall
  - unMuteWorkitemCall
  - sendDtmfTones
  - hangUpWorkitemCall
generated: '2026-07-31'
method: generated
source: openapi/nextiva-authentication-openapi.yml, openapi/nextiva-workitem-service-openapi.yml
---

# Authenticate and control a live Nextiva call

Base URL for every request: `https://api.nextiva.com`.

## 1. Mint a token

Call `generateTokenWithAuthorities` (`GET /provider/token-with-authorities`) with HTTP
basic credentials. It returns a JWT plus the user's authorities. Send that token on every
later request as `Authorization: Bearer <jwt>`.

When the token nears expiry call `refreshToken` (`GET /provider/api/token-refresh`) with the
current bearer token. A `401` from `refreshToken` means the token is already invalid or
expired — go back to `generateTokenWithAuthorities` with basic credentials.

## 2. Find the workitem

A **workitem** is the unit of work an agent handles; a live call is attached to one.

- `fetchWorkitems` (`GET /data/api/types/workitem`) lists them. Page with `start` and `rows`;
  narrow with `q` / `queryParams`. The response is the `PaginatedResponse` envelope
  (`count`, `total`, `objects`).
- `fetchWorkitemById` (`GET /users/api/workitems/{id}`) returns one, including `state`,
  `channelType`, `type`, `priority` and `agentUsername`. A `404` means the workitem does not
  exist — this is the only operation in the spec that declares one.

Take `workitemId` from the workitem you selected; every call-control operation below is keyed on it.

## 3. Drive the call

| Intent | Operation | Request |
|---|---|---|
| Answer / bridge | `activeWorkitemCall` | `POST /users/api/calls/{workitemId}/bridge` |
| Hold | `holdWorkitemCall` | `POST /users/api/calls/{workitemId}/hold` |
| Mute | `muteWorkitemCall` | `POST /users/api/calls/{workitemId}/mute` |
| Unmute | `unMuteWorkitemCall` | `DELETE /users/api/calls/{workitemId}/mute` |
| Send a DTMF digit | `sendDtmfTones` | `POST /users/api/calls/{workitemId}/dtmf/{digit}` |
| Hang up | `hangUpWorkitemCall` | `POST /users/api/calls/{workitemId}/hangup` |

Mute and unmute are the same path with different methods — `DELETE` unmutes.

## 4. Rules an agent must follow

- **Do not retry blindly.** Nextiva documents no idempotency key and no safe-retry
  guarantee. Re-sending `hangUpWorkitemCall`, `sendDtmfTones` or `activeWorkitemCall`
  after a timeout may act twice. On an ambiguous failure, re-read the workitem with
  `fetchWorkitemById` and decide from its `state`.
- **Errors are thin.** Every call-control operation declares only a `default` error
  response with a `{code, message}` JSON body. You cannot distinguish auth failure from
  a bad workitem id from the contract — read `code` and `message`.
- **No rate limits are published.** Back off on your own; there is no `429` in the
  contract and no rate-limit headers are documented.
- **State arrives out-of-band.** Call state changes are pushed on the SDK WebSocket
  stream (`WorkitemStateChangeNotification`, `PhoneStateNotification`), not returned by
  these REST operations. See `asyncapi/nextiva-events.yml`.
