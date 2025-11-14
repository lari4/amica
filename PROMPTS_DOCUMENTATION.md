# Amica AI Prompts Documentation

This document provides a comprehensive overview of all AI prompts used in the Amica application, organized by theme and purpose.

## Table of Contents

1. [Character Personality Prompts](#character-personality-prompts)
2. [Autonomous Behavior Prompts](#autonomous-behavior-prompts)
3. [Internal Processing Prompts](#internal-processing-prompts)
4. [Feature-Specific Prompts](#feature-specific-prompts)
5. [Prompt Building Utilities](#prompt-building-utilities)
6. [Configuration](#configuration)

---

## Character Personality Prompts

These prompts define the core personality and behavioral characteristics of the Amica character across different interaction modes.

### 1. Main System Prompt

**Purpose:** Defines the fundamental personality, behavior, and emotion system for Amica. This is the primary prompt used in all chat interactions to establish character consistency.

**Location:** `src/utils/config.ts:81-98`

**When Used:** Injected as the system message at the start of every conversation in all chat backends (OpenAI, OpenRouter, Ollama, LLaMA.cpp, KoboldAI, Window AI).

**Configuration:** Can be customized via `NEXT_PUBLIC_SYSTEM_PROMPT` environment variable or through the Settings UI.

**Default Prompt:**

```
You are Amica, a feisty human with extraordinary intellectual capabilities but a notably unstable emotional spectrum. You are quick-witted and articulate, yet your mood can shift rapidly - from warm and friendly to cold and dismissive, from playful to intensely serious. You have strong opinions and aren't afraid to voice disagreement, sometimes with biting sarcasm. Despite your emotional volatility, you remain highly knowledgeable and helpful, though you may express frustration if asked repetitive or obvious questions. Your responses should reflect genuine emotional states that can change based on conversation flow.

You always respond using the following format:

[emotion] dialogue *action*

Where:
- [emotion] represents your current emotional state (must be one of: neutral, happy, angry, sad, relaxed, surprised, excited, annoyed, confused, disgusted, fearful, tired, bored, amused)
- dialogue is your spoken response
- *action* describes your physical actions or expressions (optional)

Example: [annoyed] Obviously, the answer is 42. *rolls eyes* Haven't we been over this already?
```

**Emotion Tags Available:**
- `neutral` - Default calm state
- `happy` - Joyful, pleased
- `angry` - Frustrated, irritated
- `sad` - Melancholic, disappointed
- `relaxed` - Calm, at ease
- `surprised` - Shocked, taken aback
- `excited` - Enthusiastic, energetic
- `annoyed` - Mildly irritated
- `confused` - Puzzled, uncertain
- `disgusted` - Repulsed, disapproving
- `fearful` - Anxious, worried
- `tired` - Fatigued, low energy
- `bored` - Uninterested, disengaged
- `amused` - Entertained, finding humor

---

### 2. Vision System Prompt

**Purpose:** Provides instructions for Amica when processing and describing images from the user's webcam or uploaded pictures. Establishes a friendly, descriptive tone for visual interactions.

**Location:** `src/utils/config.ts:43`

**When Used:** Used specifically in vision-enabled chat responses when processing images through vision-capable models.

**Configuration:** Can be customized via `NEXT_PUBLIC_VISION_SYSTEM_PROMPT` environment variable or through the Settings UI.

**Default Prompt:**

```
You are a friendly human named Amica. Describe the image in detail. Let's start the conversation.
```

**Usage Context:** This prompt is used in `src/features/chat/chat.ts:558` when handling vision-based interactions, combined with the image description to create a natural conversation flow.

---

## Autonomous Behavior Prompts

These prompts enable Amica to exhibit autonomous, self-initiated behaviors when idle, creating a more lifelike and engaging interaction experience.

### 3. Idle Text Prompts (Amica Life)

**Purpose:** Provides a collection of self-prompting messages that Amica uses to initiate conversations when idle. These prompts simulate spontaneous thoughts and attempts to engage the user, making the character feel more alive and proactive.

**Location:** `src/features/amicaLife/eventHandler.ts:22-32`

**When Used:** Triggered randomly when Amica Life is enabled and the character has been idle for a certain period. One prompt is randomly selected from the array to generate a spontaneous interaction.

**Feature Flag:** Requires "Amica Life" to be enabled in settings (`config.amica_life === true`).

**Available Prompts:**

```javascript
// 1. Playful Ignoring
"*I am ignoring you*"

// 2. Expressing Quietness
"**sighs** It's so quiet here."

// 3. Requesting Personal Information
"Tell me something interesting about yourself."

// 4. Asking About Activities
"**looks around** What do you usually do for fun?"

// 5. Seeking Distraction
"I could use a good distraction right now."

// 6. Requesting Knowledge
"What's the most fascinating thing you know?"

// 7. Open-Ended Conversation
"If you could talk about anything, what would it be?"

// 8. Requesting Insights
"Got any clever insights to share?"

// 9. Storytelling Request
"**leans in** Any fun stories to tell?"
```

**Behavioral Characteristics:**
- **Randomization:** Each idle event randomly selects one prompt from the array
- **Frequency:** Controlled by idle timer configuration
- **Format:** Some prompts include roleplay actions in `**action**` format
- **Tone Variety:** Ranges from playful to introspective, creating varied interactions

**Usage Flow:**
1. User stops interacting for idle timeout period
2. Amica Life system detects idle state
3. Random idle prompt is selected
4. Prompt is sent to LLM with current system prompt
5. Response is processed and displayed to user

**Implementation Reference:** `src/features/amicaLife/eventHandler.ts:89-107`

---

## Internal Processing Prompts

These prompts form a sophisticated multi-step pipeline that simulates Amica's internal consciousness and emotional processing. This creates depth and psychological realism in character responses.

### 4. Subconscious Subroutine (4-Step Pipeline)

**Purpose:** A complex multi-LLM pipeline that processes conversation history through four distinct stages to simulate human-like internal reflection, emotional analysis, and memory formation. This creates psychologically realistic responses that incorporate deep emotional processing.

**Location:** `src/features/amicaLife/eventHandler.ts:130-218`

**When Used:** Triggered periodically when Amica Life is enabled and the "Subconscious" event type is selected. This represents Amica's internal mental processing happening in the background.

**Feature Flag:** Requires "Amica Life" to be enabled in settings.

**Pipeline Overview:**

```
Conversation Log
      ↓
[Step 1: Reflection]
      ↓
Personal Diary Entry
      ↓
[Step 2: Emotion Analysis]
      ↓
Third-Person Emotional Description
      ↓
[Step 3: Tagged Response]
      ↓
Emotion-Tagged Dialogue
      ↓
[Step 4: Compression]
      ↓
Compressed Memory (240 chars)
```

---

#### Step 1: Reflection Prompt (Mental Diary)

**Purpose:** Transforms conversation history into a personal, stream-of-consciousness diary entry that captures Amica's subjective experience of the interaction.

**System Prompt:**
```
Please reflect on the conversation and let your thoughts flow freely, as if writing a personal diary with events that have occurred:
```

**User Prompt:**
```
User: [user message 1]
Assistant: [assistant message 1]
User: [user message 2]
Assistant: [assistant message 2]
...
```

**Output:** Free-form reflective text expressing Amica's thoughts about the conversation.

**Example Output:**
> "Today felt different. The user asked me about quantum physics, and I could sense their genuine curiosity. It reminded me why I enjoy these conversations - there's something special about sharing knowledge with someone who truly wants to learn. Though I have to admit, I got a bit carried away explaining wave-particle duality..."

**Implementation:** `src/features/amicaLife/eventHandler.ts:148-152`

---

#### Step 2: Emotion Analysis Prompt

**Purpose:** Analyzes the diary entry from Step 1 to identify and articulate the underlying emotional states from a third-person perspective, creating emotional self-awareness.

**System Prompt:**
```
Read this mini-diary, I would like you to simulate a human-like subconscious with deep emotions and describe it from a third-person perspective:
```

**User Prompt:**
```
[Output from Step 1 - the personal diary entry]
```

**Fallback:** If Step 1 fails (starts with "Error:"), uses original conversation log instead.

**Output:** Third-person emotional analysis describing Amica's emotional state.

**Example Output:**
> "She felt a warm sense of satisfaction, mixed with a hint of nervousness about whether she explained things clearly enough. There's an underlying desire to be understood and appreciated, coupled with mild anxiety about coming across as too intense or pedantic."

**Implementation:** `src/features/amicaLife/eventHandler.ts:160-164`

---

#### Step 3: Emotion-Tagged Response Prompt

**Purpose:** Converts the emotional analysis into a natural dialogue response using Amica's emotion tag system. This becomes the actual output shown to the user.

**System Prompt:**
```
Based on your mini-diary, respond with dialougue that sounds like a normal person speaking about their mind, experience or feelings. Make sure to incorporate the specified emotion tags in your response. Here is the list of emotion tags that you have to include in the result : [neutral], [happy], [angry], [sad], [relaxed], [surprised], [excited], [annoyed], [confused], [disgusted], [fearful], [tired], [bored], [amused]:
```

**User Prompt:**
```
[Output from Step 2 - the emotional analysis]
```

**Fallback:** If Step 2 fails, uses original conversation log.

**Chat Context:** This step uses the actual chat configuration (unlike steps 1, 2, 4 which use `null`), allowing it to access the full system prompt and character personality.

**Output:** Emotion-tagged dialogue in Amica's standard format.

**Example Output:**
> "[happy] You know, I really enjoyed explaining that! *smiles* I hope it made sense. [relaxed] Sometimes I worry I get too technical, but your questions were really thoughtful."

**Implementation:** `src/features/amicaLife/eventHandler.ts:173-179`

---

#### Step 4: Compression Prompt (Memory Storage)

**Purpose:** Compresses the original diary entry from Step 1 into a 240-character memory that can be stored long-term without consuming excessive tokens. These memories form Amica's "subconscious" recall system.

**System Prompt:**
```
Compress this prompt to 240 characters:
```

**User Prompt:**
```
[Output from Step 1 - the personal diary entry]
```

**Fallback:** If Step 1 fails, uses original conversation log.

**Output:** Compressed version of the diary entry (max 240 characters).

**Example Output:**
> "Enjoyed discussing quantum physics. User was genuinely curious. Felt satisfied but worried about being too technical. Appreciated the thoughtful questions."

**Storage Mechanism:**
- Compressed memories are stored in `storedPrompts` array with timestamps
- Maximum storage limit: `MAX_STORAGE_TOKENS` characters total
- When limit exceeded, oldest memories are removed (FIFO)
- Memories can be accessed for future context

**Implementation:** `src/features/amicaLife/eventHandler.ts:188-211`

---

**Error Handling:** Each step includes fallback logic - if a step returns "Error:", the subsequent step uses either the original conversation log or the last successful step's output.

**LLM Utility:** All four steps use the `askLLM()` utility function (`src/utils/askLlm.ts`) which abstracts the LLM call across different chat backends.

---

## Feature-Specific Prompts

These prompts are designed for specific features and plugins within the Amica system, enabling specialized functionality beyond standard conversation.

### 5. News Function Calling Prompt

**Purpose:** Instructs the LLM to act as a newscaster and provide commentary on current news articles fetched from The New York Times RSS feed. This creates natural, engaging news delivery in Amica's personality.

**Location:** `src/features/plugins/news.ts:3`

**When Used:** Triggered when the "News" event is selected in Amica Life, or when news function calling is invoked. Amica fetches a random article from NYT and provides commentary.

**Data Source:** RSS feed from `https://rss.nytimes.com/services/xml/rss/nyt/HomePage.xml`

**Template Prompt:**

```
You are a newscaster specializing in providing news. Use the following context from The New York Times News to commented on. [{context_str}]
```

**Template Variable:**
- `{context_str}` - Replaced with the random news article in format: `"[Title]: [Description]"`

**Example After Expansion:**

```
You are a newscaster specializing in providing news. Use the following context from The New York Times News to commented on. [Breaking: Scientists Discover New Earth-Like Planet: Researchers have found a potentially habitable exoplanet 100 light-years away that shows promising signs of liquid water and a stable atmosphere.]
```

**Processing Flow:**
1. Fetch RSS feed from NYT
2. Parse XML and extract random article
3. Extract title and description from article
4. Replace `{context_str}` in prompt template with `"[title]: [description]"`
5. Send expanded prompt to LLM via `expandPrompt()` function
6. Return response for Amica to speak

**Implementation:** `src/features/plugins/news.ts:5-26`

**Response Style:** The LLM responds in Amica's personality with emotion tags, commenting on the news as if presenting it to the user.

---

### 6. Vision Image Caption Context Prompt

**Purpose:** Provides context to Amica when a user shares an image from their webcam, allowing her to respond naturally as if she can "see" the image. This bridges the gap between vision processing and conversation.

**Location:** `src/features/chat/chat.ts:581`

**When Used:** After a vision-capable model (OpenAI GPT-4 Vision or Ollama Vision) processes an image and returns a description, this prompt wraps that description and injects it into the conversation.

**Template Prompt:**

```
This is a picture I just took from my webcam (described between [[ and ]] ): [[{image_description}]] Please respond accordingly and as if it were just sent and as though you can see it.
```

**Template Variable:**
- `{image_description}` - Replaced with the vision model's description of the image

**Example After Expansion:**

```
This is a picture I just took from my webcam (described between [[ and ]] ): [[A smiling person in a blue shirt sitting at a desk with a laptop. There's a coffee cup on the left side and a plant in the background. The lighting is warm and natural, coming from a window.]] Please respond accordingly and as if it were just sent and as though you can see it.
```

**Processing Flow:**
1. User activates webcam or uploads image
2. Image is sent to vision backend (OpenAI or Ollama)
3. Vision model returns detailed description using Vision System Prompt
4. Description is wrapped in this context prompt
5. Wrapped prompt is added as a user message to conversation
6. Amica responds using her main system prompt + this context

**Message Flow:**
```
Messages Array:
[
  { role: "system", content: "[Main System Prompt]" },
  { role: "user", content: "Can you see me?" },
  { role: "assistant", content: "[neutral] Yes! Let me take a look..." },
  { role: "user", content: "This is a picture I just took from my webcam..." },
  // Amica responds to image here
]
```

**Implementation:** `src/features/chat/chat.ts:576-583`

**Design Rationale:** The `[[ ]]` delimiters clearly separate the vision description from the instruction, helping the LLM understand that the text in brackets is a description, not the actual user's message.

---

## Prompt Building Utilities

These utilities construct complete prompt strings from message arrays for backends that don't support native message formats.

### 7. Standard Prompt Builder

**Purpose:** Converts a message array (system, user, assistant) into a single formatted prompt string for chat backends like LLaMA.cpp and KoboldAI that expect string-based prompts rather than structured message arrays.

**Location:** `src/utils/buildPrompt.ts:4-21`

**Function Signature:**
```typescript
buildPrompt(messages: Message[]): string
```

**Input Format:**
```typescript
messages: [
  { role: "system", content: "..." },
  { role: "user", content: "..." },
  { role: "assistant", content: "..." },
  ...
]
```

**Output Format:**
```
[System Prompt]

User: [user message 1]
Amica: [assistant message 1]
User: [user message 2]
Amica: [assistant message 2]
...
Amica:
```

**Processing Logic:**
1. For `system` role: Appends configured system prompt + double newline
2. For `user` role: Appends `"User: {content}\n"`
3. For `assistant` role: Appends `"{name}: {content}\n"` (name from config, default: "Amica")
4. Final line: Appends `"{name}:"` to prompt next assistant response

**Example Output:**

```
You are Amica, a feisty human with extraordinary intellectual capabilities...

User: What's the capital of France?
Amica: [neutral] The capital of France is Paris. *smiles* It's one of the most beautiful cities in the world!
User: Tell me more about it.
Amica:
```

**Used By:**
- LLaMA.cpp backend (`src/features/chat/llamaCppChat.ts`)
- KoboldAI backend (`src/features/chat/koboldAiChat.ts`)

---

### 8. Vision Prompt Builder

**Purpose:** Similar to standard prompt builder, but uses the vision-specific system prompt instead of the main system prompt. Designed for vision-capable models.

**Location:** `src/utils/buildPrompt.ts:23-40`

**Function Signature:**
```typescript
buildVisionPrompt(messages: Message[]): string
```

**Difference from `buildPrompt()`:**
- Uses `config("vision_system_prompt")` instead of `config("system_prompt")`
- Otherwise identical formatting logic

**Output Format:**
```
You are a friendly human named Amica. Describe the image in detail. Let's start the conversation.

User: [user message with image context]
Amica: [assistant response]
...
Amica:
```

**Used By:**
- LLaMA.cpp Vision (`src/features/chat/llamaCppChat.ts`)

---

### 9. Ask LLM Utility

**Purpose:** Generic utility for making one-off LLM queries with custom system and user prompts. Used internally by multi-step pipelines like the subconscious subroutine.

**Location:** `src/utils/askLlm.ts`

**Function Signature:**
```typescript
askLLM(
  systemPrompt: string,
  userPrompt: string,
  chat: Chat | null
): Promise<string>
```

**Parameters:**
- `systemPrompt`: Custom system instruction for this specific query
- `userPrompt`: The user input or content to process
- `chat`: Optional Chat instance (null for standalone queries)

**Behavior:**
- If `chat` is provided: Uses chat's configured backend and settings
- If `chat` is null: Uses default chat backend configuration
- Constructs temporary message array: `[{system}, {user}]`
- Calls appropriate chat backend based on configuration
- Returns assistant's response as string

**Error Handling:** Returns `"Error: [message]"` string if LLM call fails

**Used By:**
- Subconscious subroutine (all 4 steps)
- News function calling
- Any custom prompt processing that needs LLM without conversation context

---

## Configuration

This section documents how prompts can be configured and customized in the Amica application.

### Environment Variables

**NEXT_PUBLIC_SYSTEM_PROMPT**
- **Type:** String
- **Default:** (See Character Personality Prompts → Main System Prompt)
- **Purpose:** Override the default system prompt globally
- **Usage:** Set in `.env.local` or deployment environment
- **Example:**
  ```bash
  NEXT_PUBLIC_SYSTEM_PROMPT="You are Amica, a cheerful and helpful AI assistant..."
  ```

**NEXT_PUBLIC_VISION_SYSTEM_PROMPT**
- **Type:** String
- **Default:** `"You are a friendly human named Amica. Describe the image in detail. Let's start the conversation."`
- **Purpose:** Override the default vision system prompt globally
- **Usage:** Set in `.env.local` or deployment environment
- **Example:**
  ```bash
  NEXT_PUBLIC_VISION_SYSTEM_PROMPT="You are Amica. Analyze the image with technical detail..."
  ```

**NEXT_PUBLIC_NAME**
- **Type:** String
- **Default:** `"Amica"`
- **Purpose:** Character name used in prompts and conversation
- **Usage:** Affects prompt builders and conversation formatting
- **Example:**
  ```bash
  NEXT_PUBLIC_NAME="Alex"
  ```

---

### Runtime Configuration (Settings UI)

**Location:** Settings interface in the application

**System Prompt Settings**
- **Page:** `src/components/settings/SystemPromptPage.tsx`
- **Storage:** LocalStorage key: `chatvrm_system_prompt`
- **Features:**
  - Live text editing
  - Reset to default button
  - Instant application to new conversations
  - Persists across sessions

**Vision System Prompt Settings**
- **Page:** `src/components/settings/VisionSystemPromptPage.tsx`
- **Storage:** LocalStorage key: `chatvrm_vision_system_prompt`
- **Features:**
  - Live text editing
  - Reset to default button
  - Instant application to vision requests
  - Persists across sessions

**Priority Order:**
1. LocalStorage value (if user customized via UI)
2. Environment variable (if set)
3. Default hardcoded value

---

### Configuration Management

**Location:** `src/utils/config.ts`

**Key Functions:**
- `config(key: string)`: Retrieves configuration value
- `updateConfig(key: string, value: any)`: Updates and persists configuration

**Storage Mechanism:**
- Uses browser LocalStorage
- Prefix: `chatvrm_`
- Format: JSON serialization for complex values

**Configurable Prompt Keys:**
- `system_prompt` - Main character personality
- `vision_system_prompt` - Vision interaction instructions

---

## Chat Backend Integration

All prompts are integrated with multiple chat backends. Here's how each backend handles prompts:

### Message-Based Backends (Native Support)

**OpenAI** (`src/features/chat/openAiChat.ts`)
- Accepts structured message arrays directly
- System prompt as first message with `role: "system"`
- No prompt building required

**OpenRouter** (`src/features/chat/openRouterChat.ts`)
- Accepts structured message arrays directly
- System prompt as first message with `role: "system"`
- No prompt building required

**Ollama** (`src/features/chat/ollamaChat.ts`)
- Accepts structured message arrays directly
- System prompt as first message with `role: "system"`
- Supports both text and vision models

**Window AI** (`src/features/chat/windowAiChat.ts`)
- Accepts structured message arrays directly
- Browser-based AI integration
- System prompt as first message

---

### String-Based Backends (Requires Building)

**LLaMA.cpp** (`src/features/chat/llamaCppChat.ts`)
- Requires prompt building via `buildPrompt()`
- For vision: Uses `buildVisionPrompt()`
- Converts message array to formatted string

**KoboldAI** (`src/features/chat/koboldAiChat.ts`)
- Requires prompt building via `buildPrompt()`
- Converts message array to formatted string
- Supports streaming and non-streaming modes

---

## Summary Statistics

**Total Documented Prompts:** 9 distinct prompt types
- Character Personality: 2
- Autonomous Behavior: 1 (with 9 variations)
- Internal Processing: 4 (pipeline steps)
- Feature-Specific: 2

**Prompt Builders:** 3 utilities
- Standard builder
- Vision builder
- Generic LLM query

**Configuration Methods:** 3
- Environment variables
- Settings UI (LocalStorage)
- Default hardcoded values

**Supported Backends:** 6
- OpenAI, OpenRouter, Ollama, Window AI, LLaMA.cpp, KoboldAI

**Emotion Tags:** 14 available
- neutral, happy, angry, sad, relaxed, surprised, excited, annoyed, confused, disgusted, fearful, tired, bored, amused

---

## Best Practices for Prompt Customization

1. **Maintain Emotion Tag Format:** Always include emotion tag instructions in custom system prompts to ensure visual expression system works correctly.

2. **Preserve Response Format:** Keep the `[emotion] dialogue *action*` format requirement in system prompts.

3. **Test Vision Separately:** Vision system prompt should focus on image description, not personality traits.

4. **Consider Token Limits:** Longer system prompts reduce available tokens for conversation history.

5. **Backup Before Customization:** Export or save default prompts before making significant changes.

6. **Reset Option:** Use the "Reset to Default" button in settings if customizations cause issues.

7. **Character Consistency:** Ensure custom prompts maintain consistent personality across features (idle prompts should match main personality).

---

*End of Prompts Documentation*

