1. **User**
   The human who types requests (e.g. “Summarize last quarter’s sales figures”) and reads the agent’s responses.

2. **UI (User Interface)**
   The front-end component (web chat window, mobile app, etc.) that captures the user’s input, displays streaming tokens, shows status indicators, and renders final messages or tool outputs (charts, tables, images).

3. **Agent**
   Your MCP-enabled client library. It:

    * Receives user messages from the UI
    * Decides when to fetch external context or invoke tools
    * Assembles prompts (system + user + context) for the LLM
    * Streams tokens back to the UI
    * Parses the LLM’s tool calls and manages tool-result injection

4. **Context Provider(s)**
   External data sources or services (databases, search indexes, document stores) that reply to the agent’s `ContextFetch` requests with relevant information segments.

5. **LLM (Large Language Model) Endpoint**
   The language model API (e.g. OpenAI’s `chat/completions`) that, given the assembled prompt, generates streaming text and/or structured tool-invocation instructions.

6. **Tool Service(s)**
   Modular services (chart generators, calculators, data-transformers, etc.) registered via MCP. When the LLM issues a tool call, the agent forwards the call to the appropriate tool service and later merges its output back into the conversation.
