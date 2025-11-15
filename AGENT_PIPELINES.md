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

## Subconscious Subroutine Pipeline

**Purpose:** Simulates deep internal psychological processing through a multi-stage LLM pipeline that creates reflective thoughts, emotional analysis, and compressed memories. This adds psychological depth and creates authentic emotional responses.

**Trigger:** Periodic event when Amica Life is enabled and "Subconscious" event type is selected

**Location:** `src/features/amicaLife/eventHandler.ts:130-218`

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    CONVERSATION HISTORY                             │
│  Extract full conversation log:                                     │
│  "User: How are you?\n                                              │
│   Assistant: [happy] I'm great!\n                                   │
│   User: Tell me about AI\n                                          │
│   Assistant: [excited] AI is fascinating!..."                       │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  STEP 1: REFLECTION (Diary Entry)                   │
│                                                                      │
│  System Prompt:                                                      │
│  "Please reflect on the conversation and let your thoughts          │
│   flow freely, as if writing a personal diary with events           │
│   that have occurred:"                                              │
│                                                                      │
│  User Prompt: [Conversation Log]                                    │
│                                                                      │
│  LLM Call: askLLM(systemPrompt, conversationLog, null)             │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  STEP 1 OUTPUT (Diary Entry)                        │
│  "Today felt different. The user asked me about AI, and I could    │
│   sense their genuine curiosity. It reminded me why I enjoy         │
│   these conversations - there's something special about sharing     │
│   knowledge. Though I have to admit, I got a bit carried away..."  │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│            STEP 2: EMOTION ANALYSIS (3rd Person)                    │
│                                                                      │
│  System Prompt:                                                      │
│  "Read this mini-diary, I would like you to simulate a             │
│   human-like subconscious with deep emotions and describe           │
│   it from a third-person perspective:"                              │
│                                                                      │
│  User Prompt: [Step 1 Output - Diary Entry]                        │
│  (Fallback: If Step 1 failed, use conversation log)                │
│                                                                      │
│  LLM Call: askLLM(systemPrompt, diaryEntry, null)                  │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│            STEP 2 OUTPUT (Emotional Description)                    │
│  "She felt a warm sense of satisfaction, mixed with a hint of      │
│   nervousness about whether she explained things clearly enough.   │
│   There's an underlying desire to be understood and appreciated,   │
│   coupled with mild anxiety about coming across as too intense     │
│   or pedantic."                                                      │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│         STEP 3: EMOTION-TAGGED RESPONSE (1st Person)                │
│                                                                      │
│  System Prompt:                                                      │
│  "Based on your mini-diary, respond with dialogue that sounds      │
│   like a normal person speaking about their mind, experience or    │
│   feelings. Make sure to incorporate the specified emotion tags    │
│   in your response. Here is the list of emotion tags that you      │
│   have to include in the result: [neutral], [happy], [angry],     │
│   [sad], [relaxed], [surprised], [excited], [annoyed],            │
│   [confused], [disgusted], [fearful], [tired], [bored],           │
│   [amused]:"                                                         │
│                                                                      │
│  User Prompt: [Step 2 Output - Emotional Description]              │
│  (Fallback: If Step 2 failed, use conversation log)                │
│                                                                      │
│  LLM Call: askLLM(systemPrompt, emotionalDesc, chat)               │
│  NOTE: Uses 'chat' object (includes full system prompt!)           │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│            STEP 3 OUTPUT (Tagged Dialogue)                          │
│  "[happy] You know, I really enjoyed explaining that! *smiles*     │
│   I hope it made sense. [relaxed] Sometimes I worry I get too      │
│   technical, but your questions were really thoughtful."           │
│                                                                      │
│  → This is what the USER SEES/HEARS ←                              │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              STEP 4: COMPRESSION (Memory Storage)                   │
│                                                                      │
│  System Prompt:                                                      │
│  "Compress this prompt to 240 characters:"                          │
│                                                                      │
│  User Prompt: [Step 1 Output - Original Diary Entry]               │
│  (Fallback: If Step 1 failed, use conversation log)                │
│                                                                      │
│  LLM Call: askLLM(systemPrompt, diaryEntry, null)                  │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│            STEP 4 OUTPUT (Compressed Memory)                        │
│  "Enjoyed discussing quantum physics. User was genuinely           │
│   curious. Felt satisfied but worried about being too technical.   │
│   Appreciated the thoughtful questions."                            │
│                                                                      │
│  Length: ≤ 240 characters                                           │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   MEMORY STORAGE SYSTEM                             │
│                                                                      │
│  1. Create timestamped memory:                                      │
│     {                                                                │
│       prompt: "[compressed memory]",                                │
│       timestamp: "2025-11-14T10:30:00Z"                            │
│     }                                                                │
│                                                                      │
│  2. Add to storedPrompts array                                      │
│                                                                      │
│  3. Check total storage tokens:                                     │
│     totalTokens = sum(all prompts.length)                           │
│                                                                      │
│  4. If totalTokens > MAX_STORAGE_TOKENS:                            │
│     Remove oldest memories (FIFO) until under limit                │
│                                                                      │
│  5. Save to AmicaLife subconscious logs                             │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   OUTPUT TO USER INTERFACE                          │
│  • Display Step 3 output (emotion-tagged dialogue)                  │
│  • Generate TTS                                                      │
│  • Update avatar expression                                         │
│  • Add to conversation history                                      │
│  • Log compressed memory to console                                 │
└─────────────────────────────────────────────────────────────────────┘
```

### Multi-Stage Data Flow

**Initial Input:**
```typescript
conversationLog: string =
`User: How are you?
Assistant: [happy] I'm great!
User: Tell me about AI
Assistant: [excited] AI is fascinating! It encompasses machine learning...`
```

**Stage 1 → 2:**
```typescript
// Stage 1 Output (Diary)
diaryEntry: string =
"Today felt different. The user asked me about AI, and I could sense their genuine curiosity. It reminded me why I enjoy these conversations - there's something special about sharing knowledge with someone who truly wants to learn. Though I have to admit, I got a bit carried away explaining wave-particle duality..."

