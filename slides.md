---
colorSchema: light
title: AI Agents Walkthrough
class: relative
transition: slide-left
mdc: true
theme: ./theme
fonts:
  sans: 'Haskoy'
---

# Building AI Agents with Ruby

<img class="h-20 w-auto inline drop-shadow absolute bottom-30 right-64" src="/woot.png"/>
<img class="h-24 w-auto inline rotate-30 drop-shadow absolute bottom-36 right-34" src="/ruby.png"/>

Pranav Raj S<br>
Co-founder, Chatwoot

---

<img src="/dashboard.png" class="inset-0 fixed">

---

# Tech Stack

- <span>**Ruby on Rails**</span> monolith with **Sidekiq** on the backend
- **Vue** for frontend
- **Postgres** is the DB
- **Redis** is the cache
- All deployed on **AWS**

<img src="/architecture.png" class="h-3/4 w-auto absolute right-0 bottom-4">

---
layout: cover
---

# Building Captain

- Native AI Agent in Chatwoot
- Trains on your website and past conversations
- Built using OpenAI Ruby SDK


---
layout: cover
---

<img src="/implementation.png"/>


---
layout: cover
---

# Limitations

<v-clicks>

- Doesn't scale beyond FAQs
- It's really difficult to handle multiple scenarios
- System prompt and user instructions may not always align
- It was harder to build agents dynamically

</v-clicks>

---
layout: cover
---

# State of AI in Ruby

<v-clicks>

- OpenAI/Claude/OpenRouter Ruby SDK
- RubyLLM
- ActiveAgents
- LangChain.rb

</v-clicks>

---
layout: cover
---

<img src="/expectation.png"/>

---
layout: cover
---

<img src="/openai-agents-sdk.png" class="absolute w-full top-8 shadow-lg outline outline-1 outline-gray-200 rounded-md overflow-hidden">

---
layout: cover
---


<section class="grid gap-5">
  <Card title="Framework to build a multi-agent system"><carbon:checkmark class="text-green-400" /></Card>
  <v-clicks>
    <Card v-after title="Supports handoffs"><carbon:checkmark class="text-green-400" /></Card>
    <Card v-after title="Supports guardrails"><carbon:checkmark class="text-green-400" /></Card>
    <Card v-after title="Great API to build Tools"><carbon:checkmark class="text-green-400" /></Card>
    <Card v-after title="Built by the folks at OpenAI"><carbon:checkmark class="text-green-400" /></Card>
    <Card v-after title="Written in Python"><carbon:close class="text-red-600" /></Card>
  </v-clicks>
</section>

---
layout: cover
---

<section class="grid grid-cols-2 gap-4">
  <v-clicks>
  <Card title="Use the OpenAI Agents SDK"/>
  <Card title="Build our own"/>
  <Card description="The framework is established. It is maintained by a team larger than ours"/>
  <Card description="We might fall short of features, will have to build a lot of expertise internally"/>
  <Card description="Since it's in Python, we will have to build infrastructure to pipe data from our Rails App"/>
  <Card description="We can natively integrate in our Rails App"/>
  <Card description="Will be another service that we, and our OSS community will have to manage"/>
  <Card description="Easy to maintain and run in the long run"/>
  </v-clicks>
</section>

---

<img class="shadow-lg outline outline-1 outline-gray-200 rounded-md overflow-hidden h-full mx-auto -mt-2" src="/ai-agents.png"/>
<div class="text-xs mx-auto text-center mt-4">ai-agents.chatwoot.dev</div>

---

# ai-agents

<v-clicks>

- Built using RubyLLM
- Multi-Agent Orchestration
  - Peer to Peer with seamless handoffs
  - Agent as a tool pattern
- Supports Tool Calling
- Supports Structured Output
- Allows for a shared context
- Provider Agnostic
</v-clicks>

---

```rb{all|2-17|20-22|24-26|2-5|7-11|12-17|20|21-22|24|26|all}
# Create specialized agents
triage = Agents::Agent.new(
  name: "Triage Agent",
  instructions: "Route customers to the right specialist"
)

sales = Agents::Agent.new(
  name: "Sales Agent",
  instructions: "Answer details about pricing, plans, how to upgrade etc.",
  tools: [CreateLeadTool.new, CRMLookupTool.new]
)

support = Agents::Agent.new(
  name: "Support Agent",
  instructions: "Handle account related and technical issues",
  tools: [FaqLookupTool.new, TicketTool.new]
)

# Wire up handoff relationships - clean and simple!
triage.register_handoffs(sales, support)
sales.register_handoffs(triage)     # Can route back to triage
support.register_handoffs(triage)

runner = Agents::Runner.with_agents(triage, sales, support)

result = runner.run("Do you have special plans for businesses?")
```

---

```rb{all|2-3|5,18|1|2|3|5|6-7|10-14|17|all}
class CrmLookupTool < Agents::Tool
  description "Look up customer account information by account ID"
  param :account_id, type: "string", desc: "Customer account ID (e.g., CUST001)"

  def perform(tool_context, account_id:)
    customer = Customer.find(account_id)
    return "Customer not found" unless customer

    # Store customer information in shared state for other tools/agents
    tool_context.state[:customer_id] = account_id.upcase
    tool_context.state[:customer_name] = customer["name"]
    tool_context.state[:customer_email] = customer["email"]
    tool_context.state[:current_plan] = customer["plan"]["name"]
    tool_context.state[:next_bill_date] = customer["billing"]["next_bill_date"]

    # Return the entire customer data as JSON for the agent to process
    customer.to_json
  end
end
```

---

# Key Concepts

-   **Agent**: An AI assistant with a specific role, instructions, and tools. Each agent can register handoffs to other agents.
-   **Tool**: A custom function that an agent can use to perform actions (e.g., look up customer data, send an email). Tools are thread-safe and receive context as parameters.
-   **Handoff**: The process of transferring a conversation from one agent to another. This happens seamlessly without the user knowing.
-   **AgentRunner**: The public API for managing multi-agent conversations. Created via `Runner.with_agents(...)`.
-   **Context**: A shared state object that stores conversation history and agent information, fully serializable for persistence across process boundaries.

---

# Project Structure

  -   `lib/agents.rb`: The main entry point, handling configuration and loading other components.
  -   `lib/agents/agent.rb`: Defines the `Agent` class, which represents an individual AI agent with role, instructions, and tools.
  -   `lib/agents/tool.rb`: Defines the `Tool` class, the base for creating custom tools that agents can execute.
  -   `lib/agents/agent_runner.rb`: Thread-safe wrapper that manages the agent registry and provides the public API for running conversations.
  -   `lib/agents/runner.rb`: Internal execution engine that handles individual conversation turns. The `Runner.with_agents` factory method returns an `AgentRunner` instance.
  -   `lib/agents/context.rb`: Manages conversation state that persists across agent interactions and handoffs.

---

<img class="shadow-lg outline outline-1 outline-gray-200 rounded-md overflow-hidden h-full mx-auto -mt-2" src="/agents-github.png"/>
<div class="text-xs mx-auto text-center mt-4">https://github.com/chatwoot/ai-agents</div>

---
layout: cover
---

# Some observations

- Make sure the scope of sub agents don't overlap. Write clear descriptions for tools and agents.
- Try out different models for different use cases.

---
class: text-center
---

<div class="flex justify-center items-center h-full flex-col">

  github.com/chatwoot/ai-agents
</div>
