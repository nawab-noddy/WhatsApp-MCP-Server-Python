# Methods - 1

**Method Name** - **`find_contacts`**

**Purpose** - Find a WhatsApp contact based on information provided by the Service.

**Input** - **`contact_name: str`**

**Return** - A structured contact object containing information the Service needs, for example:

```text
Contact
├── id
├── name
└── phone_number
```
**So conceptually:**
``` text
Service
   │
   │ find_contact("Anas")
   ▼
WhatsAppClient
   │
   │ ← Client handles how WhatsApp actually searches
   ▼
WhatsApp
   │
   ▼
Contact
```

# Method - 2

**Method Name** - **`send_text_message`**

**Purpose** - Sends a standard text message to a specific contact.

**Input** - 
- **`chat_id: str`** (The abstract ID retrieved from your `find_contacts` or `get_recent_chats` methods)
- **`text: str`** (The message body)

**Return** - A **`MessageReceipt`** object

```text
MessageReceipt
├── message_id
├── status (e.g., "SENT", "QUEUED", "FAILED")
└── timestamp
```
# Method - 3

**Method Name** - **`get_recent_chats`**

**Purpose** - Fetches a summary of the most recently active conversations. This is crucial for the AI agent to know what conversations are currently happening or need attention.

**Input** - 
- **`limit: int = 10`**  (How many recent chats to retrieve)

**Return** - A list of **`ChatSummary`** objects

```text
ChatSummary
├── chat_id
├── title (Contact name or Group name)
├── unread_count
└── last_message_preview
```

# Method - 4

**Method Name** - **`get_chat_history`**

**Purpose** - Retrieves the historical messages within a specific conversation. The AI needs this to understand the context of a conversation before drafting a reply.

**Input** - 
- **`chat_id: str`**
- **`limit: int = 20`** (Number of historical messages to fetch)

**Return** - A list of **`Message`** objects, ordered chronologically.

```text
Message
├── message_id
├── sender_id (Who sent it)
├── text_content
├── is_from_me (Boolean)
└── timestamp
```
# Method - 5

**Method Name** - **`get_client_status`**

**Purpose** - Checks the current health and authentication state of the underlying WhatsApp bridge. The Service layer can use this to gracefully tell the user if the phone is disconnected before trying to send a message.

**Input** - None

**Return** - A **`Status`** object

```text
Status
├── is_connected (Boolean)
├── state (e.g., "AUTHENTICATED", "REQUIRES_QR", "OFFLINE")
```