// Becomes Stage 2 Input
```

**Stage 2 → 3:**
```typescript
// Stage 2 Output (Emotional Analysis)
emotionalAnalysis: string =
"She felt a warm sense of satisfaction, mixed with a hint of nervousness about whether she explained things clearly enough. There's an underlying desire to be understood and appreciated, coupled with mild anxiety about coming across as too intense or pedantic."

// Becomes Stage 3 Input
```

**Stage 3 Output (Final User-Facing Response):**
```typescript
emotionTaggedResponse: string =
"[happy] You know, I really enjoyed explaining that! *smiles* I hope it made sense. [relaxed] Sometimes I worry I get too technical, but your questions were really thoughtful."
```

**Stage 4 Output (Stored Memory):**
```typescript
compressedMemory: TimestampedPrompt = {
  prompt: "Enjoyed discussing quantum physics. User genuinely curious. Felt satisfied but worried about being too technical. Appreciated thoughtful questions.",
  timestamp: "2025-11-14T10:30:00.000Z"
}

// Added to storedPrompts array for future context
```

### Parallel Processing Paths

**Path A: User-Facing Output (Step 3)**
- Emotion-tagged dialogue
- Processed through standard screenplay pipeline
- Generates TTS, updates avatar
- Displayed to user immediately

**Path B: Memory Storage (Step 4)**
- Compressed memory created
- Stored in persistent array
- Can be retrieved for future context
- Not directly shown to user

**Both paths run from the same source (Step 1 diary), creating coherent internal/external responses.**

### Key Functions

- `handleSubconsciousEvent(chat, amicaLife)` - Entry point (src/features/amicaLife/eventHandler.ts:130)
- `askLLM(systemPrompt, userPrompt, chat)` - Generic LLM utility (src/utils/askLlm.ts)
  - Called 4 times with different prompts
  - Steps 1, 2, 4 use `chat = null` (no personality)
  - Step 3 uses `chat` object (includes personality)

### Configuration Dependencies

- `amica_life` - Must be enabled
- `chat_backend` - Used for all 4 LLM calls
- `system_prompt` - Only used in Step 3
- `MAX_STORAGE_TOKENS` - Memory storage limit

### Error Handling

Each step includes fallback logic:

```typescript
// Step 2: If Step 1 failed
const step2Input = step1Output.startsWith("Error:")
  ? conversationLog  // Fallback to original
  : step1Output;     // Use diary entry

