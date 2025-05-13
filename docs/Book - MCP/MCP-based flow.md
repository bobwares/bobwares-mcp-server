Below is the extended flow, including the user interface (UI) layer and how it interacts with your MCP-enabled agent. 

1Each numbered step aligns UI events with the underlying MCP protocol exchanges.

---

## 0. User ↔ UI

1. **User types message**

    * User enters text into a chat input box (“Summarize last quarter’s sales figures.”) and clicks **Send** (or presses Enter).

2. **UI emits event**

    * A `sendMessage()` handler packages the text into a client message and forwards it to the Agent.

---

## 1. UI → Agent (Client Layer)

```text
UI.sendMessage("Summarize last quarter’s sales figures.")
```

* The client-side code (e.g. a React component) invokes the agent SDK’s `agent.receiveUserMessage(...)`.

---

## 2. Agent → Context Provider(s)

1. **Agent requests context**

   ```json
   {
     "type": "ContextFetch",
     "resource": "sales_database",
     "query": "Q1 2025 figures"
   }
   ```
2. **Context services respond**

   ```json
   {
     "type": "ContextFetchResponse",
     "data": [
       { "type": "text", "text": "Q1 2025 total revenue: $1.2M" },
       { "type": "text", "text": "Q1 2025 units sold: 34,000" }
     ]
   }
   ```

---

## 3. Agent Assembles & Sends Prompt to LLM

* Combines system, user, and context messages into an OpenAI-compatible payload.
* Invokes `chat.completions.create({ messages: […] , stream: true })`.

---

## 4. LLM → Agent (Streaming)

* As each token arrives, the agent emits a client-side event:

  ```js
  agent.onToken(token => UI.appendToken(token))
  ```

* **UI** appends tokens in real time to the chat window.

---

## 5. Detect & Execute Tool Calls

1. **LLM emits a tool invocation**

   ```json
   { "type": "tool_invocation", "tool": "generate_chart", "args": { … } }
   ```

2. **Agent** pauses streaming to run the tool:

   ```js
   UI.showStatus("Running chart generation…")
   agent.invokeTool("generate_chart", args)
     .then(result => {
       UI.hideStatus()
       agent.injectToolResult(result)
     })
   ```

3. **Tool service** returns, e.g.:

   ```json
   { "type": "tool_result", "tool": "generate_chart", "data": "<image_url>" }
   ```

4. **UI** displays the chart inline:

   ```js
   UI.displayImage("<image_url>")
   ```

---

## 6. Final Assembly & UI Rendering

* Once all tools have run, the agent emits an **assistant message** event:

  ```js
  agent.onAssistantMessage(msg =>
    UI.renderMessage(msg)
  )
  ```

* **UI** renders the final text, along with any embedded images or interactive components.

---

## Full Sequence Diagram (with UI)



```
User         UI              Agent                ContextSvc        LLM               ToolSvc
|            |               |                     |                 |                 |
|--type-->   |               |                     |                 |                 |
|  & Send    |--sendMsg----->|                     |                 |                 |
|            |               |--ContextFetch-----> |                 |                 |
|            |               |<--ContextResp-------|                 |                 |
|            |               |--ChatRequest------------------------->|                 |
|            |               |                     |<--ChatStream--->|                 |
|            |<--appendToken(token)------------StreamTokens--------->|                 |
|            |               | Detect tool call    |                 |                 |
|            |--showStatus-->|                     |                 |                 |
|            |               |--ToolInvoke-------------------------------------------->|
|            |               |<--ToolResult--------------------------------------------|
|            |<--hideStatus & displayImage---------------------------------------------|
|            |               | Merge & finalize response                               |
|            |               |<--renderMessage(assistant)------------------------------|
|<--sees reply with chart and text-----------------------------------------------------|
```


### UI Components & Handlers

* **ChatWindow**

    * `appendToken(token: string)`
    * `renderMessage(msg: { role, content, attachments })`

* **InputBox**

    * `onSubmit(text: string)`

* **StatusIndicator**

    * `show(text: string)` / `hide()`

* **ToolOutput**

    * `displayImage(url: string)`
    * `displayTable(data: any[])`

This layered flow ensures that **user actions** and **visual feedback** stay tightly synchronized with every underlying MCP and LLM exchange.
