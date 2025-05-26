# MCP Server and MCP Inspector Setup on macOS

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

## 3. Run the MCP Server with MCP Inspector

Launch MCP Inspector and the MCP server in a single command:

```bash
npx @modelcontextprotocol/inspector uv run research_server.py
```

* MCP Inspector runs a local UI for browsing and testing tools
* The MCP server runs your tool code and handles requests

Once started, open your browser to:

```
http://127.0.0.1:6274/
```

You can now interact with your tools from the browser interface.

## 4. Example MCP Server Implementation

**Credit:**
This script is adapted from the [DeepLearning.AI course on MCP](https://learn.deeplearning.ai/courses/mcp-build-rich-context-ai-apps-with-anthropic), developed in collaboration with Anthropic.

File: `research_server.py`

```python
from mcp.server.fastmcp import FastMCP
import arxiv
import os
import json
from typing import List

mcp = FastMCP("research")
PAPER_DIR = "papers"

@mcp.tool()
def search_papers(topic: str, max_results: int = 5) -> List[str]:
    client = arxiv.Client()
    search = arxiv.Search(query=topic, max_results=max_results, sort_by=arxiv.SortCriterion.Relevance)
    papers = client.results(search)

    path = os.path.join(PAPER_DIR, topic.lower().replace(" ", "_"))
    os.makedirs(path, exist_ok=True)
    file_path = os.path.join(path, "papers_info.json")

    try:
        with open(file_path, "r") as f:
            papers_info = json.load(f)
    except (FileNotFoundError, json.JSONDecodeError):
        papers_info = {}

    paper_ids = []
    for paper in papers:
        paper_ids.append(paper.get_short_id())
        papers_info[paper.get_short_id()] = {
            'title': paper.title,
            'authors': [a.name for a in paper.authors],
            'summary': paper.summary,
            'pdf_url': paper.pdf_url,
            'published': str(paper.published.date())
        }

    with open(file_path, "w") as f:
        json.dump(papers_info, f, indent=2)

    return paper_ids

@mcp.tool()
def extract_info(paper_id: str) -> str:
    for item in os.listdir(PAPER_DIR):
        path = os.path.join(PAPER_DIR, item, "papers_info.json")
        if os.path.exists(path):
            with open(path, "r") as f:
                data = json.load(f)
                if paper_id in data:
                    return json.dumps(data[paper_id], indent=2)
    return f"No data found for paper {paper_id}."

if __name__ == "__main__":
    mcp.run()
```

## Notes

* MCP Inspector runs on port 6274 and proxies requests to the MCP server.
* The `.venv` created with `uv` is local and excluded from version control via `.gitignore`.
* You do **not** need a `.python-version` file; this project uses the activated virtual environment.
