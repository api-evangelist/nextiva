---
name: Read Nextiva conversation history, transcripts and recordings
description: Pull the conversation graph for a customer, campaign or ticket and retrieve the transcriptions and call recordings attached to a workitem.
api: openapi/nextiva-conversation-openapi.yml
operations:
  - generateTokenWithAuthorities
  - fetchConversations
  - fetchConversationById
  - fetchConversationWorkItemsByStatus
  - fetchConversationHistoryByParticipantId
  - fetchConversationByCampaignId
  - fetchTicketConversations
  - fetchRecentConversationWorkitemEngagements
  - fetchUnreadCounts
  - fetchConversationActivitySummary
  - fetchTranscriptionsAndRecordings
generated: '2026-07-31'
method: generated
source: openapi/nextiva-conversation-openapi.yml
---

# Read Nextiva conversation history, transcripts and recordings

Authenticate with `generateTokenWithAuthorities`, then send `Authorization: Bearer <jwt>`.
Base URL `https://api.nextiva.com`.

## Pick the right entry point

| You have | Operation | Path |
|---|---|---|
| Nothing — browse | `fetchConversations` | `GET /data/api/types/conversation` |
| A `conversationId` | `fetchConversationById` | `GET /analytics/api/conversations/{conversationId}/workitems` |
| A conversation + status filter | `fetchConversationWorkItemsByStatus` | `GET /analytics/api/conversations/{conversationId}/workitems/{workItemStatus}` |
| A participant | `fetchConversationHistoryByParticipantId` | `GET /analytics/api/conversations/workitems` (`participantIds`) |
| A `campaignId` | `fetchConversationByCampaignId` | `GET /analytics/api/campaigns/{campaignId}/recentconversationworkitems` |
| A `ticketId` | `fetchTicketConversations` | `GET /analytics/api/tickets/{ticketId}/conversations` |
| Recent activity | `fetchRecentConversationWorkitemEngagements` | `GET /analytics/api/recentConversationWorkitems` |
| Unread badge counts | `fetchUnreadCounts` | `GET /data/api/types/conversation/unread/counts` |
| Aggregate activity | `fetchConversationActivitySummary` | `GET /analytics/api/conversationActivitySummary` |

## Filter and page

`fetchConversations` exposes a wide boolean filter surface — `calls`, `messages`,
`voicemails`, `missedCalls`, `inboundCalls`, `outboundCalls`, `read`, `archived`, `draft`,
`mentioned`, `latest`, `includeCallRecordings`, `includeSummary`, `includeVoicemail`,
`includeParentEngagement`, `ignoreVoicemailTranscripts` — plus `scope`, `tab`, `filter`,
`conversationType`, `participantIds` and `createdAt`.

**Pagination is not uniform.** Use the params the operation you called actually declares:

- `fetchConversations` — `limit` + `offset`, or `cursor`
- `fetchConversationById`, `fetchConversationWorkItemsByStatus`, `fetchTicketConversations` — `pageNumber` + `pageSize`
- `fetchConversationHistoryByParticipantId` — `start` + `rows`

Every list response is the `PaginatedResponse` envelope: `count`, `total`, `objects`.

## Get transcripts and recordings

`fetchTranscriptionsAndRecordings` —
`GET /analytics/api/types/conversation/{conversationId}/workitems/{workitemId}/transcriptions-and-recordings`.
Returns the transcripts (`content`, `startedAt`, `endedAt`, `status`) and
`ConversationCallRecordingData` (`id`, `url`) for that workitem.

For **live** transcription do not poll this operation — subscribe to the SDK WebSocket
stream instead: `sdk.events$.pipe(ofType(EventTypes.LiveTranscriptionNotification))`, with
`LiveTranscriptionCompletedNotification` marking the end of the session. See
`asyncapi/nextiva-events.yml`.

## Rules an agent must follow

- **These are all reads** (`GET`) and safe to retry.
- **Every one of the ten operations declares only a `default` error response.** You cannot
  tell an auth failure from a missing conversation from the contract — inspect `code` and
  `message` in the `{code, message}` body.
- Transcripts and recordings are customer conversation content. Treat the payloads as
  sensitive: Nextiva's platform is sold under HIPAA BAA and PCI scope, so do not persist
  or forward transcript text outside the caller's own trust boundary.
