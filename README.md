# MCP Client with LLM Integration

This project implements a client that interacts with multiple MCP (Meta-Control Protocol) servers, leverages an LLM (Language Model), and provides a chat-based interface for users.  It allows users to leverage tools exposed by MCP servers by interacting with the LLM, which orchestrates tool execution based on user input.

## Features

*   **MCP Server Interaction:** Connects to multiple MCP servers and executes tools.
*   **LLM Integration:** Uses an LLM (e.g., Groq) to understand user requests and determine which tools to use.
*   **Chat Interface:**  Provides a command-line interface for users to interact with the system.
*   **Tool Discovery:** Dynamically discovers available tools from connected MCP servers.
*   **Configuration Management:** Loads server configurations from a JSON file and API keys from environment variables.
*   **Error Handling and Retries:** Implements robust error handling and retry mechanisms for tool execution.
*   **Asynchronous Operations:** Uses `asyncio` for concurrent and efficient execution.
*   **Tool Formatting for LLM:** Formats tool information into a structured format for the LLM to understand and use.


## Requirements

- Python 3.10
- `python-dotenv`
- `requests`
- `mcp`
- `uvicorn`

## Installation

1.  **Clone the repository:**

    ```bash
    git clone <repository_url>
    cd <repository_directory>
    ```

2.  **Create a virtual environment (recommended):**

    ```bash
    python3 -m venv venv
    source venv/bin/activate  # On Linux/macOS
    # venv\Scripts\activate  # On Windows
    ```

3.  **Install the dependencies:**

    ```bash
    pip install httpx python-dotenv
    # If you have mcp locally
    pip install <path/to/mcp>
    # Or if MCP is on pypi (replace <version> with a valid version string):
    # pip install mcp==<version>
    ```

## Configuration

1.  **Set up environment variables:**

    Create a `.env` file in the project's root directory and add the LLM API key:

    ```
    LLM_API_KEY=<your_llm_api_key>
    ```

2.  **Configure MCP servers:**

    Create a `servers_config.json` file in the project's root directory.  This file should contain a JSON object with a `mcpServers` key. Each entry in `mcpServers` defines the configuration for an MCP server.  Here's an example:

    ```json
    {
    "mcpServers": {
      "sqlite": {
        "command": "uvx",
        "args": ["mcp-server-sqlite", "--db-path", "./test.db"]
      },
      "puppeteer": {
        "command": "npx",
        "args": ["-y", "@modelcontextprotocol/server-puppeteer"]
      }
    }
  }
    ```

    *   `command`: The command to execute to start the MCP server. This can be a full path, or a command available in your system's PATH (like `npx`).
    *   `args`: A list of arguments to pass to the command.
    *   `env`: (Optional) A dictionary of environment variables to set for the server process. These variables will be merged with the existing environment variables.

    **Important**: Ensure that `npx` is available in your system's PATH if you use it. You may need to install Node.js and npm. If you are directly pointing to an executable, ensure its location is correct and the script has execute permissions.


## Usage

1.  **Run the client:**

    ```bash
    python main.py
    ```

2.  **Interact with the chat interface:**

    The client will start and present a "You:" prompt. Enter your questions or commands.  The LLM will analyze your input, determine if a tool needs to be used, and execute the tool if necessary.  The response from the LLM (or the tool's result) will be displayed.

3.  **Exit the client:**

    Type `quit` or `exit` and press Enter.

## Code Structure

*   **`main.py`:** The main entry point of the application.
*   **`Configuration` Class:** Handles loading configuration from `.env` files and JSON files.
*   **`Server` Class:** Manages connections to MCP servers, initializes them, lists available tools, executes tools, and handles cleanup.
*   **`Tool` Class:** Represents a tool with its name, description, and input schema. It also formats tool information for the LLM.
*   **`LLMClient` Class:**  Communicates with the LLM provider (e.g., Groq) to get responses based on user input and tool results.
*   **`ChatSession` Class:** Orchestrates the interaction between the user, the LLM, and the MCP servers. It handles tool execution, response processing, and the overall chat flow.

## Example Interactions

**User:** What's the weather in San Francisco?

*The LLM determines that a weather tool is needed and generates the following JSON:*

```json
{
    "tool": "get_weather",
    "arguments": {
        "location": "San Francisco"
    }
}
```

### Logic Flow

1. **Tool Integration**:
   - Tools are dynamically discovered from MCP servers
   - Tool descriptions are automatically included in system prompt
   - Tool execution is handled through standardized MCP protocol

2. **Runtime Flow**:
   - User input is received
   - Input is sent to LLM with context of available tools
   - LLM response is parsed:
     - If it's a tool call → execute tool and return result
     - If it's a direct response → return to user
   - Tool results are sent back to LLM for interpretation
   - Final response is presented to user