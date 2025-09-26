---
colorSchema: light
title: AI Agents Walkthrough
class: relative
transition: slide-left
mdc: true
---

# ai-agents sdk

---

```rb{all|2-17|20-22|24-26|2-5|7-11|12-17|20|21-22|24|26|all}
# Create specialized agents
triage = Agents::Agent.new(
  name: "Triage Agent",
  instructions: "Route customers to the right specialist"
)

sales = Agents::Agent.new(
  name: "Sales Agent",
  instructions: "Answer details about plans",
  tools: [CreateLeadTool.new, CRMLookupTool.new]
)

support = Agents::Agent.new(
  name: "Support Agent",
  instructions: "Handle account realted and technical issues",
  tools: [FaqLookupTool.new, TicketTool.new]
)

# Wire up handoff relationships - clean and simple!
triage.register_handoffs(sales, support)
sales.register_handoffs(triage)     # Can route back to triage
support.register_handoffs(triage)   # Hub-and-spoke pattern

runner = Agents::Runner.with_agents(triage, sales, support)

result = runner.run("Do you have special plans for businesses?")
```

---

## Project Structure

  -   `lib/agents.rb`: The main entry point, handling configuration and loading other components.
  -   `lib/agents/agent.rb`: Defines the `Agent` class, which represents an individual AI agent with role, instructions, and tools.
  -   `lib/agents/tool.rb`: Defines the `Tool` class, the base for creating custom tools that agents can execute.
  -   `lib/agents/agent_runner.rb`: Thread-safe wrapper that manages the agent registry and provides the public API for running conversations.
  -   `lib/agents/runner.rb`: Internal execution engine that handles individual conversation turns. The `Runner.with_agents` factory method returns an `AgentRunner` instance.
  -   `lib/agents/context.rb`: Manages conversation state that persists across agent interactions and handoffs.

---

## Key Concepts

-   **Agent**: An AI assistant with a specific role, instructions, and tools. Each agent can register handoffs to other agents.
-   **Tool**: A custom function that an agent can use to perform actions (e.g., look up customer data, send an email). Tools are thread-safe and receive context as parameters.
-   **Handoff**: The process of transferring a conversation from one agent to another. This happens seamlessly without the user knowing.
-   **AgentRunner**: The thread-safe public API for managing multi-agent conversations. Created via `Runner.with_agents(...)`.
-   **Runner**: Internal execution engine that handles individual conversation turns. Each call to `AgentRunner.run` creates a new Runner instance.
-   **Context**: A shared state object that stores conversation history and agent information, fully serializable for persistence across process boundaries.
-   **Callbacks**: Event hooks for monitoring agent execution in real-time, including agent thinking, tool start/complete, and handoffs. Non-blocking and thread-safe.

---

```mermaid
graph TD
    R[Runner]
    C[Shared Context]

    R --> C

    A1[Agent: Triage]
    A2[Agent: Billing]

    R --> A1
    R --> A2

    T1[Tool: customer_lookup]
    H1[Handoff Tool]

    T2[Tool: check_balance]
    H2[Handoff Tool]

    A1 --> T1
    A1 --> H1

    A2 --> T2
    A2 --> H2

    C -.-> A1
    C -.-> A2

    style R fill:#e1f5fe
    style C fill:#f3e5f5
    style A1 fill:#fff3e0
    style A2 fill:#fff3e0
```

---

## Almost Full Architecture

```mermaid
graph LR
    U[User] -->|"Runner.with_agents(...)"| AR[AgentRunner<br/>Thread-safe API]

    AR -->|"Creates per turn"| R[Runner<br/>Execution Engine]

    R --> C[Shared Context<br/>Serializable State]

    R --> A1[Agent: Triage<br/>Role + Instructions]
    R --> A2[Agent: Billing<br/>Role + Instructions]

    A1 -->|"Has"| T1[Tool: customer_lookup]
    A1 -->|"Can handoff to"| A2

    A2 -->|"Has"| T2[Tool: check_balance]
    A2 -->|"Can handoff to"| A1

    AR -->|"Fires"| CB[Callbacks<br/>Real-time Events]

    C -.->|"Persists across"| A1
    C -.->|"Persists across"| A2

    CB -->|"on_agent_thinking"| U
    CB -->|"on_tool_start/complete"| U
    CB -->|"on_agent_handoff"| U

    style AR fill:#e1f5fe
    style R fill:#fff9c4
    style C fill:#f3e5f5
    style A1 fill:#fff3e0
    style A2 fill:#fff3e0
    style CB fill:#e8f5e9
```

---

```mermaid
sequenceDiagram
    participant U as User
    participant AR as Runner
    participant A as Agent
    participant LLM as LLM Provider

    U->>AR: "What time is it?"
    AR->>A: Select Agent + Context
    A->>LLM: Process with Instructions
    LLM-->>A: Generate Response
    A-->>AR: Final Response
    AR-->>U: "It's currently 3:30 PM"
```

---

```mermaid
sequenceDiagram
    participant U as User
    participant AR as Runner
    participant A as Agent
    participant LLM as LLM Provider
    participant T as Tools

    U->>AR: "What's my account balance?"
    AR->>A: Select Agent + Context
    A->>LLM: Process with Instructions
    LLM->>T: lookup_customer_balance
    T-->>LLM: "$1,250.00"
    LLM-->>A: Generate Response
    A-->>AR: Final Response
    AR-->>U: "Your current balance is $1,250.00"
```

---

```mermaid
sequenceDiagram
    participant U as User
    participant AR as Runner
    participant A1 as Triage Agent
    participant A2 as Billing Agent
    participant LLM as LLM Provider
    participant T as Tools

    U->>AR: "I have a billing question about my last invoice"
    AR->>A1: Select Triage Agent
    A1->>LLM: Process with Instructions
    LLM-->>A1: "Transfer to Billing"
    A1->>AR: Request Handoff to Billing
    AR->>A2: Load Billing Agent + Context
    A2->>LLM: Process with Billing Instructions
    LLM->>T: lookup_invoice_details
    T-->>LLM: Invoice data
    LLM-->>A2: Generate Response
    A2-->>AR: Final Response
    AR-->>U: "I found your invoice. Here are the details..."
```

---

## Best Practices

- Make sure the scope of sub agents don't overlap
- Use different models for different use cases
- The best tool call is the one that is not executed
- Agents Error Compound
- Cost increases quadratically

---
