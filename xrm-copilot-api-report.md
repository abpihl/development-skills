# Xrm.Copilot & Copilot Studio in Power Apps — Deep Capability Report

*Prepared 2026-06-30. Scope: the `Xrm.Copilot` client API, the PCF `context.copilot` API, and the maker-side options for embedding Copilot Studio agents into model-driven and canvas Power Apps.*

> **Headline caveat — almost everything here is in Public Preview.** The `Xrm.Copilot` namespace, the PCF Copilot APIs, the Agent Response form component, and the Microsoft 365 Copilot-in-apps experiences are all preview features under the Power Platform 2025 Wave 1 plan. APIs, signatures, and behaviors can change before GA, and they are not supported for production. Where I could not confirm an exact signature from primary docs (Microsoft Learn blocks automated fetching, so several details come from the Microsoft Learn GitHub source and reputable secondary write-ups), I flag it explicitly.

---

## 1. The big picture: three distinct "copilot" surfaces

Microsoft has overloaded the word "copilot," and the `Xrm.Copilot` namespace actually spans **two different AI systems**. Getting this distinction right is the key to the whole API:

| Surface | What it is | How you reach it |
|---|---|---|
| **Copilot Studio agents (a.k.a. "Agent APIs" / "Agent Xrm APIs")** | Custom agents/topics you build in **Microsoft Copilot Studio**, invoked programmatically and rendered however you like | `Xrm.Copilot.executePrompt`, `Xrm.Copilot.executeEvent`; PCF `context.copilot` equivalents; the low-code **Agent Response** form component |
| **Microsoft 365 Copilot in apps** | The first-party M365 Copilot chat experience hosted in the model-driven app side pane | `Xrm.Copilot.sendPromptToM365Copilot`, `openM365CopilotPanel`, `isM365CopilotEnabled`, `getCurrentAgent`, `updateContext`, and the action-handler methods |
| **Maker/no-code embedding** | Drag-and-drop and settings-based ways to put an agent into an app without JS | Agent Response component (MDA forms); App Copilot setting (canvas); the now-deprecated Copilot control (canvas) |

The same namespace serving both Copilot Studio *and* M365 Copilot is why the method list looks eclectic.

---

## 2. The `Xrm.Copilot` namespace — full method surface

Per the Microsoft Learn reference source, the `Xrm.Copilot` namespace exposes **11 methods** and **5 supporting interfaces**. ([Xrm.Copilot reference](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-copilot))

### Methods

**Copilot Studio (Agent) APIs**
1. **`executePrompt`** — Runs a Copilot Studio topic chosen by the orchestrator from a natural-language prompt; returns the agent's response.
2. **`executeEvent`** — Runs a specific Copilot Studio topic by its **registered event name**; returns a structured response.

**Microsoft 365 Copilot integration APIs**
3. **`sendPromptToM365Copilot`** — Sends a prompt programmatically into the M365 Copilot experience hosted in the app.
4. **`openM365CopilotPanel`** — Opens / makes visible the M365 Copilot side pane.
5. **`isM365CopilotEnabled`** — Tests whether M365 Copilot is available/enabled in the current context.
6. **`getCurrentAgent`** — Returns which agent is currently active.
7. **`updateContext`** — Pushes additional grounding/context signals from the app to Copilot.

**Action-handler (response-to-app wiring) APIs**
8. **`addActionHandler`** — Registers a custom handler that processes *actions* returned in a Copilot response (so an agent reply can trigger UI updates, workflows, or custom business logic in the app).
9. **`addDefaultActionHandlers`** — Registers the built-in/default set of action handlers.
10. **`removeActionHandler`** — Unregisters a specific custom action handler.
11. **`removeDefaultActionHandlers`** — Removes the default handler configuration.

### Supporting interfaces
- **`MCSResponse`** — the response shape returned by `executePrompt`/`executeEvent` (Microsoft Copilot Studio response).
- **`M365CopilotAgent`** — describes an M365 Copilot agent.
- **`M365CopilotAgentMode`** — the mode an M365 Copilot agent runs in.
- **`PowerAppsContent`** — content payload type used when passing app content to Copilot.
- **`SendPromptToM365CopilotOptions`** — options bag for `sendPromptToM365Copilot`.

> Source note: the method/interface inventory comes from the Microsoft Learn docs source on GitHub. Microsoft Learn's rendered reference pages confirm `executePrompt` and `executeEvent` in detail; the M365-Copilot and action-handler methods are newer and less thoroughly documented in public secondary sources, so treat their exact signatures as provisional.