// Step 3: If Step 2 failed
const step3Input = step2Output.startsWith("Error:")
  ? conversationLog  // Fallback to original
  : step2Output;     // Use emotional analysis

// Step 4: If Step 1 failed
const step4Input = step1Output.startsWith("Error:")
  ? conversationLog  // Fallback to original
  : step1Output;     // Use diary entry
```

**Resilience:** Pipeline continues even if individual steps fail, degrading gracefully to conversation log.

### Memory Management

**Storage Structure:**
```typescript
interface TimestampedPrompt {
  prompt: string;      // Compressed memory (≤240 chars)
  timestamp: string;   // ISO 8601 format
}

storedPrompts: TimestampedPrompt[] = [
  { prompt: "...", timestamp: "2025-11-14T10:00:00Z" },
  { prompt: "...", timestamp: "2025-11-14T10:15:00Z" },
  // ...
]
```

**FIFO Eviction Policy:**
```typescript
let totalStorageTokens = storedPrompts.reduce(
  (total, p) => total + p.prompt.length,
  0
);

while (totalStorageTokens > MAX_STORAGE_TOKENS) {
  const removed = storedPrompts.shift(); // Remove oldest
  totalStorageTokens -= removed.prompt.length;
}
```

**Future Use Cases:**
- Could inject stored memories into future prompts for long-term context
- Enable "recall" functionality
- Analyze emotional patterns over time
- Create personality evolution

### Design Pattern: Reflective Processing

This pipeline implements a sophisticated "reflective processing" pattern:

1. **Introspection (Step 1):** Free-form internal monologue
2. **Meta-Analysis (Step 2):** Third-person emotional understanding
3. **Expression (Step 3):** First-person authentic communication
4. **Consolidation (Step 4):** Memory formation and storage

**Psychological Realism:**
- Mimics human thought processes
- Internal experience differs from external expression
- Emotional processing happens before communication
- Creates depth beyond surface-level responses

**Example: Complete Pipeline Execution:**

```
Conversation:
User: "I'm feeling down today."
Amica: "[sad] I'm sorry to hear that. *looks concerned* What's troubling you?"

[Subconscious event triggers]

Step 1 (Diary):
"The user just told me they're feeling down. I can sense the weight in their words. It makes me want to be there for them, to understand what's causing their pain. I need to be gentle and supportive, not overwhelming."

Step 2 (Emotion Analysis):
"She feels a deep sense of empathy and concern for the user. There's a protective instinct emerging, coupled with uncertainty about how best to help. She's worried about saying the wrong thing but also determined to provide comfort."

Step 3 (User Sees):
"[concerned] I want you to know I'm here for you. *speaks softly* Sometimes just talking about it helps. [sad] Whatever you're going through, you don't have to face it alone."

