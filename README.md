# Song Recommender with LangGraph Memory

Demonstration of **LangGraph memory persistence** using conversational AI to recommend music based on user preferences. This project showcases how to build stateful, multi-turn conversations where the AI remembers context across messages.

## 🎯 Goals

This project exemplifies:
- **Memory Persistence**: Using LangGraph's `MemorySaver` for conversation history
- **Thread-based Sessions**: Separate conversation contexts per user/session
- **Conversational AI**: Natural dialogue that asks questions and builds context
- **Minimal Architecture**: Simple single-node graph focusing on memory
- **Real LLM Integration**: Live testing with actual OpenRouter API calls

## Features

- 🎵 **Smart Recommendations**: AI suggests songs based on learned preferences
- 💬 **Conversational Memory**: Remembers user name, favorite bands, genres, etc.
- 🔄 **Thread Isolation**: Different users/sessions maintain separate memories
- 📝 **Dynamic Learning**: Updates understanding as conversation progresses
- 🧪 **Integration Tests**: Real API tests verify memory persistence

## Architecture

### LangGraph Workflow

https://docs.langchain.com/oss/javascript/integrations/vectorstores/libsql
https://docs.langchain.com/oss/javascript/langgraph/persistence#memory-store

```
START → chat (with memory) → END
         ↓
   Checkpointer persists state
   across invocations
```

### Project Structure

```
src/
  ├── config.ts                     # Configuration with memory settings
  ├── index.ts                      # Interactive CLI chat interface
  ├── graph/
  │   ├── graph.ts                  # Simple graph with one chat node
  │   └── factory.ts                # Graph builder with memory service
  ├── services/
  │   ├── memory-service.ts         # MemorySaver initialization
  │   └── openrouter-service.ts     # LLM client with chat method
  └── utils/
      └── prompt-loader.ts          # Template loading
prompts/
  └── system.txt                    # System prompt for song recommender
tests/
  └── chat-memory.test.ts           # Integration tests with real LLM
```

## How Memory Works

This project demonstrates **two types of memory**:

### 1. Conversation Memory (MemorySaver)
- **Thread ID**: Each conversation has a unique `thread_id` for isolation
- **In-Memory Storage**: Conversation history stored in memory during the session
- **Automatic Replay**: Previous messages included when invoking with same `thread_id`

### 2. User Preferences (LibSQL Vector Store)
- **Vector Embeddings**: User preferences stored as embedded documents using OpenAI embeddings
- **Semantic Search**: Uses `@libsql/client` with vector similarity for intelligent matching
- **Persistent Storage**: Preferences persisted to SQLite database across restarts
- **Document Structure**: Each preference stored with rich metadata (name, bands, genres, age, mood)

**Why Vector Store for Preferences?**
- Better semantic understanding of user taste
- Can find similar music preferences across users
- Enables more intelligent recommendations based on embedded meaning
- Scalable approach for large user bases

This demonstrates production-ready patterns:
- Conversation state for short-term context
- Vector store for long-term, searchable user data
- Hybrid memory approach for AI applications

```typescript
// First message
await graph.invoke(
  { messages: [new HumanMessage("Hi! I'm Alex")] },
  { configurable: { thread_id: "user-123" } }
);

// Second message - AI remembers "Alex" from previous message
await graph.invoke(
  { messages: [new HumanMessage("Recommend some songs")] },
  { configurable: { thread_id: "user-123" } }
);
```

## Setup

1. **Install dependencies**:
   ```bash
   npm install
   ```

2. **Configure environment** (`.env`):
   ```bash
   OPENROUTER_API_KEY=your_openrouter_key_here
   OPENROUTER_HTTP_REFERER=http://localhost:3000
   OPENROUTER_X_TITLE=Song Recommender
   OPENAI_API_KEY=your_openai_key_here  # For embeddings
   ```

3. **Run the chat interface**:
   ```bash
   npm run chat
   ```

4. **Run tests**:
   ```bash
   npm test
   ```

## Example Conversation

```
You: Hi!
AI: Hello there! It's great to see you! What's your name, and what kind of music do you enjoy?

You: My name is Alex and I love rock music, especially Foo Fighters.
AI: Nice to meet you, Alex! Foo Fighters are amazing! Do you have any other favorite bands or specific rock subgenres you enjoy?

You: Can you recommend some songs for me?
AI: Absolutely, Alex! Since you love Foo Fighters, you might enjoy "Everlong" for its powerful energy...
```

## Key Learnings

- **Stateful Conversations**: Memory enables natural multi-turn dialogues
- **Thread Management**: Separate threads isolate different users/sessions
- **Simple Architecture**: One node + memory checkpointer is all you need
- **Testing Strategy**: Integration tests with real LLM verify behavior

## Next Steps

- Add SQLite/PostgreSQL persistence for production use
- Implement conversation summarization for long histories
- Add structured data extraction to store preferences explicitly
- Build web interface with session management
