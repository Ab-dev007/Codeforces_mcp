# Codeforces MCP Server

A Python MCP server that connects Claude and compatible MCP clients to the Codeforces public API. Fetch submissions, ratings, and contest data without manually copying problem details into a conversation.

**Python · FastMCP · httpx · Streamable HTTP**

## Why I built this

Competitive programming practice involves more than submitting a solution. Reviewing the approach and recording what went wrong are useful parts of the process. This server retrieves problem metadata and submission results so the conversation can focus on difficulty, reasoning, and lessons learned.

The server supplies Codeforces data; it does not itself store a problem log or submit solutions.

## Available tools

| Tool | Purpose | Codeforces endpoint |
| --- | --- | --- |
| `get_user_submissions` | Recent submissions, including problem metadata and verdicts | `user.status` |
| `get_user_info` | User profiles, ratings, and ranks | `user.info` |
| `get_user_rating` | Contest rating history | `user.rating` |
| `get_contest_standings` | Contest standings, optionally filtered by handle | `contest.standings` |
| `get_contest_status` | Contest submissions with optional filters | `contest.status` |
| `get_contest_list` | Past and upcoming contests | `contest.list` |

Example requests after connecting an MCP client:

- “Fetch my latest five Codeforces submissions for the handle I provide.”
- “Show my rating changes across recent contests.”
- “List upcoming Codeforces contests.”

These are example prompts, not recorded execution results.

## How it works

```text
Claude / MCP client
        |
        | Streamable HTTP
        v
Python MCP server
        |
        | Async HTTP requests
        v
Codeforces public API
```

The implementation uses a shared `httpx.AsyncClient` with a 30-second timeout, checks HTTP and Codeforces API errors, and returns formatted JSON strings or error messages to the caller. The six tools use public API methods and do not require a Codeforces API key.

## Run locally

Prerequisites: Git and Python 3.11. The commands below use Conda, matching the project's existing setup.

```bash
git clone https://github.com/MindForge-Abhishek/Codeforces_mcp.git
cd Codeforces_mcp

conda create -n codeforces-mcp python=3.11
conda activate codeforces-mcp
pip install -r requirements.txt

python codeforces_mcp.py
```

The server binds to `0.0.0.0:8000`. For a compatible local client, use:

```text
http://127.0.0.1:8000/mcp
```

The current implementation calls `server.streamable_http_app()`. Use the Streamable HTTP `/mcp` endpoint, rather than the legacy SSE `/sse` endpoint.

## Connect from Claude

For a remote Claude connector, deploy the server to an HTTPS host that Claude can reach. Add the deployed address ending in `/mcp` through Claude's connector settings.

When deploying your own instance:

1. Install the dependencies from `requirements.txt`.
2. Start the server with `python codeforces_mcp.py`; the repository also includes a `Procfile`.
3. Configure the hosting service to route requests to port `8000`, which is currently fixed in the code.
4. Add your deployment hostname to `TransportSecuritySettings.allowed_hosts` in `create_server()`.
5. Configure the client to use your HTTPS address with the `/mcp` path.

The existing hostname in the source is specific to the original deployment. Hostname validation is not user authentication; the current implementation does not add application-level authentication or rate limiting.

## Project structure

```text
Codeforces_mcp/
├── codeforces_mcp.py    # API client, six MCP tools, and server entry point
├── requirements.txt    # Python dependencies
├── Procfile            # Deployment start command
├── .gitignore
└── README.md
```

## Current limitations

- Requests depend on Codeforces availability and API limits.
- Errors are returned as readable strings; clients should distinguish them from JSON data.
- The server retrieves data but does not persist reflections or problem logs.
- Deployment configuration includes a fixed port and an explicit host allowlist.

## References

- [Codeforces API documentation](https://codeforces.com/apiHelp)
- [MCP Python SDK](https://github.com/modelcontextprotocol/python-sdk)