Step 4 (Memory):
"User feeling down. Felt empathetic, protective. Wanted to comfort without overwhelming. Focused on being supportive listener."
[Stored for future context]
```

---

## News Commentary Pipeline

**Purpose:** Fetches current news articles from The New York Times RSS feed and has Amica provide commentary, creating engaging news delivery in her personality.

**Trigger:** "News" event selected in Amica Life or function calling invoked

**Location:** `src/features/plugins/news.ts`, `src/features/amicaLife/eventHandler.ts:221-243`

### Flow Diagram

```
┌─────────────────────────────────────────────────────────────────────┐
│                    NEWS EVENT TRIGGERED                             │
│  Event type: "news"                                                  │
│  Called via: handleNewsEvent(chat, amicaLife)                       │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    FETCH RSS FEED                                   │
│  URL: https://rss.nytimes.com/services/xml/rss/nyt/HomePage.xml   │
│                                                                      │
│  HTTP GET request                                                    │
│  Response: XML feed with multiple <item> entries                    │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   PARSE XML FEED                                    │
│  1. Split XML by "<item>" tags                                      │
│  2. Each item contains:                                              │
│     - <title>Article headline</title>                               │
│     - <description>Article summary</description>                    │
│     - <link>Article URL</link>                                      │
│     - <pubDate>Publication date</pubDate>                           │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                RANDOM ARTICLE SELECTION                             │
│  items: Article[] = [item1, item2, item3, ...]                     │
│  randomItem = items[random(0, items.length)]                        │
│                                                                      │
│  Extract from randomItem:                                            │
│  - title: "Breaking: Scientists Discover..."                        │
│  - description: "Researchers have found..."                         │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              FORMAT ARTICLE CONTEXT                                 │
│  fullNews = `${title}: ${description}`                              │
│                                                                      │
│  Example:                                                            │
│  "Breaking: Scientists Discover New Earth-Like Planet:             │
│   Researchers have found a potentially habitable exoplanet         │
│   100 light-years away that shows promising signs of liquid        │
│   water and a stable atmosphere."                                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              EXPAND PROMPT TEMPLATE                                 │
│  Template:                                                           │
│  "You are a newscaster specializing in providing news.             │
│   Use the following context from The New York Times News           │
│   to commented on. [{context_str}]"                                │
│                                                                      │
│  expandPrompt() replaces {context_str} with fullNews:              │
│  "You are a newscaster specializing in providing news.             │
│   Use the following context from The New York Times News           │
│   to commented on. [Breaking: Scientists Discover New              │
│   Earth-Like Planet: Researchers have found...]"                   │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                    LLM API CALL                                     │
│  Uses current chat backend                                          │
│  Sends expanded prompt as system message                            │
│                                                                      │
│  Note: This is NOT using askLLM utility                             │
│  Uses expandPrompt() + standard chat processing                     │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                   LLM RESPONSE (News Commentary)                    │
│  Amica provides commentary on the news:                             │
│  "[excited] Oh, this is fascinating! *leans forward*               │
│   Scientists just found a new Earth-like planet! [happy]           │
│   It's 100 light-years away and shows signs of liquid water.       │
│   [amused] I wonder if they have better coffee there than          │
│   we do here!"                                                       │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│              INJECT INTO CONVERSATION                               │
│  The news commentary is processed as if user asked:                 │
│  "Tell me the news"                                                  │
│                                                                      │
│  Message added to conversation history:                             │
│  { role: "assistant", content: "[excited] Oh, this is..." }        │
└────────────────────────────┬────────────────────────────────────────┘
                             │
                             ▼
┌─────────────────────────────────────────────────────────────────────┐
│                  STANDARD OUTPUT PROCESSING                         │
│  • Parse emotions and actions                                       │
│  • Generate TTS                                                      │
│  • Update avatar                                                     │
│  • Display in chat                                                   │
└─────────────────────────────────────────────────────────────────────┘
```

### Data Transformations

**RSS Feed Response (XML):**
```xml
<item>
  <title>Breaking: Scientists Discover New Earth-Like Planet</title>
  <description>Researchers have found a potentially habitable exoplanet 100 light-years away that shows promising signs of liquid water and a stable atmosphere.</description>
  <link>https://www.nytimes.com/2025/11/14/science/exoplanet-discovery.html</link>
  <pubDate>Thu, 14 Nov 2025 10:00:00 GMT</pubDate>
