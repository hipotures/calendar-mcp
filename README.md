# Google Calendar MCP Server (Python)

This project implements a Python-based MCP (Model Context Protocol) server that acts as an interface between Large Language Models (LLMs) and the Google Calendar API. It enables LLMs to perform calendar operations via natural language requests.

## Features

*   **Authentication:** Secure Google Calendar API access using OAuth 2.0 (Desktop App Flow with automatic token storage/refresh).
*   **Core Calendar Actions:**
    *   List calendars (`list_calendars`).
    *   Create calendars (`create_calendar`).
    *   Find events with basic and advanced filtering (`find_events`).
    *   Create detailed events (`create_event`).
    *   Quick-add events from text (`quick_add_event`).
    *   Update events (`update_event`).
    *   Delete events (`delete_event`).
    *   Add attendees to events (`add_attendee`).
*   **Advanced Scheduling & Analysis:**
    *   Check attendee response status (`check_attendee_status`).
    *   Query free/busy information for multiple calendars (`query_free_busy`).
    *   Find mutual free slots and schedule meetings automatically (`schedule_mutual`).
    *   Analyze daily event counts and durations (`analyze_busyness`).
    *   *(Recurring event projection feature potentially added in Task 3.5, but not explicitly exposed as a tool yet)*
*   **Server:** FastAPI-based server exposing actions via a RESTful API.
*   **MCP Integration:** Provides MCP-compatible tools via stdio using `FastMCP`.

## MCP Tools (Current Names)

The MCP server currently exposes these tool names:

*   `list_calendars`
*   `find_events`
*   `create_event`
*   `quick_add_event`
*   `update_event`
*   `delete_event`
*   `add_attendee`
*   `check_attendee_status`
*   `query_free_busy`
*   `schedule_mutual`
*   `analyze_busyness`
*   `create_calendar`

## Setup

1.  **Prerequisites:**
    *   Python 3.8+ installed.
    *   Git installed.
    *   Access to a Google Cloud Platform project.

2.  **Clone Repository:**
    ```bash
    git clone <repository-url> # Replace with your repo URL
    cd <repository-directory>
    ```

