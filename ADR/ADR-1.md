# Title - Selection of WhatsApp Integration Strategy

# Context
Our system requires an autonomous integration layer to interact with a personal WhatsApp account from a Python-based MCP server. The integration must:
- Support standard authentication workflows (QR code or pairing code) without requiring an interactive desktop display.
- Handle bidirectional operations: sending messages, retrieving conversation history, and listening to inbound events.
- Provide clean encapsulation so the Python MCP server remains loosely coupled to the underlying messaging transport.
- Maintain low infrastructure overhead and stable 24/7 background execution.
# Decision
Adopt Baileys `@whiskeysockets/baileys` running as a lightweight Node.js sidecar service, integrated with the Python MCP server via a local REST/Streamble HTTP bridge and a dedicated Python Client abstraction layer.
## Core Implementation Strategy:
- **Bridge Layer (Transport):** Run a dedicated, headless Baileys process in Node.js (or a pre-packaged engine like WAHA) exposing an internal REST API for outbound requests and Streamble HTTP or Webhooks for incoming messages.

- **Client Abstraction Layer (Python):** Build a strongly typed `client/whatsapp_client.py` class in Python that implements rate limiting, session reconnection handling, error mapping, and response sanitization.

- **MCP Tool Layer:** Expose domain-level MCP tools (e.g., send_message, list_recent_chats, get_chat_history) that call the WhatsAppClient, ensuring zero direct dependency on the underlying WhatsApp Web protocol.

# Alternatives Considered

| Option | Architecture / Language | Resource Overhead | Inbound Event Handling | Personal Account Viability | Project Suitability |
| :--- | :--- | :--- | :--- | :--- | :--- |
| **Baileys** | Protocol WebSocket (TypeScript / Node.js) | Very Low (~30–80 MB RAM) | Native (Persistent WebSocket stream) | High (Multi-device link) | Selected |
| **OpenWA** | Browser Automation / Puppeteer (Node.js) | Medium–High (~300–600 MB RAM per session) | Good (Browser DOM hooks / Webhooks) | High (Multi-device link) | Viable fallback, but heavier infrastructure cost |
| **WhatsApp Cloud API** | Official Graph REST API (Hosted by Meta) | Zero client runtime (Cloud managed) | Native (HTTPS Webhooks) | None (Requires dedicated business phone number) | Best for enterprise, but rejected due to personal account constraint |
| **Selenium** | Direct Browser Scraping (Python) | High (~400–800 MB RAM + Chrome) | Poor (Requires aggressive DOM polling) | Medium (High ban risk) | Rejected (Brittle, DOM-churn prone) |
| **pywhatkit** | GUI / OS Automation Wrapper (Python) | Uncontrolled (Requires active OS desktop window) | None (Send-only) | Low (Simulates keyboard/mouse) | Rejected (Unsuitable for server/headless environments) |

# Consequences
## Positive
- **Operational Efficiency:** Running a protocol-level WebSocket engine avoids the overhead and instability of headless Chromium instances.

- **Clean Architectural Separation:** The Python MCP server only interacts with standardized REST/Streamble HTTP contracts, keeping AI tool definitions clean and testable.

- **Non-Blocking Execution:** Asynchronous event streaming ensures incoming messages trigger MCP agent events with minimal latency.

- **Mockable Testing:** Isolating WhatsApp logic behind a Python client interface enables deterministic unit testing and CI/CD validation without live WhatsApp connections.
## Negative 
- **Two-Runtime Deployment:** The deployment environment must orchestrate both Python (MCP Server) and Node.js (Baileys bridge) runtimes (e.g., via Docker Compose or multi-stage containers).

- **Protocol Drift Risk:** As an unofficial library, changes to WhatsApp's internal protocol may require updating the Baileys package when Meta enforces new protocol revisions.

- **ToS / Account Risk:** Automating personal WhatsApp accounts carries ban risks if outbound message rate limits and natural cadence thresholds are violated.
## Future Works
- **Client Rate-Limiting & Jitter:** Implement a token-bucket rate limiter and randomized delay generator within `client/whatsapp_client.py` to prevent anti-spam trigger flags.

- **Session Persistence:** Configure volume-backed session credential storage (e.g., SQLite / encrypted auth files) to maintain authentication across container restarts.

- **Cloud API Adapter:** Implement the `WhatsAppClient` base interface such that switching from Baileys to the official WhatsApp Cloud API requires changing only configuration flags without modifying MCP tool implementations.