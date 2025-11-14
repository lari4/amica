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

## Vision Processing Pipeline

**Purpose:** Processes images from the user's webcam or uploaded photos, generates descriptions, and enables natural conversation about what Amica "sees".

**Trigger:** User clicks camera button or uploads an image

**Location:** `src/features/chat/chat.ts:543-588`

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                      IMAGE CAPTURE                                  │
│  User action:                                                        │
│  • Webcam snapshot, OR                                              │
│  • File upload                                                       │
│  Result: Base64 encoded image data                                  │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  VISION BACKEND SELECTION                           │
│  Checks config:                                                      │
│  • vision_backend = "openai" → GPT-4 Vision                         │
│  • vision_backend = "ollama" → Ollama Vision                        │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  VISION PROMPT CONSTRUCTION                         │
│  Messages array for vision model:                                   │
│  [                                                                   │
│    {                                                                 │
│      role: "system",                                                │
│      content: config("vision_system_prompt")                        │
│      // "You are a friendly human named Amica.                      │
│      //  Describe the image in detail..."                           │
│    },                                                                │
│    {                                                                 │
│      role: "user",                                                  │
│      content: [                                                      │
│        { type: "image", image_url: "data:image/jpeg;base64,..." }  │
│      ]                                                               │
│    }                                                                 │
│  ]                                                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   VISION MODEL API CALL                             │
│  OpenAI: getOpenAIVisionChatResponse()                              │
│  OR                                                                  │
│  Ollama: getOllamaVisionChatResponse()                              │
│                                                                      │
│  Sends: vision_system_prompt + image data                           │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   VISION MODEL RESPONSE                             │
│  Detailed image description:                                        │
│  "A smiling person in a blue shirt sitting at a desk with a        │
│   laptop. There's a coffee cup on the left side and a plant in     │
│   the background. The lighting is warm and natural, coming from    │
│   a window."                                                         │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              VISION CONTEXT PROMPT WRAPPING                         │
│  Wraps description in context prompt:                               │
│  "This is a picture I just took from my webcam (described          │
│   between [[ and ]] ): [[A smiling person in a blue shirt...]]    │
│   Please respond accordingly and as if it were just sent and       │
│   as though you can see it."                                        │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│            INJECT INTO STANDARD CHAT PIPELINE                       │
│  Creates new message array:                                         │
│  [                                                                   │
│    { role: "system", content: [Main System Prompt] },              │
│    ...previousMessages,                                             │
│    {                                                                 │
│      role: "user",                                                  │
│      content: "This is a picture I just took from my webcam..."    │
│    }                                                                 │
│  ]                                                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  STANDARD CHAT PROCESSING                           │
│  Calls: makeAndHandleStream(messages)                               │
│  Uses: Main System Prompt (NOT vision prompt)                       │
│  Result: Amica responds in character to the image                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      AMICA'S RESPONSE                               │
│  "[happy] Oh, you look great in that blue shirt! *waves*           │
│   I see you're working hard at your desk. That's a nice plant      │
│   you have there!"                                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              STANDARD OUTPUT PROCESSING                             │
│  • Extract emotions                                                  │
│  • Generate TTS                                                      │
│  • Update avatar expression                                         │
│  • Display in chat                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Two-Stage LLM Processing

**Stage 1: Vision Analysis (Vision Model)**
- **Input:** Image data + Vision System Prompt
- **Model:** GPT-4 Vision or Ollama Vision
- **Purpose:** Describe the image objectively
- **Output:** Detailed image description (text only)

**Stage 2: Character Response (Chat Model)**
- **Input:** Wrapped description + Main System Prompt
- **Model:** Regular chat model (GPT-4, Claude, Ollama, etc.)
- **Purpose:** Respond to image in character with personality
- **Output:** Emotion-tagged dialogue

### Data Transformations