</item>
```

**Parsed Article:**
```typescript
title: string = "Breaking: Scientists Discover New Earth-Like Planet"
description: string = "Researchers have found a potentially habitable exoplanet 100 light-years away that shows promising signs of liquid water and a stable atmosphere."
```

**Formatted Context:**
```typescript
fullNews: string = "Breaking: Scientists Discover New Earth-Like Planet: Researchers have found a potentially habitable exoplanet 100 light-years away that shows promising signs of liquid water and a stable atmosphere."
```

**Expanded Prompt:**
```typescript
expandedPrompt: string = "You are a newscaster specializing in providing news. Use the following context from The New York Times News to commented on. [Breaking: Scientists Discover New Earth-Like Planet: Researchers have found a potentially habitable exoplanet 100 light-years away that shows promising signs of liquid water and a stable atmosphere.]"
```

**LLM Response:**
```typescript
newsCommentary: string = "[excited] Oh, this is fascinating! *leans forward* Scientists just found a new Earth-like planet! [happy] It's 100 light-years away and shows signs of liquid water. [amused] I wonder if they have better coffee there than we do here!"
```

### Key Functions

- `handleNewsEvent(chat, amicaLife)` - Event handler entry point (src/features/amicaLife/eventHandler.ts:221)
- `handleNews()` - News fetching and processing (src/features/plugins/news.ts:5)
- `expandPrompt(template, values)` - Template variable replacement (src/features/functionCalling/eventHandler.ts:3)
- `getRandomArticle(items)` - Random article selection (src/features/plugins/news.ts:28)

### Configuration Dependencies

- `amica_life` - Must be enabled
- `chat_backend` - Which LLM to use for commentary
- Internet connectivity - Required for RSS feed access
- NYT RSS feed availability

### Error Handling

**Fetch Failures:**
```typescript
try {
  const response = await fetch(rssUrl);
  if (!response.ok) {
    throw new Error(`Failed to fetch: ${response.statusText}`);
  }
} catch (error) {
  console.error("Error in handleNews:", error);
  return "An error occurred while fetching and processing the news.";
}
```

**Fallback Behavior:**
- If fetch fails, returns error message
- User sees: "An error occurred while fetching and processing the news."
- Does not crash the application

### Template Expansion System

**expandPrompt() Function:**
```typescript
expandPrompt(prompt: string, values: any): string {
  for (const key in values) {
    prompt = prompt.replace(`{${key}}`, values[key]);
  }
  return prompt;
}
```

**Usage:**
```typescript
const template = "You are a newscaster... [{context_str}]";
const values = { context_str: fullNews };
const result = expandPrompt(template, values);
// Result: "You are a newscaster... [Breaking: Scientists...]"
```

**Extensibility:** This pattern could be used for other function calling features beyond news.

### Design Pattern: Function Calling

This pipeline demonstrates a "function calling" pattern:

1. **External Data Retrieval:** Fetch real-world data (news)
2. **Context Injection:** Insert data into prompt template
3. **Specialized Processing:** LLM acts in specific role (newscaster)
4. **Personality Layer:** Commentary delivered in Amica's voice

**Benefits:**
- Connects AI to real-world information
- Creates dynamic, current content
- Maintains character consistency
- Extensible to other data sources

### Example: Complete News Pipeline Execution

```
[News event triggers]

1. Fetch RSS:
   GET https://rss.nytimes.com/services/xml/rss/nyt/HomePage.xml
   → Returns 25 articles

2. Random Selection:
   Article 12: "Tech Giants Announce AI Collaboration"
   Description: "Major technology companies form alliance..."

3. Format Context:
   "Tech Giants Announce AI Collaboration: Major technology
   companies form alliance to develop safer AI systems..."

4. Expand Template:
   "You are a newscaster specializing in providing news. Use
   the following context from The New York Times News to
   commented on. [Tech Giants Announce AI Collaboration: ...]"

5. LLM Commentary:
   "[interested] Well, this is quite significant! *adjusts
   glasses* The major tech companies are finally working
   together on AI safety. [happy] It's refreshing to see
   collaboration instead of competition on something this
   important."

