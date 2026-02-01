# Changelog - Photo Events Project

## [2026-02-01] - Analysis, Hardening, and Expansion of MCP Calendar

### Changes in `calendar-mcp`:
- **Expanded Reminder Functionality**:
    - Updated `src/models.py` by adding the `reminders` field to the `EventUpdateRequest` model.
    - Modified `src/calendar_actions.py` to enable updating reminders for existing events.
    - Extended the MCP bridge in `src/mcp_bridge.py` by adding `reminder_minutes` and `reminder_methods` parameters to the `create_event` and `update_event` tools.
- **Network Security Enhancement**: Modified `src/server.py` to change the uvicorn listening address from `0.0.0.0` to `127.0.0.1`.
- **Stability Improvements**: Disabled `reload` mode in `run_server.py` to prevent server restart loops caused by logging.
- **Security Audit**: Verified the OAuth 2.0 implementation, dependencies, and source code for vulnerabilities.