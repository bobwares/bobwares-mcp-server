In AI agent design, the **Manager Pattern** is a hierarchical, multi-agent orchestration approach in which a top-level “manager” agent:

* **Assigns tasks** to specialized sub-agents (e.g. market research, financial analysis, competitor analysis)
* **Coordinates** their work flows and dependencies
* **Aggregates** or compiles the final outputs into a cohesive result

This pattern is especially useful for complex, multi-step projects that benefit from distributing responsibility across focused agents while retaining centralized oversight and integration. For example, in a business-report use case you might have:

1. **Manager Agent** – oversees the entire report creation
2. **Sub-Agent A** – conducts market research
3. **Sub-Agent B** – performs financial analysis
4. **Sub-Agent C** – gathers competitor insights
5. **Manager Agent** – compiles and synthesizes all sub-agent outputs into the final document ([linkedin.com][1])

[1]: https://www.linkedin.com/pulse/ai-agents-uncovered-must-know-summary-design-patterns-otim-fredrick-yazbf?utm_source=chatgpt.com "AI Agents Uncovered: Must-Know Summary, Design Patterns, and ..."
                        |