**Stage 1 Input (Vision Model):**
```typescript
messages: [
  {
    role: "system",
    content: "You are a friendly human named Amica. Describe the image in detail. Let's start the conversation."
  },
  {
    role: "user",
    content: [
      { type: "image", image_url: "data:image/jpeg;base64,/9j/4AAQSkZJRg..." }
    ]
  }
]
```

**Stage 1 Output:**
```typescript
visionDescription: string = "A smiling person in a blue shirt sitting at a desk with a laptop. There's a coffee cup on the left side and a plant in the background. The lighting is warm and natural, coming from a window."
```

**Stage 2 Input (Chat Model):**
```typescript
messages: [
  {
    role: "system",
    content: "You are Amica, a feisty human with extraordinary intellectual capabilities..."
  },
  {
    role: "user",
    content: "This is a picture I just took from my webcam (described between [[ and ]] ): [[A smiling person in a blue shirt sitting at a desk with a laptop. There's a coffee cup on the left side and a plant in the background. The lighting is warm and natural, coming from a window.]] Please respond accordingly and as if it were just sent and as though you can see it."
  }
]
```

**Stage 2 Output:**
```typescript
response: string = "[happy] Oh, you look great in that blue shirt! *waves* I see you're working hard at your desk. That's a nice plant you have there!"
```

### Key Functions

- `chat.getVisionResponse(imageData)` - Entry point (src/features/chat/chat.ts:543)
- `getOpenAIVisionChatResponse(messages, imageData)` - OpenAI vision (src/features/chat/openAiChat.ts)
- `getOllamaVisionChatResponse(messages, imageData)` - Ollama vision (src/features/chat/ollamaChat.ts)
- `chat.makeAndHandleStream(messages)` - Stage 2 processing (src/features/chat/chat.ts:336)

### Configuration Dependencies

- `vision_backend` - "openai" or "ollama"
- `vision_system_prompt` - Vision description instructions
- `system_prompt` - Main character personality (used in Stage 2)
- `openai_api_key` or Ollama configuration

### Design Rationale

**Why Two Stages?**
1. **Separation of Concerns:** Vision models excel at image description, chat models excel at personality
2. **Backend Flexibility:** Vision backend can differ from chat backend
3. **Prompt Isolation:** Vision prompt focuses on objective description, chat prompt adds personality
4. **Context Preservation:** Image description becomes part of conversation history

**Why Wrap Description in `[[ ]]`?**
- Clearly delimits the vision model's output from the instruction
- Helps chat model understand it's reading a description, not the user's direct message
- Prevents confusion between meta-information and actual user input

---

## Idle Event Pipeline

**Purpose:** Enables autonomous behavior by having Amica initiate conversations when the user has been inactive, creating a more lifelike and engaging experience.

**Trigger:** User inactivity for configured idle timeout period (when Amica Life is enabled)

**Location:** `src/features/amicaLife/eventHandler.ts:89-107`

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                      USER INACTIVITY                                │
│  No messages sent for X seconds (configurable)                      │
│  Idle timer expires                                                  │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  AMICA LIFE ENABLED CHECK                           │
│  if (config.amica_life !== true) → Exit                            │
│  if (eventProcessing === true) → Exit (already processing)         │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                 RANDOM IDLE PROMPT SELECTION                        │
│  idleTextPrompts = [                                                │
│    "*I am ignoring you*",                                           │
│    "**sighs** It's so quiet here.",                                 │
│    "Tell me something interesting about yourself.",                 │
│    "**looks around** What do you usually do for fun?",             │
│    "I could use a good distraction right now.",                     │
│    "What's the most fascinating thing you know?",                   │
│    "If you could talk about anything, what would it be?",          │
│    "Got any clever insights to share?",                             │
│    "**leans in** Any fun stories to tell?"                         │
│  ]                                                                   │
│                                                                      │
│  selectedPrompt = random.choice(idleTextPrompts)                    │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              MESSAGE CONSTRUCTION (AS USER)                         │
│  Creates message array simulating user input:                       │
│  [                                                                   │
│    { role: "system", content: [Main System Prompt] },              │
│    ...existingConversation,                                         │
│    {                                                                 │
│      role: "user",                                                  │
│      content: selectedPrompt                                        │
│      // e.g., "Tell me something interesting about yourself."      │
│    }                                                                 │
│  ]                                                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  STANDARD CHAT PROCESSING                           │
│  Calls: chat.processUserMessage(selectedPrompt)                     │
│  Treats idle prompt as if user typed it                             │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                      LLM RESPONSE                                   │
│  Amica responds to the self-generated prompt:                       │
│  "[relaxed] Well, I find quantum mechanics absolutely              │
│   fascinating! *adjusts glasses* The idea that particles can       │
│   exist in superposition until observed is mind-bending."          │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  STANDARD OUTPUT PROCESSING                         │
│  • Parse emotions and actions                                       │
│  • Generate TTS                                                      │
│  • Update avatar                                                     │
│  • Display in chat                                                   │
│  • Add to conversation history                                      │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    RESET IDLE TIMER                                 │
│  Timer resets, waiting for next idle period                         │
│  eventProcessing = false                                             │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Transformations

