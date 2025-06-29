# MCP Server Deployment for HTTP Serving - macOS Setup + Code

This project demonstrates how to build and run a local **MCP server** that exposes tools, and how to test those tools using **MCP Inspector** through a browser-based interface.

You'll define and run two simple tools on the server:
- One to search papers from arXiv based on a topic
- One to extract details about a specific paper

The project runs entirely on your local **macOS** system using **Python 3.13.2**. 

The server is implemented using the Model Context Protocol (MCP), and the setup is already initialized with [`uv`](https://github.com/astral-sh/uv). After cloning the repository, you’ll create a virtual environment, install dependencies, and launch the MCP server and Inspector with a single command.

This is a self-contained example for learning how MCP servers are defined, how tools are registered, and how to interactively test them using the Inspector UI.


## Project Workflow Overview

```mermaid
flowchart TD
    A[Clone Repository] --> B[Set Up Virtual Environment]
    B --> C[Install Project Dependencies]
    C --> D[Run MCP Server with Inspector]
    D --> E[Open Inspector in Browser]
    E --> F[Interact with MCP Tools Visually]
````

## 1. Clone and Set Up the Environment

Clone this repository:

```bash
git clone https://github.com/shaikhq/mcpworks.git
cd mcpworks
```

Create a virtual environment using Python 3.13:

```bash
uv venv --python /usr/local/bin/python3.13
```

Activate the environment:

```bash
source .venv/bin/activate
```

Confirm your Python version:

```bash
python --version
# Should show Python 3.13.x
```

## 2. Install Dependencies

```bash
uv add mcp arxiv
```

This will install the `mcp` runtime and `arxiv` client library required to run the server.

## 3. Run the MCP Server
```shell
uv run research_server_http.py
```

The above command will start the MCP server at the following endpoint: 
http://127.0.0.1:8000

![alt text](image.png)

## 4. Run the MCP Inspector

From another terminal window.

Launch MCP Inspector without authentication. For most production deployments, you will likely need to setup authentication. However, in my current exercise, my focus was to deploy a MCP server with HTTP end point. So, I haven't enabled authentication yet.

```bash
export DANGEROUSLY_OMIT_AUTH=true
npx @modelcontextprotocol/inspector
```

![alt text](image-1.png)

* MCP Inspector runs a local UI for browsing and testing tools
* The MCP server runs your tool code and handles requests

Once started, open your browser to:

```
http://127.0.0.1:6274/
```

You can now interact with your tools from the browser interface.

In the URL field of the inspector, type the HTTP URL of the MCP server and click connect. Then go to the tools tab to see the tools from the MCP server and try them out. 

![alt text](image-2.png)
