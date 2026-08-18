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