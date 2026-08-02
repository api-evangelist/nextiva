---
name: Transfer a Nextiva workitem or send an email response
description: Escalate an in-flight contact-center interaction — change ACD priority, transfer to another agent or to an inbox queue, or answer an email workitem.
api: openapi/nextiva-workitem-service-openapi.yml
operations:
  - generateTokenWithAuthorities
  - fetchWorkitemById
  - changeWorkitemPriority
  - transferWorkitemToAgent
  - transferWorkitemToInbox
  - sendEmailResponse
generated: '2026-07-31'
method: generated
source: openapi/nextiva-workitem-service-openapi.yml
---

# Transfer a Nextiva workitem or send an email response

Authenticate first with `generateTokenWithAuthorities` and send
`Authorization: Bearer <jwt>` on every request. Base URL `https://api.nextiva.com`.

Confirm the workitem exists and read its `state`, `channelType` and `priority` with
`fetchWorkitemById` (`GET /users/api/workitems/{id}`) before acting.

## Raise or lower ACD priority

`changeWorkitemPriority` — `POST /workflows/api/workitems/{workitemId}/acd`.
The body carries a `WorkItemPriority`. Use this when a contact needs to jump the queue
rather than move to a specific destination.

## Transfer to a named agent

`transferWorkitemToAgent` — `POST /users/api/calls/{workitemId}/usertransfer/{userId}`.
The `TransferToAgentPayload` body accepts `eventName`, `userId`, `workitemId`, plus
`recording` and `survey` flags that control whether recording continues and whether a
post-interaction survey is offered.

## Transfer to an inbox queue

`transferWorkitemToInbox` — `POST /users/api/calls/{workitemId}/queuetransfer/{inboxId}`.
The `TransferToInboxPayload` body adds `ringAll` alongside `inboxId`, `eventName`,
`recording` and `survey`. Prefer this over an agent transfer when any available agent in
the queue can take the contact.

## Answer an email workitem

`sendEmailResponse` — `POST /users/api/workitems/{workitemId}/sendemailresponse`.
The `SendEmailResponsePayload` body carries `from`, `to`, `toAddresses`, `ccAddresses`,
`bccAddresses`, `subject`, `bodyParts` and `createdAt`. Only use this on a workitem whose
`channelType` is email.

## Rules an agent must follow

- **These are one-way, non-idempotent writes.** A transfer that times out may or may not
  have executed. Do not retry — re-read the workitem with `fetchWorkitemById` and branch
  on its `state`.
- **Confirm the destination first.** The contract does not validate that `userId` or
  `inboxId` exists; a bad id surfaces only as the generic `default` error.
- **Errors carry `{code, message}` only.** No RFC 9457 problem details, no error-code
  registry — see `errors/nextiva-problem-types.yml`.
