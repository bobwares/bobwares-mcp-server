## Agent Design Foundations

Actors 

  - **Agent**: The core component that manages the interaction between the user, UI, LLM, and tools.
  - **User**: The individual who interacts with the agent, providing input and receiving output.
  - **UI**: The interface through which users interact with the agent.
  - **Context Provider**: External data sources that provide context to the agent.
  - **LLM Endpoint**: The API endpoint for the language model.
  - **Tool Service**: Modular services that perform specific tasks as requested by the agent.


Model - The LLM powering the agent's reasoning and decision-making.

Tools - External functions or APIs the agent can call to perform specific tasks (e.g., data retrieval, calculations).

Instructions - Explicit guidlines and guardrails for the agent's behavior, including safety and ethical considerations.

Agent categories ie chatbot, analyst, assistant, etc.
- **Chatbot**: Engages in conversational interactions with users.
- **Analyst**: Analyzes data and provides insights.
- **Assistant**: Assists users in completing tasks or finding information.
  - **Agent**: A general-purpose agent that can perform a variety of tasks.
    - **Chatbot**: A specialized agent focused on conversational interactions.


Tools  


## Flow Engineering 
- It is designing the workflow of the agents.
- defines who does what and who has what role.
- In conjunction with Prompt Engineering, it creates a complete system.


Self Consistency 
Tree of thought



