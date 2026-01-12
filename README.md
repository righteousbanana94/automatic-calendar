# Google Calendar Assistant (n8n + AI + Telegram)

This is an AI-powered calendar assistant built with [n8n](https://n8n.io). It uses Telegram and Gemini (or OpenAI) to receive natural language inputs, check availability via Google Calendar, and schedule events only when no conflicts exist. All replies are clean, structured, and strictly functional.

## Features

- Natural language input via Telegram
- AI assistant responds with structured JSON only
- Checks availability using Google Calendar
- Schedules events only if explicitly requested
- Prevents scheduling conflicts
- Uses Europe/Brussels timezone
- Blocks unauthorized users via allowlist

## Workflow Overview

![workflow](/resources/workflow.png)

![telegram1-1](/resources/telegram1-1.jpeg) ![telegram1-2](/resources/telegram1-2.jpeg)

![calendar](/resources/calendar.png)
---

### 1. Telegram Trigger
- Listens for incoming messages
- Extracts:
  - First name, last name
  - Message text
  - Unix timestamp (converted to ISO format in Brussels time)

### 2. Allowlist IF Node
- Ensures only approved Telegram users can interact with the assistant

### 3. Google Gemini AI Agent
- Receives user input and determines intent
- Calls tools (`get_calendar_availability`, `create_event`) before replying
- Never replies before tool calls are made
- Responds only with:
  ```json
  { "reply": "..." }
  ```

### 4. Structured Output Parser
- Validates the AI's JSON output before processing
- Fails safely if structure is invalid

### 5. Tool Execution

#### get_calendar_availability
- Calls Google Calendar Free/Busy API
- Returns busy blocks for the specified range
- A script checks for open time gaps large enough to fit the requested slot

#### create_event
- Only used if:
  - The user explicitly requests an event
  - There are no scheduling conflicts
- Event is created with correct timezone and ISO format

### 6. Telegram Response
- Final `reply` string is sent back to the user
- Always short, functional, and context-aware

## Scopes Required

In your Google Calendar OAuth2 credential, make sure the following scopes are set:

```text
https://www.googleapis.com/auth/calendar.events
```

Or for full calendar access:

```text
https://www.googleapis.com/auth/calendar
```

## Timezone Handling

- All timestamps are explicitly converted to `Europe/Brussels`
- ISO strings are generated using:

```javascript
new Date(unix * 1000).toLocaleString('sv-SE', { timeZone: 'Europe/Brussels' }).replace(' ', 'T') + ':00'
```

## AI Agent Configuration

- Responds with only structured JSON
- No filler language or greetings
- Only creates events if explicitly told to
- Always calls tools before replying

## Output Format

The AI always returns:

```json
{ "reply": "your response here" }
```

No markdown, no code blocks, no extra fields.

## Example Interactions

**User:** "What’s my availability Thursday?"  
**Reply:** "You are available between 1PM and 3PM."

**User:** "Schedule lunch at 1PM tomorrow."  
**Reply:** "I have scheduled you for lunch at 1PM tomorrow."

**User:** "Schedule a meeting at 9AM tomorrow."  
**Reply:** "You already have an event at that time."

## License

MIT