3.  **Google Cloud Setup (OAuth Credentials):**
    *   Go to the [Google Cloud Console](https://console.cloud.google.com/).
    *   Create a new project or select an existing one.
    *   **Enable the Google Calendar API** for your project.
    *   Navigate to "APIs & Services" > "Credentials".
    *   Click "+ CREATE CREDENTIALS" > "OAuth client ID".
    *   Select **Application type: Desktop app**. Give it a name (e.g., "Calendar MCP Local").
    *   Click "CREATE". A pop-up will show your **Client ID** and **Client Secret**. **Copy these now** - you'll need them for the `.env` file. You *do not* need to download the JSON file offered for other app types.
    *   Configure the **OAuth consent screen**:
        *   Set User Type to "External".
        *   Fill in required app info (App name, User support email, Developer contact).
        *   Add Scopes: Click "Add or Remove Scopes", search for `calendar`, add the `.../auth/calendar` scope (read/write access). Click "Update".
        *   Add Test Users: Add the Google Account email address(es) you will authenticate with.
        *   Save and return to the dashboard.
    *   Go back to "APIs & Services" > "Credentials" and click on the name of the Desktop App credential you created.
    *   Under "Authorized redirect URIs", click "+ ADD URI" and enter `http://localhost:8080/`. Click "Save". (Adjust the port if you change `OAUTH_CALLBACK_PORT` in `.env`).
    *   If your OAuth setup uses a path-based callback, also add `http://localhost:8080/oauth2callback`.

4.  **Environment Configuration (`.env` file):**
    *   In the project's root directory, copy the `example.env` file and rename the copy to `.env`.
    *   Open the `.env` file and paste your **Client ID** and **Client Secret** obtained from Google Cloud:
        ```dotenv
        # Google OAuth 2.0 Client Credentials (from Google Cloud Console - Desktop app type)
        GOOGLE_CLIENT_ID='YOUR_GOOGLE_CLIENT_ID_HERE'
        GOOGLE_CLIENT_SECRET='YOUR_GOOGLE_CLIENT_SECRET_HERE'

        # Path to the file where the user's OAuth tokens will be stored after first auth
        # This file is created automatically. Default is .gcp-saved-tokens.json
        TOKEN_FILE_PATH='.gcp-saved-tokens.json'

        # Port for the local webserver during the OAuth callback (must match Google Cloud redirect URI)
        OAUTH_CALLBACK_PORT=8080

        # Google Calendar API Scopes (read/write is default)
        # Use 'https://www.googleapis.com/auth/calendar.readonly' for read-only access
        CALENDAR_SCOPES='https://www.googleapis.com/auth/calendar'
        ```
    *   Ensure `TOKEN_FILE_PATH` points to a location where the app can write the token file (the default `.gcp-saved-tokens.json` in the root is usually fine). This file is automatically added to `.gitignore`.

5.  **Install Dependencies:**
    *   Navigate to the project directory in your terminal.
    *   Install the required Python packages:
        ```bash
        pip install -r requirements.txt
        ```
    *   *(Using a Python virtual environment is recommended but optional)*

## Running the Server (for Initial Authentication & Testing)

You only need to run the server manually **once** to complete the initial Google OAuth authentication flow. After that, your MCP client will launch the server automatically using the command specified in its configuration.

1.  **First Run (Authentication):**
    *   Run the server script from your terminal:
        ```bash
        python run_server.py
        ```
    *   The script checks for saved tokens (`.gcp-saved-tokens.json`). Since they don't exist yet, it will:
        *   Print an authorization URL.
        *   Automatically open your browser to that URL.
        *   Guide you through logging into your Google Account and granting calendar permissions.
        *   After you grant permissions, Google redirects back to a local URL (`http://localhost:8080/` by default, or `/oauth2callback` if configured that way).
        *   The script captures the authorization code and **saves the necessary tokens** to the file specified in `.env` (`.gcp-saved-tokens.json`).
    *   Once tokens are saved, the script will typically start the FastAPI server (e.g., on `http://localhost:8000`). You can usually stop it (Ctrl+C) after you see confirmation that tokens were saved or the server has started.

2.  **Optional: Testing the Server Directly:**
    *   If you want to test the FastAPI server directly (e.g., by sending HTTP requests with tools like `curl` or Postman), you can run `python run_server.py` again. It will load the saved tokens and start the server without requiring browser authentication.

**Note:** For regular use with an MCP client, you **do not** need to run `python run_server.py` manually after the initial authentication. The client handles launching it.

## MCP Client Configuration

To use this server as a tool within an MCP client, you need to configure the client to run the `run_server.py` script. This is typically done in a JSON settings file.

**Example `mcp.json` Entry:**

```json
{
  "mcpServers": {
    "calendar-mcp": {
      "command": "python",
      "args": [
        "C:/path/to/your/calendar-mcp/run_server.py"
      ],
      "env": {
        "PYTHONPATH": "C:/path/to/your/calendar-mcp"
      }
    }
  }
}
```

**Configuration Details:**

*   **`calendar-mcp`:** A unique name you choose for this tool instance within your MCP client.
*   **`command`:** Set this to `python` if it's in your system's PATH. If not, provide the *full absolute path* to the `python.exe` or `python` executable (e.g., `/path/to/your/venv/bin/python` or `C:/path/to/your/venv/Scripts/python.exe`).
*   **`args`:** Provide the *full, absolute path* to the `run_server.py` script in your project directory. **Replace the placeholder `/path/to/your/calendar-mcp/run_server.py` with the actual path on your system.**
*   **`env.PYTHONPATH`:** Set to the project root path so imports in `src/` are resolved consistently.
*   **(Optional) `api`:** Some clients might still need the `api` field to point to the underlying FastAPI server (e.g., `"api": "http://localhost:8000"`) for schema discovery, even though communication happens via stdio.
*   **(Optional) `timeout`:** You can add a timeout (e.g., `"timeout": 30000` for 30 seconds).

**How it Works:** When the MCP client invokes this tool, it executes the specified `command` with the `args`. The `run_server.py` script detects it's being run this way (via piped stdin/stdout) and automatically starts the MCP communication bridge instead of just the HTTP server.

**Important:**
*   Your Google Client ID/Secret remain secure in your project's `.env` file, *not* in the MCP client configuration.
*   Consult your specific MCP client's documentation for the exact configuration file location and required fields.

### Codex CLI Local Project Config (Optional)

If you use Codex CLI with per-project config, you can place this in `.codex/config.toml` inside the repository:

```toml
[mcp_servers.calendar-mcp]
command = "/home/xai/DEV/photo-events/.venv/bin/python"
args = ["/home/xai/DEV/photo-events/run_server.py"]
env = { PYTHONPATH = "/home/xai/DEV/photo-events" }
startup_timeout_ms = 20_000
```

## Quick Usage Examples (Tool Calls)

### 1) Create event with reminders

```json
{
  "name": "calendar-mcp__create_event",
  "args": {
    "calendar_id": "9pvcn5nlplvejrc9npeco7u6a4@group.calendar.google.com",
    "summary": "Sesja Zdjeciowa DWC",
    "start_time": "2026-03-23T08:00:00",
    "end_time": "2026-03-23T22:00:00",
    "location": "Nowohuckie Centrum Kultury, Krakow",
    "description": "Organizator: L'Art de la Danse",
    "reminder_minutes": [10080, 1440],
    "reminder_methods": ["popup", "popup"]
  }
}
```

### 2) Quick add

```json
{
  "name": "calendar-mcp__quick_add_event",
  "args": {
    "calendar_id": "primary",
    "text": "Spotkanie z klientem jutro o 10:00"
  }
}
```

### 3) Update event reminders

```json
{
  "name": "calendar-mcp__update_event",
  "args": {
    "calendar_id": "primary",
    "event_id": "EVENT_ID_HERE",
    "reminder_minutes": [1440, 60],
    "reminder_methods": ["popup", "popup"]
  }
}
```

### Datetime and Timezone Notes

*   `create_event` and `update_event` expect ISO date-time strings in `YYYY-MM-DDTHH:MM:SS`.
*   To avoid timezone ambiguity, prefer offset-aware values when possible (e.g., `2026-03-23T08:00:00+01:00`).
*   Calendar-level timezone settings still apply on Google side.

## Access Verification Checklist

Before creating events, verify access in this order:

1.  Ensure MCP server is visible in your client (`calendar-mcp`).
2.  Call `list_calendars`.
3.  Confirm expected calendar appears (e.g., `primary` or target group calendar id).
4.  Only then call `create_event`.

## OAuth Troubleshooting

### `invalid_grant: Bad Request`

This usually means the refresh token is invalid/revoked or consent changed.

1.  Back up token file:
    ```bash
    cp .gcp-saved-tokens.json .gcp-saved-tokens.json.bak
    ```
2.  Remove or rename the current token file:
    ```bash
    mv .gcp-saved-tokens.json .gcp-saved-tokens.json.invalid-old
    ```
3.  Re-run auth flow:
    ```bash
    python run_server.py
    ```
4.  Verify with `list_calendars`.

### `insufficientPermissions` (HTTP 403)

Token exists but OAuth scope is too narrow.

1.  Check `.env`:
    ```dotenv
    CALENDAR_SCOPES='https://www.googleapis.com/auth/calendar'
    ```
2.  Ensure OAuth consent screen includes Calendar scope.
3.  Re-authenticate to mint a new token with updated scope.

### `Google API credentials unavailable` (HTTP 503 from MCP tool)

MCP bridge is running, but backend cannot obtain valid OAuth credentials.

1.  Complete the re-authentication steps above.
2.  Confirm token file exists and is fresh.
3.  Call `list_calendars` again.

## Development

*   **Code Structure:**
    *   `run_server.py`: Main entry point, handles server startup and MCP detection.
    *   `src/server.py`: FastAPI application definition, HTTP endpoints.
    *   `src/calendar_actions.py`: Core logic interacting with the Google Calendar API.
    *   `src/analysis.py`: Advanced analysis functions.
    *   `src/auth.py`: Handles OAuth 2.0 authentication flow and token management.
    *   `src/models.py`: Pydantic models for request/response data structures.
    *   `src/mcp_bridge.py`: Implements the MCP tool definitions using `mcp_sdk`, delegating to the FastAPI server.
*   **Logging:** Logs are written to `calendar_mcp.log` in the project root.
*   **Testing:** (Details TBD)
*   **Contributing:** (Details TBD)

## Next Steps (Planned Tasks)

*   Implement MCP Resources/Prompts support (Task 6.1, 6.2).
*   Enhance MCP tool argument validation and response formatting (Task 6.3, 6.4).
*   Improve MCP error handling (Task 6.5).
*   Refine development workflow (Task 7).

## License

This project is dual-licensed to support both open-source collaboration and sustainable development:

1.  **GNU Affero General Public License v3.0 (AGPL-3.0):**
    *   This software is free to use, modify, and distribute under the terms of the AGPLv3 license. 
    *   Key conditions include that derivative works (including modifications used over a network) must also be licensed under AGPLv3 and their source code made available.
    *   This license is suitable for open-source projects or internal use where AGPLv3 compliance is feasible.
    *   See the [LICENSE](LICENSE) file for the full text.

2.  **Commercial License:**
    *   If the terms of the AGPLv3 are not suitable for your specific use case (e.g., integrating this software into a proprietary, closed-source commercial product or service without complying with AGPLv3's source-sharing requirements), a separate commercial license is available.
    *   Please contact **deciduusleaf@gmail.com** for inquiries regarding commercial licensing options.

By using, modifying, or distributing this software, you agree to be bound by the terms of either the AGPLv3 or a separately negotiated commercial license. 
