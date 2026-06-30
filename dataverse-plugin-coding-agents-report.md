# Dataverse Plugin for Coding Agents — Deep Capability Report

*Prepared 2026-06-30. Scope: the Microsoft **Dataverse plugin / Dataverse Skills** for AI coding agents (`microsoft/Dataverse-skills`), the **Dataverse MCP server** it is built on, the tool surface, setup, security/guardrails, licensing, and status.*

> **What this is.** "The Dataverse plugin for coding agents" is Microsoft's open-source plugin (the **Dataverse Skills** project) that lets AI coding agents — **Claude Code, GitHub Copilot, Cursor, Codex** — build, query, deploy, and administer Microsoft Dataverse through natural language. It is distributed on the **Claude Marketplace** and the **GitHub Copilot (awesome-copilot)** marketplace, and it sits on top of the first-party **Dataverse MCP server**. ([Dataverse Plugin on the Claude Marketplace](https://devblogs.microsoft.com/powerplatform/dataverse-plugin-claude-marketplace/); [microsoft/Dataverse-skills](https://github.com/microsoft/Dataverse-skills))

---

## 1. The big picture: two layers

There are **two distinct things** people call "Dataverse for coding agents," and the plugin combines them:

| Layer | What it is | Status |
|---|---|---|
| **Dataverse MCP server** | First-party Model Context Protocol endpoint exposed by Dataverse itself. Any MCP client (Copilot Studio, VS Code Copilot, Claude, Cursor) can connect and call a fixed set of **tools** (describe schema, query, CRUD, files). | **GA on 15 Dec 2025**; preview tools still ship on a separate endpoint |
| **Dataverse Skills / "the plugin"** | Open-source plugin (`microsoft/Dataverse-skills`, MIT) that wraps the MCP server **plus** the Dataverse CLI, Python SDK, and PAC CLI behind **8 specialist "skills."** This is what you install into Claude Code / Copilot. | Plugin is **available now** (MIT, open source); the Dataverse CLI it uses is in **preview** |

