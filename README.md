# Nextiva

Nextiva is a Scottsdale, Arizona cloud communications and customer experience company. Its
NextOS / NEXT platform combines UCaaS business phone service, contact center (NCX), SMS and
team messaging, voice AI agents and conversation analytics.

- Website: https://www.nextiva.com/
- Developer portal: https://developer.nextiva.com/nextiva/
- Status: https://status.nextiva.com/
- GitHub: https://github.com/nextiva

## APIs

Five REST OpenAPI 3.0 contracts, harvested from the ReadMe API Registry behind
developer.nextiva.com and served from `https://api.nextiva.com`:

| API | Ops | Spec |
|---|---|---|
| Provider Authentication | 2 | `openapi/nextiva-authentication-openapi.yml` |
| Provider Token Service | 4 | `openapi/nextiva-provider-authentication-openapi.yml` |
| Workitem Service | 12 | `openapi/nextiva-workitem-service-openapi.yml` |
| Conversation | 10 | `openapi/nextiva-conversation-openapi.yml` |
| SMS Messaging | 1 | `openapi/nextiva-sms-messaging-openapi.yml` |

Auth is HTTP basic → JWT bearer. No OAuth, no OIDC, no API keys.

## Events

No AsyncAPI and no customer-configurable webhooks. Events reach clients over two WebSocket
channels the SDK opens after login — see `asyncapi/nextiva-events.yml`.

## Artifacts

`openapi/` `overlays/` `packages/` `authentication/` `conventions/` `errors/`
`conformance/` `lifecycle/` `data-model/` `asyncapi/` `mcp/` `skills/` `llms/`
`security/` `well-known/`
