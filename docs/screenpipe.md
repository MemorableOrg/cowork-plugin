# Use Screenpipe context with Memorable

[Screenpipe](https://github.com/screenpipe/screenpipe) can supply recorded desktop
and meeting context to a task whose procedure is recalled by Memorable. This
recipe connects both to Claude Code on your computer. Install and enable the
Memorable plugin using the [README](../README.md#install) first.

## Connect the local context source

Keep the Screenpipe desktop app running on the same computer as the MCP client.
You need Node.js and `npx` on your PATH. Obtain your local API key with
`screenpipe auth token`, or use the key configured by the desktop app. Put the
key in `SCREENPIPE_LOCAL_API_KEY` in the terminal where you start the bridge.
Do not paste it into a chat or commit it to a repository.

```bash
npx -y screenpipe-mcp@0.20.2 --http --port 3031
```

The bridge binds to loopback by default and forwards searches to Screenpipe on
port 3030. Leave `--listen-on-lan` off for this setup. If port 3031 is occupied,
choose another free port and use it in the client configuration too.

The HTTP transport exposes the `search_content` tool. Use a narrow time range,
a small limit, and a specific query; an empty result means no matching evidence
was returned. The full stdio tool set has different tool names, so inspect the
connected server's tools rather than substituting a stdio example.

In another terminal, add the bridge to Claude Code:

```bash
claude mcp add --transport http screenpipe http://127.0.0.1:3031/mcp
```

Restart the Claude Code session if necessary. Confirm that its tool list includes
Screenpipe's `search_content` and Memorable's `memorable_recall` and
`memorable_show`. This local URL is for an agent running on the same computer;
a hosted connector cannot reach your computer's loopback address.

## Use the two sources in one task

For example, when preparing a recurring project handoff:

1. Recall the relevant handoff procedure from Memorable, and check it still fits
   the current request.
2. Search Screenpipe for the selected project's recent work. Cite the returned
   timestamps when using that evidence in a draft.
3. Complete the requested task and check the outcome. The plugin's existing
   prompt-boundary hook can record the completed agent session when consent is
   enabled; adding Screenpipe does not introduce another recording hook.
4. On a later handoff, recall the procedure again, then retrieve fresh project
   context from Screenpipe.

Screenpipe observations do not establish which actions succeeded. Do not turn
OCR text or a meeting summary into invented successful tool calls. For a custom
harness that has actual tool-call results, use Memorable's documented
[trace ingestion API or CLI](https://www.memorable.sh/doc/integrate).

## Data and removal

Screenpipe search results enter the agent's context and may be processed by the
agent's configured model provider. Memorable's existing capture and consent
rules still apply. This recipe does not bulk-export Screenpipe history, create a
background sync, or reconcile deletions between the products.

Remove the client connection with `claude mcp remove screenpipe`, then stop only
the bridge process started for this recipe. This does not stop the Screenpipe
recorder or remove Memorable procedures.

See the [Screenpipe MCP documentation](https://github.com/screenpipe/screenpipe/tree/main/packages/screenpipe-mcp)
for authentication and other transports.
