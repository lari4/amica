# Amica Agent Pipelines Documentation

This document provides a comprehensive overview of all agent workflows and data pipelines in the Amica application. Each pipeline shows how data flows through the system, which prompts are used, and what transformations occur.

## Table of Contents

1. [Standard Chat Pipeline](#standard-chat-pipeline)
2. [Vision Processing Pipeline](#vision-processing-pipeline)
3. [Idle Event Pipeline](#idle-event-pipeline)
4. [Subconscious Subroutine Pipeline](#subconscious-subroutine-pipeline)
5. [News Commentary Pipeline](#news-commentary-pipeline)
6. [Function Calling Pipeline](#function-calling-pipeline)

---

## Standard Chat Pipeline

**Purpose:** Handles regular user text input and generates appropriate AI responses with emotion tags and actions.

**Trigger:** User types a message and sends it

**Location:** `src/features/chat/chat.ts`

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                         USER INPUT                                  │
│                      "Tell me about AI"                             │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    MESSAGE CONSTRUCTION                             │
│  Creates Message Array:                                             │
│  [                                                                   │
│    { role: "system", content: [Main System Prompt] },              │
│    { role: "user", content: "Tell me about AI" }                   │
│  ]                                                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    BACKEND SELECTION                                │
│  Determines which chat backend to use:                              │
│  • OpenAI                                                            │
│  • OpenRouter                                                        │
│  • Ollama                                                            │
│  • LLaMA.cpp (requires buildPrompt())                               │
│  • KoboldAI (requires buildPrompt())                                │
│  • Window AI                                                         │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              PROMPT FORMATTING (if needed)                          │
│  For LLaMA.cpp/KoboldAI:                                            │
│  buildPrompt(messages) →                                             │
│  "You are Amica, a feisty human...\n\n                             │
│   User: Tell me about AI\n                                          │
│   Amica:"                                                            │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      LLM API CALL                                   │
│  Sends formatted prompt/messages to chosen backend                  │
│  Streaming enabled (tokens arrive in real-time)                     │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    LLM RESPONSE (Streaming)                         │
│  "[happy] AI is fascinating! *leans forward* It encompasses         │
│   machine learning, natural language processing..."                 │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   RESPONSE PROCESSING                               │
│  processResponse() extracts:                                        │
│  • Emotion tags: [happy]                                            │
│  • Roleplay actions: *leans forward*                                │
│  • Clean dialogue text                                              │
│  • Sentence boundaries for TTS                                      │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    SCREENPLAY CREATION                              │
│  Converts to Screenplay format:                                     │
│  [                                                                   │
│    {                                                                 │
│      expression: "happy",                                           │
│      talk: {                                                         │
│        message: "AI is fascinating!",                               │
│        speakerId: "amica"                                           │
│      }                                                               │
│    },                                                                │
│    { expression: "happy", talk: { ... } },                         │
│    ...                                                               │
│  ]                                                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    PARALLEL EXECUTION                               │
│  ┌─────────────────────┐    ┌──────────────────────┐               │
│  │   TTS GENERATION    │    │  EMOTION EXPRESSION  │               │
│  │  Text → Audio       │    │  Update 3D Avatar    │               │
│  │  Per sentence       │    │  Facial animation    │               │
│  └─────────────────────┘    └──────────────────────┘               │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      USER SEES/HEARS                                │
│  • 3D avatar shows happy expression                                 │
│  • Lips sync with generated speech                                  │
│  • Text displays in chat interface                                  │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Transformations

**Input:**
```typescript
userMessage: string = "Tell me about AI"
```

**After Message Construction:**
```typescript
messages: Message[] = [
  { role: "system", content: "You are Amica, a feisty human..." },
  { role: "user", content: "Tell me about AI" }
]
```

**After LLM Response:**
```typescript
rawResponse: string = "[happy] AI is fascinating! *leans forward* It encompasses machine learning..."
```

**After Processing:**
```typescript
screenplay: Screenplay[] = [
  {
    expression: "happy",
    talk: {
      message: "AI is fascinating!",
      speakerId: "amica"
    }
  },
  {
    expression: "happy",
    talk: {
      message: "It encompasses machine learning, natural language processing, and more.",
      speakerId: "amica"
    }
  }
]
```

### Key Functions

- `chat.processUserMessage(text)` - Entry point (src/features/chat/chat.ts:258)
- `chat.makeAndHandleStream(messages)` - Handles LLM call and streaming (src/features/chat/chat.ts:336)
- `processResponse(text)` - Parses emotion tags and creates screenplay (src/utils/processResponse.ts)
- Backend-specific chat functions:
  - `getOpenAIChatResponse()` - src/features/chat/openAiChat.ts
  - `getOllamaChatResponse()` - src/features/chat/ollamaChat.ts
  - `getLlamaCppChatResponse()` - src/features/chat/llamaCppChat.ts
  - etc.

### Configuration Dependencies

- `system_prompt` - Character personality
- `chat_backend` - Which LLM service to use
- `name` - Character name in prompts
- Model parameters (temperature, max tokens, etc.)

---