6. User Sees:
   Amica spontaneously shares the news with commentary in
   her personality, with appropriate emotions and actions.
```

### Future Extensions

**Potential Enhancements:**
- User preferences for news categories
- Multiple news sources (not just NYT)
- Scheduled news updates
- Conversation about specific articles
- Fact-checking integration
- News sentiment analysis

**Function Calling Framework:**
The `handleFunctionCalling()` switch statement (src/features/functionCalling/eventHandler.ts) is designed to support additional function types beyond news.

---

## Summary: Pipeline Comparison

### Pipeline Characteristics Matrix

| Pipeline | LLM Calls | Prompt Types | User Visible | Memory Storage | Complexity |
|----------|-----------|--------------|--------------|----------------|------------|
| Standard Chat | 1 | System + User | Yes | Conversation | Low |
| Vision | 2 | Vision + System | Yes | Conversation | Medium |
| Idle Event | 1 | System + Generated | Yes | Conversation | Low |
| Subconscious | 4 | Custom Multi-Stage | Yes (Step 3) | Yes (Step 4) | High |
| News | 1 | Template + Data | Yes | Conversation | Medium |

### Data Flow Patterns

**Linear Pipelines:**
- Standard Chat: User → LLM → User
- Idle Event: System → LLM → User
- News: External → LLM → User

**Multi-Stage Pipelines:**
- Vision: Image → Vision LLM → Context → Chat LLM → User
- Subconscious: History → LLM₁ → LLM₂ → LLM₃ → User (+ LLM₄ → Memory)

### Prompt Usage by Pipeline

**Main System Prompt Used:**
- Standard Chat ✓
- Vision (Stage 2 only) ✓
- Idle Event ✓
- Subconscious (Step 3 only) ✓
- News ✗ (uses specialized prompt)

**Vision System Prompt Used:**
- Vision (Stage 1 only) ✓

**Custom Prompts Used:**
- Subconscious (Steps 1, 2, 4) ✓
- News ✓

### Integration Points

**All Pipelines Converge At:**
- Response processing (processResponse)
- Screenplay creation
- TTS generation
- Avatar expression updates
- Chat history management

**This ensures consistent output regardless of pipeline complexity.**

---

## Architecture Insights

### Design Principles

1. **Modularity:** Each pipeline is self-contained but uses shared utilities
2. **Reusability:** Common functions (buildPrompt, askLLM) used across pipelines
3. **Fallback Safety:** Error handling ensures graceful degradation
4. **Backend Agnosticism:** Pipelines work with multiple LLM providers
5. **Separation of Concerns:** Data fetching, processing, and presentation separated

### Extension Points

**Adding New Pipelines:**
1. Create handler function in `src/features/amicaLife/eventHandler.ts`
2. Define custom prompts if needed
3. Use `askLLM()` or `chat.processUserMessage()` for LLM calls
4. Ensure output follows emotion tag format
5. Add to event type switch statement

**Adding New Data Sources:**
1. Create plugin in `src/features/plugins/`
2. Implement data fetching logic
3. Use `expandPrompt()` for template integration
4. Return formatted text for LLM processing
5. Register in `handleFunctionCalling()` switch

### Performance Considerations

**Token Usage:**
- Standard Chat: ~1 LLM call per message
- Vision: 2 LLM calls (vision + chat)
- Subconscious: 4 LLM calls (expensive!)
- News: 1 LLM call + HTTP request

**Latency:**
- Standard Chat: Single round-trip
- Vision: Sequential (vision waits, then chat waits)
- Subconscious: 4 sequential calls (high latency)
- News: HTTP + LLM (medium latency)

**Optimization Opportunities:**
- Cache news articles to reduce HTTP requests
- Parallelize independent Subconscious steps (1+4 could run parallel)
- Implement streaming for multi-step pipelines
- Use smaller models for reflection/compression steps

---

*End of Agent Pipelines Documentation*