**Input (Random Selection):**
```typescript
selectedPrompt: string = "Tell me something interesting about yourself."
// Randomly chosen from idleTextPrompts array
```

**Message Array Construction:**
```typescript
messages: Message[] = [
  { role: "system", content: "You are Amica, a feisty human..." },
  { role: "user", content: "How are you?" },
  { role: "assistant", content: "[happy] I'm doing great! How about you?" },
  { role: "user", content: "Tell me something interesting about yourself." }
  // ^ This is the idle prompt, injected as if user sent it
]
```

**LLM Response:**
```typescript
response: string = "[relaxed] Well, I find quantum mechanics absolutely fascinating! *adjusts glasses* The idea that particles can exist in superposition until observed is mind-bending."
```

**Conversation History Update:**
```typescript
// Both the idle prompt AND response are added to history
messageList.push({
  role: "user",
  content: "Tell me something interesting about yourself."
});

messageList.push({
  role: "assistant",
  content: "[relaxed] Well, I find quantum mechanics..."
});
```

### Key Functions

- `handleIdleEvent(chat, amicaLife)` - Entry point (src/features/amicaLife/eventHandler.ts:89)
- `chat.processUserMessage(prompt)` - Standard chat processing (src/features/chat/chat.ts:258)
- Idle timer logic in `src/features/amicaLife/AmicaLife.ts`

### Configuration Dependencies

- `amica_life` - Must be enabled (boolean)
- `idle_timeout` - Seconds of inactivity before triggering (number)
- `system_prompt` - Used for response generation
- `chat_backend` - Which LLM to use

### Behavioral Characteristics

**Randomization:**
- Each idle event randomly selects from 9 different prompts
- Creates variety in self-initiated interactions
- Prevents repetitive behavior

**Conversation Integration:**
- Idle prompts are treated as actual user messages
- They become part of conversation history
- Responses reference previous context if relevant

**Timing Control:**
- Configurable idle timeout period
- Prevents spam by checking `eventProcessing` flag
- Resets timer after each interaction

**Event Types:**
Currently only "Idle" event type is implemented in idle handler. Other event types (Subconscious, News) use separate handlers.

### Design Pattern: Self-Prompting

This pipeline uses a "self-prompting" pattern where:
1. System generates a prompt internally
2. Injects it as a user message
3. Processes through standard chat pipeline
4. Appears to user as if Amica spontaneously spoke

**Benefits:**
- Reuses existing chat infrastructure
- Maintains conversation coherence
- Enables contextual self-initiated dialogue
- Simulates autonomous thought

**Example Interaction Flow:**

```
User: "How are you?"
Amica: "[happy] I'm doing great! Thanks for asking."

[30 seconds of inactivity]

Amica: "[bored] **sighs** It's so quiet here."
       (Self-initiated from idle prompt)

User: "Sorry, I was reading."
Amica: "[relaxed] No worries! What are you reading?"
```

---

