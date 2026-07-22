# Frameworks & infrastructure

## Agent frameworks

- **LangChain** — modular components; a bit verbose; large ecosystem. Noted as the recommended default.
- **LangGraph** — orchestration by graph; precise control but harder to read; suited to a more symbolic agent style.

## API layer

- **Flask vs. FastAPI** — open question for exposing the agent as a service.

## MCP

- Run an MCP server for tool discovery; think of it as client and server.
- See [architecture](../02-design/architecture.md) for where MCP sits in the layers.
