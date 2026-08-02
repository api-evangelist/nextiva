---
name: Send an SMS from Nextiva
description: Mint a Nextiva token and send an outbound SMS message on a campaign, and know where inbound replies arrive.
api: openapi/nextiva-sms-messaging-openapi.yml
operations:
  - generateTokenWithAuthorities
  - sendSmsMessage
generated: '2026-07-31'
method: generated
source: openapi/nextiva-sms-messaging-openapi.yml
---

# Send an SMS from Nextiva

## 1. Authenticate

`generateTokenWithAuthorities` with HTTP basic credentials → JWT. Send it as
`Authorization: Bearer <jwt>`.

## 2. Send

`sendSmsMessage` — `POST https://api.nextiva.com/users/api/sms` with a
`SmsSendMessagePayload` body. A **campaign ID is required**: a `400` response with
"Missing required Campaign ID or invalid parameters" is what you get without one. A `401`
means the bearer token is missing or invalid — re-mint it.

## 3. Inbound replies

There is no inbound-SMS webhook to register. Per the SMS Messaging API's own description,
inbound SMS arrives as a **workitem** on the SDK event stream, with webhook configuration
handled internally by the Nextiva platform. To read a reply, either subscribe to the SDK
events socket or poll `fetchWorkitems` (`GET /data/api/types/workitem`) in the Workitem
Service API and filter on `channelType`.

## Rules an agent must follow

- **Sending is not idempotent.** No `Idempotency-Key` is supported. A retry after a
  timeout can send the message twice. If the response is ambiguous, check for the message
  on the conversation surface (`fetchConversations` with `messages=true`) before resending.
- **No rate limit is published** — no `429` in the contract and no documented per-second
  ceiling. Throttle conservatively.
- SMS to US numbers is subject to campaign registration (10DLC) and Nextiva's SMS Terms
  (`https://www.nextiva.com/legal.html?doc=19`); the `campaignId` requirement is how that
  registration is enforced at the API.
