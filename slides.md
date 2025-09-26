---
colorSchema: light
title: AI Agents Walkthrough
class: relative
transition: slide-left
mdc: true
theme: ./theme
---

# Building Agents with Ruby

<img class="h-20 w-auto inline drop-shadow absolute bottom-30 right-64" src="/woot.png"/>
<img class="h-24 w-auto inline rotate-30 drop-shadow absolute bottom-36 right-34" src="/ruby.png"/>

Shivam Mishra<br>
Lead Engineer @ Chatwoot

---

<img src="/screenshot.png" class="inset-0 fixed">

<!--
Let's talk about Chatwoot.
Chatwoot is an open source omnichannel support desk for small teams as well as enterprise, we allow you to connect multiple channels like instagram, whatsapp, email etc.
We've been doing all this with a small team of 8 people, managing this and a mobile app too.
-->

---

# Boring Tech-stack

- <span v-mark="{ at: 2, color: '#CC0000', type: 'box', animationDuration: 500 }">**Ruby on Rails**</span> monolith with **Sidekiq** on the backend
- **Vue** for frontend
- **Postgres** is the DB
- **Redis** is the cache
- All deployed on **AWS**


---
layout: cover
---

# Building Captain

<v-clicks>

- Native AI Agent in Chatwoot
- Trains on your website and past conversations
- Built using OpenAI Ruby SDK

</v-clicks>

---
layout: cover
---

# Limitations

<v-clicks>

- Doesn't scale beyond FAQs
- It's really difficult to handle multiple use-cases
- System prompt and user instructions may not always align

</v-clicks>

---
layout: cover
---

<img v-click.hide src="/openai-agents-sdk.png" class="absolute w-full top-8 shadow-lg outline outline-1 outline-gray-200 rounded-md overflow-hidden">
<img v-after src="/openai-agents-sdk.png" class="absolute w-full top-8 shadow-lg outline outline-1 outline-gray-200 rounded-md overflow-hidden opacity-50 blur-sm">

<section class="grid gap-5">
<v-clicks>
<Card v-after title="Framework to build a multi-agent system"><carbon:checkmark class="text-green-400" /></Card>
<Card v-after title="Supports handoffs"><carbon:checkmark class="text-green-400" /></Card>
<Card v-after title="Supports guardrails"><carbon:checkmark class="text-green-400" /></Card>
<Card v-after title="Great API to build Tools"><carbon:checkmark class="text-green-400" /></Card>
<Card v-after title="Built by the folks at OpenAI"><carbon:checkmark class="text-green-400" /></Card>
<Card v-after title="Built in Python"><carbon:close class="text-red-600" /></Card>
</v-clicks>
</section>

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

# Two Choices

<section class="grid grid-cols-2 gap-4">
  <v-clicks>
  <Card title="Use the OpenAI Agents SDK"/>
  <Card title="Build our own"/>
  <Card description="The framework is established is maintained by a team larger than ours"/>
  <Card description="We might fall short of features, will have to build a lot of expertise internally"/>
  <Card description="Since it's in Python, we will have to build infrastructure to pipe data from our Rails App"/>
  <Card description="Can we natively integrated in our Rails App"/>
  <Card description="Will be another service that we, and our OSS community will have to manage"/>
  <Card description="Easy to maintain and run in the long run"/>
  </v-clicks>
</section>

---

<img class="shadow-lg outline outline-1 outline-gray-200 rounded-md overflow-hidden h-full mx-auto -mt-2" src="/ai-agents.png"/>
<div class="text-xs mx-auto text-center mt-4">ai-agents.chatwoot.dev</div>

---

# Agents SDK

<v-clicks>

- Built using RubyLLM
- Multi-Agent Orchestration
- Seamless Handoffs
- Tool Integration
- Callbacks
- Shared Context
- Thread-Safe Architecture
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

# Key Concepts

-   **Agent**: An AI assistant with a specific role, instructions, and tools. Each agent can register handoffs to other agents.
-   **Tool**: A custom function that an agent can use to perform actions (e.g., look up customer data, send an email). Tools are thread-safe and receive context as parameters.
-   **Handoff**: The process of transferring a conversation from one agent to another. This happens seamlessly without the user knowing.
-   **AgentRunner**: The thread-safe public API for managing multi-agent conversations. Created via `Runner.with_agents(...)`.
-   **Runner**: Internal execution engine that handles individual conversation turns. Each call to `AgentRunner.run` creates a new Runner instance.
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
class: text-center
layout: cover
---

# Demo Time

---

# Best Practices

- Make sure the scope of sub agents don't overlap
- Use different models for different use cases
- The best tool call is the one that is not executed
- Agents Error Compound
- Cost increases quadratically

---
class: text-center
---

# That's all folks

<div class="grid grid-cols-4 gap-4 pt-12 mb-20">

<div class="flex justify-center items-center h-full flex-col">
  <img class="w-44" src="/chatwoot-com.svg">
  chatwoot.com
</div>

<div class="flex justify-center items-center h-full flex-col">
  <img class="w-44" src="/github-chatwoot.svg">
  github.com/chatwoot
</div>

<div class="flex justify-center items-center h-full flex-col">
  <img class="w-44" src="/shivam-dev.svg">
  shivam.dev
</div>

<div class="flex justify-center items-center h-full flex-col">
  <img class="w-44" src="/agents-rb.svg">
  ai-agents github
</div>

</div>

---
