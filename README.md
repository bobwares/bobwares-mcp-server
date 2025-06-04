# Building a Remote MCP Server on Cloudflare (Without Auth)

A Model Context Protocol (MCP) server is a backend system that empowers AI agents by providing them with a structured way to access and utilize tools, and interact with external services or data sources (Context Providers). Its primary purpose is to extend the capabilities of AI agents beyond the core functionalities of the Large Language Model (LLM), enabling them to perform specific tasks, retrieve information, and execute actions in a secure and managed environment.

This project provides a template for building and deploying such an MCP server on Cloudflare Workers. It demonstrates how to define an agent and equip it with tools that can be called upon by the AI model.

This example allows you to deploy a remote MCP server that doesn't require authentication on Cloudflare Workers. 

## Getting Started

You have two main options to get started with this MCP server template:

1.  **Deploy directly to Cloudflare Workers:**

    [![Deploy to Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/cloudflare/ai/tree/main/demos/remote-mcp-authless)

    Clicking this button will deploy the MCP server directly to your Cloudflare account. After deployment, it will be accessible at a URL similar to: `remote-mcp-server-authless.<your-account>.workers.dev/sse`

2.  **Set up a local project for development and deployment:**

    Use the following command to create a new project on your local machine based on this template:
    ```bash
    npm create cloudflare@latest -- my-mcp-server --template=cloudflare/ai/demos/remote-mcp-authless
    ```
    This will create a directory named `my-mcp-server` (or your chosen name). Once created:
    ```bash
    cd my-mcp-server
    # To deploy to Cloudflare Workers:
    npm run deploy
    # Or to run a local development server:
    npm run dev
    ```
    The local development server will typically be available at `http://localhost:8787`.

## Core Functionality

This MCP server template is designed to be deployed on **Cloudflare Workers**, providing a scalable and serverless environment for your AI agent's tools.

### Tool Definition

Tools are the heart of the MCP server, enabling AI agents to perform specific actions and interact with external systems. In this template, tools are defined within the `init()` method in the `src/index.ts` file. You can add new tools or modify existing ones by using the `this.server.tool()` method. Each tool definition specifies:
- A unique name for the tool.
- The expected input parameters (often defined using a schema library like Zod for validation).
- An asynchronous function that implements the tool's logic and returns the result.

### Example Tools

The template includes a couple of example tools in `src/index.ts` to demonstrate how to define them:

1.  **`add`**:
    *   A simple tool that takes two numbers, `a` and `b`, as input.
    *   It returns their sum.

2.  **`calculate`**:
    *   A more versatile calculator tool that accepts three parameters:
        *   `operation`: A string specifying the operation to perform (can be `"add"`, `"subtract"`, `"multiply"`, or `"divide"`).
        *   `a`: The first number.
        *   `b`: The second number.
    *   It performs the specified arithmetic operation and returns the result. It also includes error handling for cases like division by zero.

These examples serve as a starting point. You are encouraged to expand upon them or create new, more complex tools tailored to your AI agent's specific needs, such as interacting with APIs, databases, or other services.

## Customizing your MCP Server

The true power of your MCP server comes from the custom tools you define. These tools extend the capabilities of your AI agent, allowing it to perform a wide range of tasks.

### Adding New Tools

As highlighted in the "Core Functionality" section, all tools are defined within the `init()` method of the `MyMCP` class in `src/index.ts`. To add a new tool or modify an existing one, you will edit this file.

Each tool is registered using the `this.server.tool()` method, which takes three main arguments:

1.  **`name` (string):**
    *   This is the unique identifier for your tool that the AI agent will use to call it. Choose a descriptive and concise name (e.g., `"getUserDetails"`, `"searchKnowledgeBase"`).

2.  **`inputSchema` (Zod schema object):**
    *   This defines the expected structure and data types for the input parameters your tool accepts. This project uses the `zod` library for schema definition and validation.
    *   `zod` helps ensure that your tool receives data in the correct format before execution, preventing errors and improving reliability. For example, you might define an input schema like `{ userId: z.string(), query: z.string().optional() }`.
    *   You can learn more about `zod` from its official documentation.

3.  **`handlerFunction` (async function):**
    *   This is an asynchronous JavaScript function that contains the actual logic of your tool.
    *   It receives an object containing the validated input parameters (matching the `inputSchema`).
    *   The function should perform the desired action (e.g., fetch data from an API, perform a calculation, interact with a database) and then return a result object, typically in the format: `{ content: [{ type: "text", text: "Your result here" }] }`.

### Example Tool Definition Structure

Here's a simplified example of how you might define a custom tool within the `init()` method in `src/index.ts`:

```typescript
// Inside src/index.ts, within the init() method of the MyMCP class:

// ... other tool definitions like add, calculate ...

this.server.tool(
  "myCustomTool", // 1. Tool name
  { // 2. Input schema using zod
    productId: z.string(),
    quantity: z.number().positive()
  },
  async (inputs) => { // 3. Handler function
    // inputs.productId and inputs.quantity are validated
    console.log(`Processing tool: myCustomTool with inputs:`, inputs);

    // Your tool logic here (e.g., interact with an inventory API)
    const { productId, quantity } = inputs;
    const stockAvailable = await getStockLevel(productId); // Imaginary function

    if (quantity > stockAvailable) {
      return { content: [{ type: "text", text: `Error: Only ${stockAvailable} items available for ${productId}.` }] };
    }

    return { content: [{ type: "text", text: `Successfully processed order for ${quantity} of ${productId}.` }] };
  }
);

// Make sure to define or import any functions like getStockLevel if you use them.
```

The existing `add` and `calculate` tools in `src/index.ts` serve as practical examples. We encourage you to study them and then build your own tools to unlock the full potential of your AI agent. Whether it's connecting to third-party APIs, querying databases, or performing complex business logic, custom tools are the key.

## Deployment

This MCP server is designed for deployment on Cloudflare's global network as a Cloudflare Worker.

### Deployment Methods

As outlined in the "Getting Started" section, you have two primary ways to deploy your server:

1.  **One-Click Deploy:** Use the [![Deploy to Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/cloudflare/ai/tree/main/demos/remote-mcp-authless) button for an automated setup and deployment to your Cloudflare account.
2.  **Manual Deployment via CLI:** If you've set up the project locally using `npm create cloudflare@latest ...`, you can deploy it from your project directory using the command:
    ```bash
    npm run deploy
    ```

Both methods will package and deploy your worker code, including your custom tools, to Cloudflare.

### Server Endpoints

Once successfully deployed, your MCP server will expose two main endpoints for communication with MCP clients:

*   **`/sse`**: This endpoint uses Server-Sent Events (SSE) to stream responses back to the client. SSE is a common standard for uni-directional communication from server to client and is often preferred for AI agent interactions that involve streaming of text or tool updates.
*   **`/mcp`**: This endpoint provides a standard HTTP request/response mechanism for MCP interactions.

### URL Structure

After deployment, your server will be accessible via a unique URL provided by Cloudflare Workers, typically in the format:
`https://remote-mcp-server-authless.<your-cloudflare-account-name>.workers.dev`

To connect an MCP client, you will use the full path to one of the server endpoints. For example, to use the SSE endpoint, your URL would be:
`https://remote-mcp-server-authless.<your-cloudflare-account-name>.workers.dev/sse`

Remember to replace `<your-cloudflare-account-name>` with your actual Cloudflare account identifier.

## Connecting to Your MCP Server

Once your MCP server is deployed or running locally, you can connect to it from various MCP-compatible clients. Here are a couple of examples:

### Using Cloudflare AI Playground

The Cloudflare AI Playground provides a web interface to interact with your MCP server:

1.  Navigate to [https://playground.ai.cloudflare.com/](https://playground.ai.cloudflare.com/).
2.  In the appropriate input field (often labeled "MCP Server URL" or similar), enter the `/sse` endpoint of your deployed MCP server. This URL will typically be:
    `https://remote-mcp-server-authless.<your-cloudflare-account-name>.workers.dev/sse`
    (Replace `<your-cloudflare-account-name>` with your actual Cloudflare account identifier).
3.  You should then be able to access and use the tools defined in your MCP server directly from the playground.

### Using Claude Desktop (via mcp-remote)

You can connect local MCP clients like Claude Desktop to your MCP server (whether it's running remotely on Cloudflare Workers or locally via `npm run dev`) by using the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) proxy tool. This tool forwards a local port to your MCP server's endpoint.

1.  **Install `mcp-remote` (if you haven't already):**
    It's typically run via `npx`, so direct installation might not be needed. If you prefer to install it globally:
    ```bash
    npm install -g mcp-remote
    ```

2.  **Configure Claude Desktop:**
    *   Follow the instructions in [Anthropic's Quickstart for users](https://modelcontextprotocol.io/quickstart/user) to locate the Claude Desktop configuration file.
    *   Go to Settings > Developer > Edit Config within Claude Desktop.
    *   Update the `mcpServers` section. The `command` will be `npx` (or `mcp-remote` if globally installed and in your PATH), and the `args` will include `mcp-remote` followed by your server's `/sse` endpoint URL.

    **Example Configuration:**
    ```json
    {
      "mcpServers": {
        "my_remote_calculator": { // You can name this entry as you like
          "command": "npx",
          "args": [
            "mcp-remote",
            // Replace with YOUR actual MCP server /sse endpoint URL below:
            // e.g., "https://remote-mcp-server-authless.YOUR_ACCOUNT_NAME.workers.dev/sse" for a deployed server
            // OR "http://localhost:8787/sse" if you are running the server locally with `npm run dev`
            "YOUR_MCP_SERVER_SSE_URL"
          ]
        }
      }
    }
    ```
    **Important:** Replace `"YOUR_MCP_SERVER_SSE_URL"` with the actual `/sse` URL of your MCP server.

3.  **Restart Claude Desktop:** After saving the configuration changes, restart Claude Desktop. Your server's tools should then be available.

## Advanced Topics

While this template provides a solid foundation for a single MCP server, the world of AI agents and their capabilities is vast. Here are some pointers for further exploration:

### Agent Manager Pattern

For complex tasks that might benefit from the coordination of multiple specialized AI agents, you might consider the **Agent Manager Pattern**. This is a hierarchical approach where a top-level "manager" agent assigns tasks to and coordinates the work of several sub-agents.

*   To learn more about this orchestration pattern, refer to the document: [`docs/Book - MCP/AI Agent Manager Pattern.md`](docs/Book%20-%20MCP/AI%20Agent%20Manager%20Pattern.md).

### Further Exploration

*   The `docs/` directory in this project contains additional information about AI agents, their design, and related concepts.
*   For more comprehensive information about the Model Context Protocol itself, including specifications and community resources, visit [modelcontextprotocol.io](https://modelcontextprotocol.io).

## Contributing

Contributions are welcome and greatly appreciated! If you have ideas for improvements, new features, or bug fixes, please follow these steps:

1.  **Fork the Repository:** Start by forking the main repository to your own GitHub account.
2.  **Create a New Branch:** For each new feature or bug fix, create a dedicated branch in your fork. A good branch name might be `feature/your-feature-name` or `fix/issue-description`.
3.  **Make Your Changes:** Implement your changes, additions, or fixes in your branch. Ensure your code is clear and well-commented where necessary.
4.  **Commit Your Changes:** Commit your work with clear and descriptive commit messages. This helps everyone understand the purpose of your changes.
5.  **Push Your Branch:** Push your local branch and its commits to your fork on GitHub.
6.  **Submit a Pull Request:** Open a pull request from your branch in your fork to the main repository. Provide a clear title and description for your pull request, outlining the changes you've made.

We look forward to your contributions!

## License

This project is licensed under the MIT License. See the `LICENSE` file for details.
