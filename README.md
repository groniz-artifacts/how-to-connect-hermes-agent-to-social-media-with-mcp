# How to Connect Hermes Agent to Social Media With MCP

Connect Hermes Agent to Groniz through MCP, first exposing the tools for listing accounts and reading destination requirements. Add scheduling only when you are ready to review a concrete post. You can then check the account and its current schema before attempting a delivery.

Groniz supports a Hermes MCP connection and handles OAuth, platform formatting, and social delivery. This guide uses that documented route. We have not tested a complete delivery inside Hermes. The setup uses MCP directly and does not require a skill installation URL. For the broader architecture, see the [AI agent publishing setup guide](https://groniz.com/blog/ai-agent-social-media-publishing-setup-a-client-by-client-guide).

## Configure discovery first

Hermes places MCP configuration under `mcp_servers` in `~/.hermes/config.yaml`. Its HTTP configuration accepts a URL and headers, with runtime environment substitution using `${VARIABLE}`. Tool inclusion can restrict which tools reach the agent. Merge this entry into your existing configuration. [Hermes MCP documentation](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp/)

```yaml
mcp_servers:
  groniz:
    url: https://mcp.groniz.com/mcp
    headers:
      Authorization: "Bearer ${GRONIZ_API_KEY}"
    tools:
      include:
        - integrationList
        - integrationSchema
```

Make `GRONIZ_API_KEY` available through the credential mechanism approved for the environment running Hermes. Keep the actual key out of the YAML, shared transcripts, and diagnostic reports. A report can say "environment variable configured" without including its value.

Hermes documents these commands for testing and configuring an MCP server:

```bash
hermes mcp test groniz
hermes mcp configure groniz
```

Use the results to inspect connection and tool availability. [Hermes MCP documentation](https://hermes-agent.nousresearch.com/docs/user-guide/features/mcp/)

A passing connection test confirms that Hermes can reach the server. It does not identify which social account you intend to use, prove a provider accepts the proposed media, or authorize a scheduled post.

## Choose which tools Hermes can use

The initial list lets Hermes discover accounts and their requirements. To schedule a post, you will need to enable another tool.

| Tool | First task | Evidence to keep |
| --- | --- | --- |
| `integrationList` | Find the intended connected account | Account label, provider, and integration ID |
| `integrationSchema` | Read requirements for that integration | Required fields, limits, and applicable settings |
| `schedulePostTool` | Submit a reviewed delivery when later enabled and authorized | Exact request and returned post ID |

The inclusion list controls which tools Hermes can call. The reviewer still needs to approve the specific post before Hermes uses a write tool. Keep the approval instruction attached to the content, destination, media, and time it covers.

Ask Hermes to list integrations, then select the intended account by ID before retrieving its schema. Where two accounts have similar names, pause the preparation until the ID is resolved. Check the account owner as well as the displayed name.

The [MCP server guide](https://groniz.com/blog/social-media-mcp-servers-how-ai-agents-actually-publish-posts) explains why account discovery belongs ahead of platform-specific preparation.

## Prepare the first post for review

Use one small, already reviewed announcement for the first delivery. Ask Hermes to fill out this packet before requesting a write:

```text
Purpose: One approved product announcement
Destination label: [account returned by discovery]
Integration ID: [exact returned ID]
Body: [complete final text, including links]
Media: [uploaded Groniz .path, or none]
Provider settings: [values required by the current schema]
Schedule: [exact ISO timestamp]
Display timezone: [timezone the reviewer expects]
Approval: [who approved which content, destination, and time]
Execution: Awaiting explicit instruction to schedule
```

This is a suggested review document, not a Groniz API payload. The agent still needs to construct the actual tool input from the discovered schema.

For the simplest first run, choose a destination and post type that permit text without media. If media is required or part of the approved post, arrange the upload through an authorized available route, then copy only the returned `.path` into the prepared post. The two discovery tools above cannot upload media.

Check links in the final body, including any tracking parameters the reviewer expects. A later rewrite, media swap, or destination change creates a different packet and needs review appropriate to that change. The [agent-to-channel checklist](https://groniz.com/blog/the-agent-to-channel-publishing-checklist) can support this review.

## Enable and authorize the delivery

When the packet is ready, add `schedulePostTool` to the `include` list using the same indentation as the other tool names. Confirm Hermes can see it. Then authorize the specific reviewed delivery in the conversation or approval mechanism your team uses.

The operator enables the tool, while the reviewer approves the named payload. Keep that approval specific when you reuse the configuration for another announcement.

After submission, preserve the returned post ID and observed state. Record a queued result as queued, then verify publication when delivery is due. If a write returns no clear outcome, reconcile the existing remote records before retrying; a timeout does not prove that nothing was created.

Start by [connecting the intended account in Groniz](https://groniz.com/console/connectors), then use Hermes discovery to build your first packet.
