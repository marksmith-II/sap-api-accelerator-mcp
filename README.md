# SAP API Accelerator MCP

An MCP (Model Context Protocol) server that surfaces curated SAP Business Accelerator Hub data as MCP tools. The server talks directly to SAP's public OData catalog, making it easy to discover, search, and explore SAP APIs and artifacts without leaving your MCP-compatible client.

## Purpose of This MCP Server

- Provide AI assistants and MCP tooling with comprehensive read-only access to SAP's vast API catalog.
- Offer powerful search and discovery tools for finding SAP APIs, packages, and artifacts.
- Enable intelligent exploration of SAP's API ecosystem with filtering, analytics, and relationship navigation.
- Demonstrate how to wrap a public OData API with the `mcp.server.fastmcp.FastMCP` helper so it can be inspected, tested, and shipped with any MCP host.

## Prerequisites

- **Python** 3.12 or newer.
- **uv** (recommended) or `pip` + a virtual environment.
- Outbound HTTPS access to `https://api.sap.com/odata/1.0/catalog.svc`.
- (Optional) **Node.js 18+** if you want to use the MCPJam Inspector via `npx`.

## Requirements & Dependencies

All Python requirements are defined in `pyproject.toml`:

- `httpx` – async HTTP client used for SAP API calls.
- `mcp[cli]` – brings in `FastMCP`, CLI helpers, and MCP protocol types.

Install them in editable mode so local code changes are picked up immediately:

```bash
cd sap-api-accelerator-mcp
uv pip install -e .
# or: python -m pip install -e .
```

## Available MCP Tools

The server provides **3 powerful tools** for exploring SAP's API catalog:

### Core Discovery Tools

| Tool | Parameters | Description |
| --- | --- | --- |
| `list_sap_content_packages` | `search_term` (optional), `max_results` (default: 100) | Lists all content packages with optional search filtering. Enhanced with OData filtering and result limiting. |
| `get_sap_package_info` | `package_id` | Get detailed information about a specific content package. |
| `get_sap_artifact_details` | `artifact_name`, `artifact_type` (default: API) | Get complete details for a specific artifact including all metadata. |

All tools normalize OData v2/v4 responses, force JSON output (`$format=json`), and include defensive error handling so failures are reported rather than crashing the MCP session.

## How to Run the MCP Server

```bash
cd sap-api-accelerator-mcp
uv run accelerator.py
# or
python accelerator.py
```

The script registers both tools with `FastMCP` and starts an MCP stdio server (the default transport). Any MCP-compatible client can now spawn this process and call the tools.

## Testing with MCPJam Inspector

MCPJam Inspector offers a browser UI for exercising MCP servers without wiring up an LLM. Typical workflow:

1. **Launch the inspector**
   ```bash
   npx @mcpjam/inspector@latest
   ```
   The CLI prints a URL such as `http://localhost:6274`. Open it in your browser.

2. **Add the SAP accelerator server**
   - Click **Add Server** → **Custom / stdio**.
   - Command: `uv run accelerator.py` (or `python accelerator.py`).
   - Provide a friendly name (e.g., “SAP Accelerator”) and save. The status should flip to **Connected**.

3. **Exercise the tools**
   - Open the **Tools** tab, choose `list_sap_content_packages`, and click **Execute** to confirm catalog access.
   - Pick `get_sap_package_info`, supply a `TechnicalName` from the previous response, and execute again to view package details.

4. **Inspect traffic / troubleshoot**
   - Use the **Messages** view to see raw MCP JSON exchange.
   - If a call fails, MCPJam surfaces the stderr logs (the server prints every SAP request it makes) to help diagnose connectivity or parameter issues.

## Use Case Examples

### Example 1: Explore a Specific Package
```
get_sap_package_info(package_id="SAPS4HANACloud")
```
Get detailed information about a specific package.

### Example 2: Get Detailed Artifact Information
```
get_sap_artifact_details(artifact_name="CE_PROJECTDEMANDCATEGORY_0001", artifact_type="API")
```
Get complete details for an artifact.

## Tips & Best Practices
- **Set `max_results`** appropriately - default values are conservative to avoid timeouts. Increase for comprehensive searches.
- **Filter by `state="ACTIVE"`** to exclude deprecated APIs unless you're specifically looking for them.
- **Use `get_sap_artifact_details`** after finding interesting artifacts to get complete metadata.
- **Combine tools** - start with search, then drill down with detail tools.
- Because the SAP endpoints are public, no API key is required, but the server still sends a custom `User-Agent` so rate limiting can be monitored.
- Pair this server with any MCP host (Anthropic Claude Desktop, MCP CLI, custom LLM sandboxes) to let assistants browse SAP content.

## Tool Priority Guide

**Start with these tools:**
1. `list_sap_content_packages` - Discover available packages
2. `get_sap_package_info` - Package details
3. `get_sap_artifact_details` - Detailed artifact information


## Performance Considerations

- Large result sets may take time - use `max_results` to limit responses
- The SAP API may be slow for complex queries - be patient
- Use `$select` in queries (already implemented) to reduce payload size
- Consider caching results for frequently accessed data