---

## 3. The two core Copilot Studio APIs in depth

These are the APIs you'll actually build against today. They exist in **two parallel namespaces** with the same semantics:
- `Xrm.Copilot.*` — for **form scripts / ribbon commands** (classic client API). ([Bring intelligence using Agent Xrm APIs](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/bring-intelligence-using-agent-apis))
- `context.copilot.*` (PCF `PcfContext.Copilot`) — for **custom PCF controls**. ([PCF Agent APIs](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/bring-intelligence-using-agent-apis))

Both return a **rich JSON payload** (`MCSResponse[]`) that can carry **markdown, adaptive cards, images, and video**, giving you full control over rendering. ([Customize model-driven forms to leverage Copilot Studio content](https://www.microsoft.com/en-us/power-platform/blog/power-apps/customize-model-driven-forms-to-leverage-copilot-studio-content-preview/))

### 3.1 `executePrompt` — prompt-driven (orchestrator picks the topic)

Sends a natural-language prompt; **Copilot Studio's orchestration decides which topic to fire** based on the topics' trigger phrases. Use this for free-form/NL interactions. ([executePrompt reference](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-copilot/executeprompt))

**Signatures (two overloads):**
```javascript
// 1) simple prompt string
Xrm.Copilot.executePrompt(promptText).then(successCallback, errorCallback);

// 2) prompt + explicit record context
const parameters = {
  prompt: "Summarize this account and recent cases",
  context: {
    type: "record",
    entityName: formContext.data.entity.getEntityName(),
    recordId: formContext.data.entity.getId()
  }
};
const response = await Xrm.Copilot.executePrompt(parameters);
// response[0].text  -> the agent's reply
```

- **Parameters:** either a `promptText` string, or an object `{ prompt, context: { type: "record", entityName, recordId } }`.
- **Returns:** a **`Promise<MCSResponse[]>`** (array). For a prompt, the relevant element is a **message** activity.
- **App/page/record context** is also passed implicitly so the agent is grounded in where the user is.

**Representative response (message activity):**
```json
{
  "type": "message",
  "timestamp": "…",
  "replyToId": "…",
  "attachments": [ /* adaptive cards, media, etc. */ ],
  "textFormat": "markdown",
  "text": "Here's a summary of the account…",
  "speak": "Here's a summary of the account…"
}
```

### 3.2 `executeEvent` — event-driven (you name the topic)

Fires a **specific** Copilot Studio topic by a **registered event name**. Use this for deterministic, button-/form-triggered calls where you know exactly which topic should run. ([executeEvent reference](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-copilot/executeevent))

**Signature:**
```javascript
Xrm.Copilot.executeEvent(eventName, eventParameters).then(successCallback, errorCallback);

// example
const result = await Xrm.Copilot.executeEvent("onAccountReview", {
  priority: "high",
  region: "EMEA"
});
// result[0].value  -> structured data returned by the topic
```

- **Parameters:**
  - `eventName` *(string)* — the event registered on a Copilot Studio topic (set the topic's trigger to **"Activity received" / a registered event**, matching this name).
  - `eventParameters` *(object, optional)* — extra context. **Inside the topic these arrive as `Activity.Value`** (e.g. `Activity.Value.priority`).
- **Returns:** a **`Promise<MCSResponse[]>`** — for an event, an **event** activity.
- **Implicit context** (app, page, record) is passed automatically; `eventParameters` is *additional* context on top.

**Representative response (event activity):**
```json
{
  "type": "event",
  "timestamp": "…",
  "replyToId": "…",
  "attachments": [ … ],
  "value": { /* structured data your topic returns */ },
  "name": "onAccountReview"
}
```

### 3.3 Choosing between them
- **`executePrompt`** → conversational, NL, "let the agent figure out what to do." Topic chosen by trigger-phrase orchestration.
- **`executeEvent`** → programmatic, deterministic, "run *this* topic with *these* parameters." Topic chosen by event name; params land in `Activity.Value`.

### 3.4 Which agent answers?
The Agent APIs resolve to one of:
- an **app assistant agent** explicitly selected in the **model-app designer**, or
- implicitly, the **"Copilot in Dynamics 365 Sales"** agent in apps that contain the **lead or opportunity** tables. ([Bring intelligence using Agent Xrm APIs](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/bring-intelligence-using-agent-apis))

---

## 4. Embedding Copilot Studio into **model-driven apps**

You have a ladder from zero-code to full-code:

### 4.1 Agent Response component (no-code, form designer) — *easiest*
A drag-onto-the-form component that **calls a Copilot Studio topic and renders the reply directly on the form**. It is built on top of `executeEvent`. ([Customize model-driven forms blog](https://www.microsoft.com/en-us/power-platform/blog/power-apps/customize-model-driven-forms-to-leverage-copilot-studio-content-preview/); [Add agent response using the form designer](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/form-designer-add-configure-agent-response))

- **Renders** markdown, adaptive cards, image, or video returned by the topic.
- **Configuration:** in the *Add Agent Response* dialog, enter the topic's **Event name** in the *Static value* box.
- **Implicit context** (app, page, record) is available to the topic automatically.
- **Limitations:**
  - **No additional context** can be passed (unlike a custom PCF using `executeEvent`). If you need to pass parameters, build a custom component.
  - **No live preview** in the form designer — it shows *"Agent Response is only available when you play the app."*
  - Requires the environment setting **"Allow users to analyze data using an AI-powered chat experience…"** enabled.

### 4.2 Custom PCF control (pro-code) — *most flexible*
Build a PCF control that calls `context.copilot.executeEvent` / `executePrompt`, then render the `MCSResponse` however you want. This is the path when you need to **pass extra context**, do custom rendering, or chain logic. ([PCF Agent APIs](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/bring-intelligence-using-agent-apis); [Diana Birkelbach: PCF + Copilot Studio first look](https://dianabirkelbach.wordpress.com/2025/07/02/pcf-%F0%9F%A9%B7-copilot-studio-first-look-at-agent-apis/))

### 4.3 Form-script / ribbon (pro-code)
Call `Xrm.Copilot.executePrompt` / `executeEvent` from `onLoad`, `onChange`, or a ribbon button to generate **context-aware responses** and surface them via notifications, dialogs, or by writing to fields. ([Smarter Model-Driven Apps: context-aware responses](https://thedynamicpowerdisciple.com/smarter-model-driven-apps-use-copilot-to-generate-context-aware-responses/))

### 4.4 App assistant agent + Copilot chat (configuration)
Attach an **app assistant agent** to the app in the designer, exposing a Copilot chat pane scoped to the app. The Agent APIs then route to that agent. ([Add agents to your model-driven app](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/add-agents-to-app))

### 4.5 Microsoft 365 Copilot in the side pane (configuration + API)
Separate from Copilot Studio: surfaces **M365 Copilot** in the model-driven **side pane**, where users can pick or **@-mention agents**, run multiple agents in one conversation, and ask read-only questions over Dataverse data. Programmatically driven by `openM365CopilotPanel`, `sendPromptToM365Copilot`, `updateContext`, etc. ([Use Microsoft 365 Copilot in model-driven apps](https://learn.microsoft.com/en-us/power-apps/user/use-microsoft-365-copilot-model-driven-apps); [Add Microsoft 365 Copilot for app users](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/add-microsoft-365-copilot))

- **Read-only:** M365 Copilot in model-driven apps answers questions over Dataverse but **cannot modify data**.
- Pane can be **expanded/collapsed**; agents added/removed mid-conversation via @-mention.

---

## 5. Embedding Copilot Studio into **canvas apps**

> **⚠️ Major deprecation.** Starting **February 2, 2026**, you **cannot add** the preview **Copilot control**, **Copilot Answer control**, or **Custom Copilot** to new canvas apps. Existing apps keep working temporarily but are unsupported and slated for removal. Microsoft's direction is **Microsoft 365 Copilot in Power Apps** as the replacement. ([add-ai-copilot reference](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/add-ai-copilot); [MC1215683 deprecation notice](https://blog.tophhie.cloud/m365-message-center/message/mc1215683/); [Important changes coming in Power Platform](https://learn.microsoft.com/en-us/power-platform/important-changes-coming))

Current options:

### 5.1 App Copilot setting (no-code) — *recommended path for canvas*
Embed a custom Copilot Studio copilot across **all screens** without changing the design. ([Add a custom Copilot to a canvas app](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/add-custom-copilot))

**Steps:** Power Apps Studio → **Settings → Updates → Preview** → toggle **App Copilot** *On* → **Copilot** tab → under **Connect a copilot**, pick a custom copilot **published and shared in the same environment**.

### 5.2 PCF ChatControl (pro-code) — *advanced*
The official **Copilot Studio Samples** GitHub repo ships a **ChatControl PCF** that embeds a Copilot Studio agent in a canvas app using **Bot Framework WebChat** with a Fluent UI theme. Best for full control over the chat surface. ([Embed Copilot Studio Agents in Canvas Apps with a PCF Control](https://microsoft.github.io/mcscatblog/posts/embed-copilot-studio-agents-canvas-apps/))

### 5.3 Microsoft 365 Copilot in canvas apps (preview) — *strategic direction*
The forward-looking replacement for the deprecated controls. ([Use Microsoft 365 Copilot in canvas apps](https://learn.microsoft.com/en-us/power-apps/user/use-microsoft-365-copilot-canvas-apps))

---

## 6. Context passing (how the agent "knows where you are")

When any Agent API is called, **app, page, and record context** are passed to the Copilot Studio topic via a set of system variables, so the topic can reason about the current app/form/record without you wiring it up. On top of that:
- `executeEvent`'s `eventParameters` → available in-topic as **`Activity.Value`**.
- `executePrompt`'s `context: { type: "record", entityName, recordId }` grounds the prompt on a specific record.
- The no-code **Agent Response** component gets implicit context **only** (no custom params).
([Use Copilot Studio agents in model-driven apps — reference architecture](https://learn.microsoft.com/en-us/power-platform/architecture/reference-architectures/contextual-ai-model-driven-app))

---

## 7. Prerequisites, licensing & enablement

### 7.1 Environment / admin settings
- **Environment setting:** *"Allow users to analyze data using an AI-powered chat experience in canvas and model-driven apps"* (Power Platform admin center). Values: ([Add Copilot chat for app users](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/add-ai-copilot))
  - **Default** — disabled for a **Power Apps-licensed** environment; **enabled** for a **Dynamics 365-licensed** environment.
  - **On** — enabled regardless of licensing type.
  - **Off** — disabled regardless of licensing type.
- **Not on by default** for model-driven apps — an admin must enable it for the environment.
- **Per-app opt-out:** App designer → **Settings → Upcoming → Copilot control** = Default/Off.
- **Preview access:** as of mid-2025 the Agent APIs required an **early-release-cycle environment** and `make.preview.powerapps.com`.

### 7.2 Copilot Studio licensing
Running Copilot Studio agents consumes **Copilot Studio messages/capacity** (pay-as-you-go or prepaid message packs). Building/calling the agent therefore needs the underlying Copilot Studio entitlement in addition to the Power Apps/Dynamics 365 licensing for the host app. ([Copilot Studio licensing](https://learn.microsoft.com/en-us/microsoft-copilot-studio/billing-licensing); [Requirements & licensing](https://learn.microsoft.com/en-us/microsoft-copilot-studio/requirements-licensing-subscriptions))

### 7.3 M365 Copilot path
The M365-Copilot-in-apps experience is governed by Microsoft 365 Copilot licensing/admin controls rather than (or in addition to) the Power Platform settings above.

---

## 8. Status, versions & limitations summary

| Item | Status | Notes |
|---|---|---|
| `Xrm.Copilot.executePrompt` / `executeEvent` | **Preview** | 2025 Wave 1; client API + PCF. ([release plan](https://learn.microsoft.com/en-us/power-platform/release-plan/2025wave1/power-apps/bring-intelligence-into-model-driven-apps-custom-components-using-agent-xrm-pcf-apis)) |
| M365 Copilot methods (`sendPromptToM365Copilot`, `openM365CopilotPanel`, etc.) | **Preview** | Newer; least documented publicly. |
| Action-handler methods (`addActionHandler`, etc.) | **Preview** | Wire agent responses back to app logic. |
| Agent Response form component | **Preview** | No custom context; no design-time preview. |
| Canvas Copilot control / Answer control / Custom Copilot | **Deprecated** | No new apps after **Feb 2, 2026**; migrate to M365 Copilot in Power Apps. |
| M365 Copilot in model-driven / canvas apps | **Preview** | Read-only over Dataverse in MDA. |

**Key limitations to design around:**
1. **Preview everywhere** — no production SLA; expect signature churn.
2. **MDA M365 Copilot is read-only** over Dataverse.
3. **Agent Response component can't pass parameters** — use a PCF + `executeEvent` if you need context.
4. **No design-time preview** for the Agent Response component.
5. **Canvas copilot controls are end-of-life** — don't start new builds on them.
6. **Enablement is multi-layered** — environment setting + admin enablement + (for canvas) same-environment published/shared copilot + Copilot Studio capacity.

---

## 9. Practical recommendations

- **Model-driven, quick win, no code:** Agent Response component bound to a topic event name.
- **Model-driven, need context/custom UI:** PCF control calling `context.copilot.executeEvent` (deterministic) or `executePrompt` (NL).
- **Model-driven, form logic:** `Xrm.Copilot.executePrompt`/`executeEvent` from form scripts/ribbon, render via notifications or field writes; use the action-handler APIs to let agent replies drive app behavior.
- **Canvas:** use the **App Copilot** setting or the **ChatControl PCF**; **do not** build new apps on the deprecated Copilot control. Plan for **M365 Copilot in Power Apps**.
- **Everything:** budget for **Copilot Studio message consumption** and keep a close eye on the **2025 Wave 1 release notes** for GA timing and signature changes.

---

## Sources
- [Xrm.Copilot (Client API reference) — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-copilot)
- [executePrompt (Client API reference) — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-copilot/executeprompt)
- [executeEvent (Client API reference) — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/reference/xrm-copilot/executeevent)
- [Bring intelligence into your app using Agent Xrm APIs — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/developer/model-driven-apps/clientapi/bring-intelligence-using-agent-apis)
- [Bring intelligence into your components using Agent APIs (PCF) — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/bring-intelligence-using-agent-apis)
- [executePrompt / executeEvent (PCF API reference) — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/developer/component-framework/reference/copilot/executeprompt)
- [Release plan: Agent Xrm & PCF APIs (2025 Wave 1) — Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/release-plan/2025wave1/power-apps/bring-intelligence-into-model-driven-apps-custom-components-using-agent-xrm-pcf-apis)
- [Customize model-driven forms to leverage Copilot Studio content (Preview) — Power Platform Blog](https://www.microsoft.com/en-us/power-platform/blog/power-apps/customize-model-driven-forms-to-leverage-copilot-studio-content-preview/)
- [Add agent response using the form designer — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/form-designer-add-configure-agent-response)
- [Add agents to your model-driven app — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/add-agents-to-app)
- [Use Copilot Studio agents in model-driven apps (reference architecture) — Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/architecture/reference-architectures/contextual-ai-model-driven-app)
- [Add a custom Copilot to a canvas app — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/add-custom-copilot)
- [Add a Copilot control to a canvas app (preview) — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/canvas-apps/add-ai-copilot)
- [Embed Copilot Studio Agents in Canvas Apps with a PCF Control — MCS CAT blog](https://microsoft.github.io/mcscatblog/posts/embed-copilot-studio-agents-canvas-apps/)
- [Use Microsoft 365 Copilot in model-driven apps — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/user/use-microsoft-365-copilot-model-driven-apps)
- [Add Microsoft 365 Copilot for app users in model-driven apps — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/add-microsoft-365-copilot)
- [Add Copilot chat for app users in model-driven apps — Microsoft Learn](https://learn.microsoft.com/en-us/power-apps/maker/model-driven-apps/add-ai-copilot)
- [Copilot Studio licensing — Microsoft Learn](https://learn.microsoft.com/en-us/microsoft-copilot-studio/billing-licensing)
- [Important changes (deprecations) coming in Power Platform — Microsoft Learn](https://learn.microsoft.com/en-us/power-platform/important-changes-coming)
- [MC1215683: Deprecation of Preview Copilot Controls in Canvas Apps](https://blog.tophhie.cloud/m365-message-center/message/mc1215683/)
- [PCF 🩷 Copilot Studio: First Look at Agent APIs — Diana Birkelbach](https://dianabirkelbach.wordpress.com/2025/07/02/pcf-%F0%9F%A9%B7-copilot-studio-first-look-at-agent-apis/)
- [Smarter Model-Driven Apps: Use Copilot to Generate Context-Aware Responses — The Dynamic Power Disciple](https://thedynamicpowerdisciple.com/smarter-model-driven-apps-use-copilot-to-generate-context-aware-responses/)
- [Exploring the new Xrm.Copilot API client reference — CRM Crate](https://www.crmcrate.com/power-apps/exploring-the-new-xrm-copilot-api-client-reference-for-power-apps/)
- [Unlocking XRM.Copilot: APIs for Dynamics 365 — Dynamics Services Group](https://dynamicsservicesgroup.com/2025/09/05/unlocking-xrm-copilot-apis-for-dynamics-365/)
