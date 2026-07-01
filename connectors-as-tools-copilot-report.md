# The "Connector Namespace": Connectors-as-Tools & Code-First Connectors as a Substitute for Power Automate + Azure Functions in Copilot Solutions

*Prepared 2026-06-30. Scope: the newer connector capabilities that let you wire integrations **directly into Copilot Studio agents** and push logic **into the connector itself** — reducing or removing the classic "Power Automate cloud flow → Azure Function → Copilot" middle tier.*

> **Framing note (important).** "Connector namespace" is not a single official Microsoft product name. Based on your description — *new, could substitute building Power Automate with Azure Functions, and tie into Copilot solutions* — it maps onto a **cluster of converging 2025–2026 capabilities**:
> 1. **Connectors-as-tools in Copilot Studio** (add a connector action straight to an agent — no flow needed).
> 2. **Code-first / enhanced connectors** (put logic *inside* the connector via C# custom code or the **Power Platform Connector SDK + Power Fx**) — the piece that can replace an Azure Function.
> 3. **Power Fx connector namespaces** (connector actions surface as namespaced Power Fx functions).
> This report covers all three and how they combine, plus the governance layer (Advanced Connector Policies) you'll hit in production.

---

## 1. The pattern you're replacing

**Classic pattern (3 hops):**
```
Copilot Studio agent  →  Power Automate cloud flow  →  Azure Function (custom logic/transform)  →  external API/data
```
You built a flow to orchestrate, an Azure Function to hold the code the flow couldn't express, and wired the flow into the agent as an action.

**New pattern (1 hop):**
```
Copilot Studio agent  →  connector action (with logic inside the connector)  →  external API/data
```
The agent calls a **connector tool** directly; any transform/logic lives in the connector (custom code or an enhanced-connector Web API) instead of a separate Function. ([Use connectors in Copilot Studio agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-connectors); [Add tools to custom agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/add-tools-custom-agent))

This is the substitution you're asking about. Below, each layer.

---

## 2. Layer 1 — Connectors as **tools** in Copilot Studio (removes the flow)

In current Copilot Studio, an agent's capabilities are **"tools,"** and connectors are one tool type you add **directly** — no Power Automate flow in between. ([Add tools to custom agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/add-tools-custom-agent))

### Tool types available to an agent
| Tool type | What it is | Replaces |
|---|---|---|
| **Connector (Power Platform connector) action** | A specific action from a prebuilt/standard/premium/custom connector, added straight to the agent | A flow that only wrapped one connector call |
| **REST API tool** *(preview)* | Point the agent at an **OpenAPI** spec (JSON v2; v3 auto-converted); the agent calls endpoints directly | A custom connector *and* the flow around it, for simple API calls |
| **Custom connector** | Wrap any public API (incl. one fronting an Azure Function) as a reusable connector, then add as a tool | Hand-rolled flow + connection plumbing |
| **Agent flow** | Deterministic, rule-based logic authored in **Power Fx** (the successor framing of Power Automate inside agents) | A cloud flow — when you still need real orchestration |
| **MCP server** | Real-time tool/data access via Model Context Protocol (e.g., Dataverse MCP) | Bespoke integration code |
| **Prompt / AI Builder prompt** | A reusable generative prompt as a tool | — |

Key mechanics:
- Each tool has a **natural-language description** the **orchestrator** uses to decide *when* to call it and to **generate follow-up questions to collect inputs**. This is what lets you skip explicit flow authoring — the LLM drives parameter collection. ([Extend your agent with tools from a REST API](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agent-extend-action-rest-api))
- Adding a **custom connector** tool takes you into the Power Apps *Custom connectors* portal to define it (or reuse an existing one). ([Use connectors in Copilot Studio agents](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-connectors))
- **Flows still exist and still matter** — when you need multi-step, deterministic orchestration, add an **agent flow** (Power Fx) or an existing Power Automate flow as a tool. Connectors-as-tools *reduce* the need for a flow; they don't abolish it. ([Agent flows overview](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flows-overview); [Add an agent flow or workflow as a tool](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flow-agent))

---

## 3. Layer 2 — Put logic **inside the connector** (removes the Azure Function)

The Azure Function usually existed to hold **code the flow couldn't express** (payload transforms, auth dance, shaping). Two features now let that code live in the connector itself.

### 3.1 Custom connector **custom code** (C#) — for transforms
A custom connector can run **C# custom code** that transforms request/response payloads beyond what the codeless policy templates allow. ([Write code in a custom connector](https://learn.microsoft.com/en-us/connectors/custom-connectors/write-code))
- Implement a class named **`Script`** deriving from **`ScriptBase`**, with an **`ExecuteAsync`** method invoked at runtime.
- **When code is present, it takes precedence** over the codeless definition.
- **Hard limits:** execution must finish within **5 seconds**; script file **≤ 1 MB**.
- **Substitution verdict:** replaces an Azure Function **when** the logic is a bounded, synchronous transform. It is *not* a general compute host — long-running, heavy, or multi-service orchestration still belongs in a Function/Logic App.

### 3.2 **Power Platform Connector SDK + Power Fx** — "enhanced connectors" for structured data
This is the more strategic, genuinely new piece (GA **October 3, 2025**). ([Build enhanced connectors with the Power Platform Connector SDK and PowerFx](https://learn.microsoft.com/en-us/power-platform/release-plan/2025wave1/microsoft-copilot-studio/build-enhanced-connectors-power-platform-connector-sdk-powerfx); [Create enhanced data connectors](https://learn.microsoft.com/en-us/connectors/custom-connectors/enhanced-connectors))

- You build a **Web API** (ASP.NET Core) that exposes **structured/tabular data** following the **enhanced connector protocol**, then **register it as a connector** in any Power Platform environment. Registration **auto-enables the enhanced maker experience**.
- The protocol is implemented as a small set of **endpoints** describing your service's capabilities — metadata, dataset enumeration, table listing, table schema, and **row retrieval with OData-style `$filter` / `$select` / `$top` / `$orderby`** (operators `eq, ne, gt, ge, lt, le`). ([microsoft/power-fx-enhanced-connector sample](https://github.com/microsoft/power-fx-enhanced-connector))
- The sample reuses Power Fx's **`RecordType`** to describe schema (no hand-built schema), and you override an **`ITableProvider`**-style interface to plug in your datasource.
- **Payoff:** the connector's data becomes **first-class in Power Fx** (delegable, filterable) *and* usable as a **knowledge source / tool in Copilot Studio agents** — so an agent can reason over your service's data without a flow or a Function shaping it. This is the clearest "sub Power Automate + Azure Functions, tie into Copilot" story.

---

## 4. Layer 3 — Power Fx **connector namespaces** (the literal "namespace")

This is likely where the word *namespace* in your question comes from. In Power Fx, **REST endpoints are imported as functions into a namespace** — i.e., each connector is exposed as a **namespace whose actions are functions** (e.g., `MyConnector.DoThing(args)`). ([Microsoft.PowerFx.Connectors discussion](https://github.com/microsoft/Power-Fx/discussions/537); [Power Fx namespaces](https://learn.microsoft.com/en-us/power-platform/test-engine/powerfx-namespaces))
- Namespaces separate **core language functions** from **extension-specific actions**, and can be **allow/block-listed** in configuration.
- Practically: with enhanced connectors + Power Fx, calling an integration becomes **writing a Power Fx expression against a connector namespace** rather than building a flow — the connector actions are just functions in scope.
- The same connector namespace/actions are what Copilot Studio surfaces as **tools**, so the agent and the low-code maker are pointing at the **same connector contract**.

> Caveat: "connector namespace" as a formal, singular feature name isn't in Microsoft's docs — it's the Power Fx mechanism by which connector actions are namespaced functions. If you saw the phrase in a specific blog/release note, send me the link and I'll pin the exact feature.

---

## 5. How it ties into Copilot solutions (end to end)

A modern, flow-light / Function-light Copilot solution:
1. **Data/logic** lives in an **enhanced connector** (Connector SDK + Power Fx) or a **custom connector with C# custom code**.
2. That connector is added to a **Copilot Studio agent as a tool** (or consumed in Power Fx via its **namespace**).
3. The agent's **orchestrator** decides when to call the tool and **collects inputs conversationally** from the connector action's descriptions/schema.
4. Only where you need **deterministic multi-step orchestration** do you add an **agent flow (Power Fx)** — otherwise no Power Automate.
5. Everything is packaged in a **solution** with **connection references** for ALM across environments. ([Use a connection reference in a solution](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference))

---

## 6. Governance you *will* hit: Advanced Connector Policies (ACP)

Because agents now reach the world through connectors/actions/MCP servers, governance moved to the **action level**. **Advanced Connector Policies (ACP)** went **GA on June 4, 2026**. ([Advanced connector policies are generally available](https://www.microsoft.com/en-us/power-platform/blog/2026/06/04/advanced-connector-policies-are-generally-available/); [ACP admin docs](https://learn.microsoft.com/en-us/power-platform/admin/advanced-connector-policies))

- **Default-deny allowlist** — replaces the old Business/Non-Business/Blocked model; **everything is blocked until allowed**, so a brand-new connector can't slip in "because it's new."
- **Action-level control** — govern **which actions** (not just which connectors) AI tools may use.
- **MCP server governance** — you can **block an entire MCP server** like any connector; **granular per-MCP-tool control isn't available yet**.
- **Design-time enforcement** — makers are told at authoring time whether a connector/action is allowed, not at runtime.
- **One policy per environment** — inherited from an environment group or set directly.
- **Connector inventory** (public preview since ~June 2, 2026) captures connector/operation usage across every app, flow, agent flow, and agent. ([What's new: June 2026](https://www.microsoft.com/en-us/power-platform/blog/2026/06/11/whats-new-in-power-platform-june-2026-feature-update/))

---

## 7. Licensing & cost considerations

- **Premium connectors / custom connectors** (incl. custom code and enhanced connectors) require **premium Power Platform licensing** (per-user/per-app or the relevant Power Apps/Automate/Copilot Studio plan). Custom connectors are a premium capability.
- **Copilot Studio** consumption is billed via **Copilot Studio messages / capacity** (prepaid packs or pay-as-you-go); tool/connector calls from an agent consume messages.
- **MCP tools** called by agents **outside Copilot Studio** are **charged per tool call from 15 Dec 2025** (relevant if you mix in Dataverse MCP).
- **No Azure Function bill** for logic you move into connector custom code / enhanced connectors — a real cost/ops simplification for bounded logic.

*(Confirm exact SKUs against the current Power Platform licensing guide before committing — licensing shifts frequently.)*

---

## 8. Status summary

| Capability | Status |
|---|---|
| Connectors as tools in Copilot Studio | **GA** |
| Agent flows (Power Fx) as tools | **GA** |
| REST API tool (OpenAPI) in Copilot Studio | **Preview** |
| Custom connector C# custom code | **GA** (long-standing) |
| Enhanced connectors (Connector SDK + Power Fx) | **GA — 3 Oct 2025** |
| Power Fx connector namespaces | **GA mechanism**; test-engine namespace controls in preview |
| Advanced Connector Policies (ACP) | **GA — 4 Jun 2026** |
| Connector inventory | **Public preview** (~Jun 2026) |

---

## 9. Substitution analysis — what you can actually drop

**You can remove the Azure Function when:**
- The logic is a **bounded, synchronous transform** (fits ≤5s / ≤1MB) → **custom connector C# code**.
- You're exposing **structured/tabular data** for makers/agents → **enhanced connector (SDK + Power Fx)**.

**You can remove the Power Automate flow when:**
- The agent just needs to **call one action** and collect inputs conversationally → **connector/REST API tool directly on the agent**.

**Keep a flow / Function when:**
- **Multi-step, stateful, or long-running orchestration**, retries, fan-out, durable/async, heavy compute, or logic exceeding the connector code limits → **agent flow (Power Fx)** for orchestration, **Azure Function/Logic App** for real compute.

**Net:** the "connector namespace" story collapses the common **"flow that wraps one call"** and the **"Function that does one transform"** into the **connector + agent-tool** layer — a genuine simplification for the majority of simple integrations, while true orchestration/compute still has its place.

---

## 10. Recommendations

- **Prototype the flow-light path:** build a **custom connector** (add C# custom code if you need a transform), then add it to a Copilot Studio agent **as a tool**. Measure whether the orchestrator's conversational input-collection removes your flow.
- **For data-heavy agents:** invest in an **enhanced connector** (Connector SDK + Power Fx) so the data is delegable in Power Fx *and* a knowledge/tool source in Copilot Studio.
- **Reserve agent flows** for genuine orchestration; don't rebuild everything as flows out of habit.
- **Plan governance first:** under **ACP default-deny**, explicitly **allowlist the connectors/actions** your agents need, and remember you can only block MCP servers **wholesale** today.
- **Watch the limits:** the 5s/1MB custom-code ceiling and premium licensing are the two things that push logic back to a Function/Logic App.

---

## Sources
- [Use connectors in Copilot Studio agents — Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-copilot-studio/advanced-connectors)
- [Add tools to custom agents — Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-copilot-studio/add-tools-custom-agent)
- [Extend your agent with tools from a REST API (preview) — Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-copilot-studio/agent-extend-action-rest-api)
- [Agent flows overview — Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flows-overview)
- [Add an agent flow or workflow as a tool to an agent — Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-copilot-studio/flow-agent)
- [Write code in a custom connector — Microsoft Learn](https://learn.microsoft.com/en-us/connectors/custom-connectors/write-code)
- [Build enhanced connectors with the Power Platform Connector SDK and PowerFx (2025 Wave 1) — Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/release-plan/2025wave1/microsoft-copilot-studio/build-enhanced-connectors-power-platform-connector-sdk-powerfx)
- [Create enhanced data connectors (preview) — Microsoft Learn](https://learn.microsoft.com/en-us/connectors/custom-connectors/enhanced-connectors)
- [microsoft/power-fx-enhanced-connector — GitHub sample](https://github.com/microsoft/power-fx-enhanced-connector)
- [Power Fx namespaces — Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/test-engine/powerfx-namespaces)
- [Microsoft.PowerFx.Connectors — Power-Fx discussion #537](https://github.com/microsoft/Power-Fx/discussions/537)
- [Use a connection reference in a solution — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/create-connection-reference)
- [Advanced connector policies are generally available — Power Platform Blog](https://www.microsoft.com/en-us/power-platform/blog/2026/06/04/advanced-connector-policies-are-generally-available/)
- [Advanced connector policies — Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/admin/advanced-connector-policies)
- [What's new in Power Platform: June 2026 feature update — Power Platform Blog](https://www.microsoft.com/en-us/power-platform/blog/2026/06/11/whats-new-in-power-platform-june-2026-feature-update/)
- [Create REST API actions for custom agents (2024 Wave 2) — Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/release-plan/2024wave2/microsoft-copilot-studio/create-rest-api-copilot-connectors-copilot-studio)