The key insight: **the MCP server is the protocol/data plane; the plugin is the intelligence layer** that decides *which* of four toolchains (MCP, CLI, SDK, PAC CLI) to use for a given natural-language request, and adds guardrails, routing, and setup automation on top. ([Dataverse Is Your Agent Data Platform](https://www.microsoft.com/en-us/power-platform/blog/2026/05/05/dataverse-agent-data-platform/))

---

## 2. The plugin: 8 specialist skills

The plugin packages **eight skills** (per the repo README). Crucially, **you never invoke a skill directly** — you describe intent ("create a recruiting system with five tables and sample data") and the agent loads the right skills in the right order. ([microsoft/Dataverse-skills README](https://github.com/microsoft/Dataverse-skills); [Dataverse Skills: Your Coding Agent Now Speaks Dataverse](https://devblogs.microsoft.com/powerplatform/dataverse-skills-your-coding-agent-now-speaks-dataverse/))

| Skill | Purpose |
|---|---|
| **`dv-overview`** | Cross-cutting router; loaded **first** to direct each request to the right specialist skill and enforce shared rules |
| **`dv-connect`** | One-time setup: installs the CLI/SDK/PAC CLI, authenticates against your environment, registers the MCP server with your agent |
| **`dv-query`** | Reads and filters records; handles pagination; supports DataFrame analysis ("show open deals over $100K") |
| **`dv-data`** | CRUD, bulk imports, CSV loading, upsert, multi-table imports with foreign-key dependencies |
| **`dv-metadata`** | Designs the data model: tables, columns, relationships, forms, and views |
| **`dv-solution`** | Solution lifecycle: create, export, import, promote across environments, validate deployments |
| **`dv-admin`** | Environment administration: bulk delete, retention/archival, org & OrgDB settings, recycle bin, audit, and allowlisted PPAC toggles |
| **`dv-security`** | Security roles, user access, application users, business units, admin self-elevation |

> Reconciliation note: a couple of secondary write-ups list only six skills (omitting `dv-query` and `dv-data`); the **repo README is authoritative at eight**. The `dv-admin` and `dv-security` capabilities reached **Public Preview** as "Dataverse Admin Skills" in May 2026. ([Agentic Administration: Dataverse Admin Skills in Public Preview](https://www.microsoft.com/en-us/power-platform/blog/2026/05/12/dataverse-agentic-administration/))

### The four underlying toolchains the skills orchestrate
1. **Dataverse MCP server** — ad-hoc discovery and natural-language/structured queries.
2. **Dataverse CLI** *(preview)* — rich data-plane actions.
3. **Python SDK** — batch and scripted operations, DataFrame analysis.
4. **PAC CLI** — admin gestures (solution export/import, environment management).

([Dataverse Plugin for Coding Agents: Three Enterprise Scenarios from Build 2026](https://www.microsoft.com/en-us/power-platform/blog/2026/06/04/microsoft-dataverse-plugin-unleashing-coding-agents-on-the-enterprise-microsoft-build-2026/))

---

## 3. The Dataverse MCP server — tool surface

The MCP server exposes a **fixed, named set of tools** ("the tool shape") — the contract the agent reasons about, and what can be allowed/blocked/audited. Microsoft revised this contract in 2026 ("the new tool shape"). ([Dataverse MCP Server: Understanding the New Tool Shape](https://www.microsoft.com/en-us/power-platform/blog/2026/06/08/dataverse-mcp-server-understanding-the-new-tool-shape/); [Dataverse MCP Server reference](https://learn.microsoft.com/en-us/microsoft-agent-365/mcp-server-reference/dataverse))

### Current tools (new tool shape)

**Schema discovery**
- **`describe`** — inspect a table's schema (columns, types, relationships), returned as T-SQL-style schema. *Replaces the older `list_tables` + `describe_table`.*
- **`search`** — search **metadata** to locate the right tables/columns (note: this searches the data model, not row data).

**Read / query**
- **`read_query`** — run a structured query and return rows (the structured-query path).
- *(broader semantic/data search where the scenario calls for it.)*

**Write / records**
- **`create_record`** — insert a row; returns the new record's GUID for downstream linking.
- **`update_record`** — modify an existing row.
- **`delete_record`** — remove a row (gated by approvals/safeguards).

**Schema authoring**
- **`create_table`** / **`update_table`** / **`delete_table`** — scaffold or evolve simple schemas.

**Files**
- **`init_file_upload`** / **`commit_file_upload`** / **`file_download`** — move files in/out of Dataverse.

> **Migration note:** the legacy tools `list_tables`, `describe_table`, and `fetch` were **removed/replaced** by the consolidated `describe`/`search`/`read_query` tools. If you have older configs or docs referencing those names, update them.

### GA vs preview endpoints
- **`/api/mcp`** — the **generally available** tool set.
- **`/api/mcp_preview`** — adds **preview tools** under evaluation before GA. ([Enable preview tools and features in Dataverse MCP server](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-preview-tools))

---

## 4. Installation & setup

### Install the plugin (one line)
- **Claude Code:** `/plugin install dataverse@claude-plugins-official`
- **GitHub Copilot (VS Code):** `install dataverse@awesome-copilot`
- **Cursor / Codex:** supported via their marketplace sources.

([Dataverse Plugin on the Claude Marketplace](https://devblogs.microsoft.com/powerplatform/dataverse-plugin-claude-marketplace/))

### Connect (the `dv-connect` skill does the heavy lifting)
You literally say *"connect to Dataverse"* and the skill: ([Getting Started with Dataverse Skills](https://medium.com/@zsolt.zombik/getting-started-with-dataverse-skills-install-configure-and-build-your-first-agent-driven-0f9f8092f563))
1. Installs the toolchain — **PAC CLI + .NET SDK**, **Node.js + Dataverse CLI**, **Python + Dataverse SDK**.
2. **Authenticates** against your environment.
3. Copies an **auth helper (`auth.py`)** and an **MCP enablement script (`enable-mcp-client.py`)**.
4. Writes a **`.env`** with your environment URL, tenant ID, and the correct **MCP client ID** for your agent.
5. **Verifies** auth by running `pac org who` and the Python auth script; loops back to diagnose if either fails.
6. **Registers the MCP server** with your agent.

Credentials are stored in the **OS-native credential store** only.

### Prerequisites
- A **Microsoft Dataverse environment** — Power Apps, Dynamics 365, or the **free Developer Plan**. ([microsoft/Dataverse-skills README](https://github.com/microsoft/Dataverse-skills))
- **Enable the MCP server in PPAC:** Power Platform Admin Center → *Dataverse Model Context Protocol* → turn on **"Allow MCP clients to interact with Dataverse MCP server."** ([Connect to Dataverse with MCP](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp); [Configure the Dataverse MCP server](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-disable))
- Know your **org URL** and **tenant ID**.
- **For client-credential auth:** create an **Application User** (PPAC → Users + Permissions → Application Users), link it to your App Registration, and assign a security role — otherwise Dataverse rejects every request. ([Connect to Dataverse MCP in non-Microsoft clients](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-other-clients))

---

## 5. Security & governance model

This is the differentiator vs. a raw "let the LLM hit the API" approach — it solves the hallucination/over-reach problem with layered guardrails: ([Microsoft Dataverse Agent Plugin: Safe Natural-Language Coding](https://windowsnews.ai/article/microsoft-dataverse-agent-plugin-safe-natural-language-coding-for-enterprise-ops.422785); [Dataverse Plugin: Three Enterprise Scenarios](https://www.microsoft.com/en-us/power-platform/blog/2026/06/04/microsoft-dataverse-plugin-unleashing-coding-agents-on-the-enterprise-microsoft-build-2026/))

- **Least privilege / identity bound** — the agent **cannot exceed the authenticated user's Dataverse permissions**; server-side security-role enforcement applies to every call.
- **Confirmation prompts before destructive operations** — and the agent **confirms the user's role** before offering to do anything destructive.
- **Validation safeguards** — FetchXML validation and **system-table warnings**.
- **Admin allowlist** — a **37-toggle PPAC allowlist** bounds what `dv-admin` can touch.
- **Audit** — all actions land in **Dataverse audit history** (supports GDPR/HIPAA-style compliance).
- **Telemetry is application-level only** — outbound requests may carry plugin/version/skill/agent labels for server-side dashboards, but **no prompts, tool arguments, or record data** are transmitted externally.
- **Multi-environment parallel execution** — with confirmation flows for cross-environment promotion.

---

## 6. Licensing & cost

| Item | Detail |
|---|---|
| **The plugin itself** | Open source, **MIT license** — free to install/use ([repo](https://github.com/microsoft/Dataverse-skills)) |
| **Dataverse environment** | Standard Dataverse licensing; a **free Developer Plan** environment works for trial/dev |
| **MCP server GA** | **Generally available 15 Dec 2025**, with new governance + billing ([MCP server GA notice](https://m365admin.handsontek.net/dataverse-dataverse-model-context-protocol-mcp-server-general-availability/)) |
| **MCP tool-call billing** | **From 15 Dec 2025, Dataverse MCP tools are charged when accessed by AI agents created *outside* Microsoft Copilot Studio** ([Licensing requirements](https://licensing.guide/dataverse-mcp-server-licensing-requirements/)) |
| **What licenses cover** | Access to **non-Dynamics 365 data** is included with the **Microsoft 365 Copilot User SL**; access to **Dynamics 365 data** is included with a **Dynamics 365 Premium** license; **other applicable licenses are charged per tool call** |
| **Auth requirement** | Agents built outside Copilot Studio require **User Subscription Licenses (User SLs)** to authenticate and call the MCP server/tools |

> Practical implication: a coding agent (Claude Code, VS Code Copilot) is "outside Copilot Studio," so **expect per-tool-call charges** for Dataverse MCP usage from a coding agent unless covered by the M365 Copilot / D365 Premium entitlements above. Budget and govern accordingly.

---

## 7. What you can actually do — capability summary

- **Stand up a data model from a sentence** — "Build a roast-batch tracking system" → multiple tables with choice columns, lookups, self-referential and many-to-many relationships, a main form, a filtered view, all packaged into a **solution**. (Build 2026 "Maya" scenario.)
- **Run your CRM in natural language** — "Friday pipeline prep" without Advanced Find/Excel pivots: query, filter, summarize, and update records conversationally. (Build 2026 "Riya" scenario.)
- **Bulk data operations** — multi-table CSV imports honoring FK dependencies, upserts, batch updates via the Python SDK.
- **Solution ALM** — create/export/import/promote solutions across environments and validate deployments.
- **Agentic administration** — bulk delete, retention/archival, org settings, recycle bin, audit, security-role assignment, business-unit config — all within the allowlisted, confirmation-gated guardrails.
- **Schema inspection + code generation in-IDE** — no portal-hopping; discovery, CRUD, and codegen happen inside the editor. ([Dataverse MCP: A Game Changer](https://www.microsoft.com/en-us/power-platform/blog/2025/07/07/dataverse-mcp/))

Early beta customers in financial services and healthcare reported a **~40% reduction** in time for routine data-modeling tasks (Microsoft's figure; treat as vendor-reported).

---

## 8. Status, limitations & gotchas

| Item | Status |
|---|---|
| Dataverse MCP server (`/api/mcp`) | **GA** (15 Dec 2025) |
| Dataverse MCP preview tools (`/api/mcp_preview`) | **Preview** |
| Dataverse plugin / Skills | **Available now** (MIT, open source) |
| Dataverse CLI (used by the plugin) | **Preview** |
| `dv-admin` / `dv-security` skills | **Public Preview** (May 2026) |

**Gotchas to design around:**
1. **Per-tool-call billing** for coding-agent (non-Copilot-Studio) MCP usage since 15 Dec 2025 — know which entitlement covers you.
2. **Tool-shape churn** — legacy `list_tables`/`describe_table`/`fetch` are gone; use `describe`/`search`/`read_query`.
3. **MCP must be explicitly enabled** in PPAC; it is **not on by default**, and admins control the allowed clients.
4. **App User required** for client-credential auth or every request is rejected.
5. **Agent acts as you** — capabilities are bounded by the signed-in user's security roles; over-broad roles = over-broad agent.
6. **`search` searches metadata, not data** — use `read_query` for row data; don't confuse the two.
7. **CLI/admin skills are preview** — not for unguarded production automation yet.

---

## 9. Recommendations

- **To try it:** spin up a **free Developer Plan** environment, enable MCP in PPAC, then `/plugin install dataverse@claude-plugins-official` (Claude Code) or `install dataverse@awesome-copilot` (VS Code Copilot), and say *"connect to Dataverse."*
- **For data modeling / prototyping:** lean on `dv-metadata` + `dv-solution` — this is the strongest, most mature use case.
- **For day-to-day data work:** `dv-query` / `dv-data` give you conversational CRUD and bulk import.
- **For admin/security:** treat `dv-admin`/`dv-security` as **preview** — use in non-prod, rely on the confirmation flows and PPAC allowlist, and keep the operator on a least-privilege role.
- **Governance first:** scope the **Application User / security role** tightly, enable **Dataverse auditing**, and **model the per-tool-call cost** before rolling out to a team.

---

## Sources
- [Dataverse Plugin Is Now on the Claude Marketplace — Power Platform Developer Blog](https://devblogs.microsoft.com/powerplatform/dataverse-plugin-claude-marketplace/)
- [microsoft/Dataverse-skills — GitHub](https://github.com/microsoft/Dataverse-skills)
- [Dataverse Skills: Your Coding Agent Now Speaks Dataverse — Developer Blog](https://devblogs.microsoft.com/powerplatform/dataverse-skills-your-coding-agent-now-speaks-dataverse/)
- [Dataverse Plugin for Coding Agents: Three Enterprise Scenarios from Build 2026 — Power Platform Blog](https://www.microsoft.com/en-us/power-platform/blog/2026/06/04/microsoft-dataverse-plugin-unleashing-coding-agents-on-the-enterprise-microsoft-build-2026/)
- [Dataverse Is Your Agent Data Platform: Here's What's New — Power Platform Blog](https://www.microsoft.com/en-us/power-platform/blog/2026/05/05/dataverse-agent-data-platform/)
- [Agentic Administration: Dataverse Admin Skills now in Public Preview — Power Platform Blog](https://www.microsoft.com/en-us/power-platform/blog/2026/05/12/dataverse-agentic-administration/)
- [Connect to Dataverse with model context protocol (MCP) — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp)
- [Connect Dataverse MCP with GitHub Copilot in VS Code and Copilot CLI — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-vscode)
- [Connect to Dataverse with MCP in non-Microsoft clients — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-other-clients)
- [Configure the Dataverse MCP server — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-disable)
- [Enable preview tools and features in Dataverse MCP server — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-preview-tools)
- [Connect to Dataverse with MCP FAQ — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/data-platform/data-platform-mcp-faq)
- [Dataverse MCP Server reference (preview) — Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-agent-365/mcp-server-reference/dataverse)
- [Dataverse MCP Server: Understanding the New Tool Shape — Power Platform Blog](https://www.microsoft.com/en-us/power-platform/blog/2026/06/08/dataverse-mcp-server-understanding-the-new-tool-shape/)
- [Dataverse MCP Server: A Game Changer for AI-Driven Workflows — Power Platform Blog](https://www.microsoft.com/en-us/power-platform/blog/2025/07/07/dataverse-mcp/)
- [Dataverse MCP Server general availability — M365 Admin](https://m365admin.handsontek.net/dataverse-dataverse-model-context-protocol-mcp-server-general-availability/)
- [Dataverse MCP Server licensing requirements — The Licensing Guide](https://licensing.guide/dataverse-mcp-server-licensing-requirements/)
- [Microsoft Dataverse Agent Plugin: Safe Natural-Language Coding for Enterprise Ops — Windows News](https://windowsnews.ai/article/microsoft-dataverse-agent-plugin-safe-natural-language-coding-for-enterprise-ops.422785)
- [Getting Started with Dataverse Skills — Zsolt Zombik / Medium](https://medium.com/@zsolt.zombik/getting-started-with-dataverse-skills-install-configure-and-build-your-first-agent-driven-0f9f8092f563)
- [How to Connect Dataverse MCP Server to Claude Code — Rajeev Pentyala](https://rajeevpentyala.com/2026/03/05/how-to-connect-dataverse-mcp-server-to-claude-code/)
