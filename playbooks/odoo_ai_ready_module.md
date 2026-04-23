# Odoo AI Ready Module

Use this playbook only when AI readiness is explicitly requested or clearly in scope.

Do not add Odoo AI integration to every addon by default.
If a module exposes a strong candidate workflow but the user did not ask for AI, confirm once before adding it.

## What This Playbook Optimizes For

- Reusing the actual Odoo Enterprise AI contract instead of inventing a parallel abstraction
- Small, reviewable AI integrations that stay close to normal Odoo business logic
- Stable tool naming, bounded schemas, and predictable runtime behavior
- Clear separation between tools for acting and sources for answering

## Core Contract

The reusable Odoo AI integration contract is built around these building blocks:

- Odoo AI does not discover custom business actions through a generic model registry. Reusable integrations are exposed through `ir.actions.server` tools, grouped into `ai.topic`, then attached to an `ai.agent` or an AI server action.
- `ai.agent` is the runtime agent. It combines a system prompt, optional sources, and `topic_ids`. At runtime it exposes only the tools coming from the selected topics.
- `ai.topic` is the main reusable capability package. It bundles instructions plus `tool_ids`.
- `ir.actions.server` is the executable AI tool surface. AI tools are normal server actions with `use_in_ai=True`, `ai_tool_description`, and `ai_tool_schema`.
- Tool execution stays server-action based. The usual pattern is a thin wrapper that calls a model method and stores the result in `ai['result']`.
- Tool names shown to the LLM come from the XML ID suffix when available, otherwise `action_<id>`. Stable XML IDs matter.
- `__end_message` is a special synthetic parameter injected when `ai_tool_allow_end_message=True`. Use it for tools that should end the loop cleanly.
- AI record automation is a separate pattern. A parent server action with `state="ai"` uses `ai_action_prompt` plus `ai_tool_ids` and lets the LLM pick from a bounded tool set for a record flow.
- Prompt context is inserted with bounded HTML markers such as `data-ai-field` and `data-ai-record-id`, not by dumping uncontrolled free text.
- RAG is source-based through `ai.agent.source`. Sources help the model answer. Tools help it act.

## Choose The Pattern

### 1. AI Tools For Business Actions

Use when the agent should create, update, classify, or trigger a business flow.

Pattern:

- model methods such as `_ai_create_*`, `_ai_get_*_params`, `_ai_apply_*`
- `ir.actions.server` wrappers with `use_in_ai=True`
- `ai.topic` for instructions and grouping
- optional `ai.agent` binding where the behavior should be available

### 2. AI Record Automation

Use when AI should operate on a record from a prompt plus a tightly bounded tool list.

Pattern:

- parent `ir.actions.server` with `state="ai"`
- `ai_action_prompt`
- `ai_tool_ids`
- inserted record and field context

### 3. AI Fields Or AI Server Actions

Use only when the requirement is AI-computed values, not conversational business actions.

Pattern:

- `ai_fields`
- `ai_server_actions`

### 4. AI Sources

Use when the agent needs retrieval-backed answers from custom records, documents, or knowledge content.

Pattern:

- extend source handling only when the problem is answering from data, not acting on data

## Official Recipe 1: Domain AI Tools

Use this as the standard pattern for custom business actions.

1. Identify one narrow business action worth exposing.
2. Put the business logic in a model method such as `_ai_create_xxx()` or `_ai_apply_xxx()`.
3. Add an optional helper method such as `_ai_get_xxx_params()` when the agent may need bounded discovery first.
4. Wrap each method in an `ir.actions.server` with:
   - `use_in_ai=True`
   - `ai_tool_description`
   - `ai_tool_schema`
   - optional `ai_tool_allow_end_message=True`
5. Group related tools in an `ai.topic`.
6. Write topic instructions that define:
   - when the topic applies
   - when to ask for missing data
   - when to call each tool
   - what the tool may never do
7. Bind the topic only to the agents or entrypoints that should expose the behavior.
8. Add tests for both the model method and the AI tool wrapper path.

## Official Recipe 2: AI Record Automation

Use this as the standard pattern for workflow-style automation such as document processing.

1. Create a parent server action with `state="ai"`.
2. Set a strict `ai_action_prompt`.
3. Pass only the minimal allowed `ai_tool_ids`.
4. Insert bounded field and record context instead of vague free text.
5. Keep the tool set business-specific and minimal.
6. Validate both the successful path and refusal or boundary cases.

## Implementation Workflow

1. Confirm the addon should be AI-ready.
2. Identify the smallest useful business capability to expose.
3. Decide whether the requirement is a tool, record automation, AI-computed field, or source-backed answer.
4. Keep the main business logic in normal model methods and make the AI layer a thin wrapper.
5. Use stable XML IDs for every AI tool.
6. Write `ai_tool_description` around usage intent, not implementation detail.
7. Write a tight JSON schema:
   - use a root object
   - keep required fields explicit
   - use enums or bounded IDs when the option space is known
   - keep optional fields truly optional
8. Return short result strings from tools because the result is fed back into the LLM loop.
9. Do not return interactive client actions from AI tools.
10. Use `__end_message` only when a tool should finish the loop cleanly.
11. Keep access handling explicit. Use user rights by default and only `sudo()` in the underlying model method when the business flow justifies it.
12. Bind custom topics deliberately. Do not assume the global Ask AI agent will discover new tools automatically.

## Design Rules

- Prefer one clear tool per business action.
- Prefer deterministic tools over interactive or ambiguous tools.
- Prefer short, stable outputs over rich UI responses.
- Prefer bounded prompts with inserted values over free-text prompts with implied context.
- Prefer extending a specific agent surface over attaching everything to a generic agent.
- Prefer a topic with strict instructions over a broad bundle of weakly related tools.

## Enterprise Reference Map

These Enterprise addons are the best reference points for custom integrations:

- `ai_crm`: clean example of AI tools in XML that delegate to model methods
- `ai_crm`: strong example of topic instructions driving a business action flow
- `ai_crm_livechat`: shows the business-topic-on-agent composition pattern
- `ai_documents`: strongest example of record automation with a parent AI action and bounded tools
- `ai_documents_account`: shows extending an existing automation with extra allowed tools
- `ai_livechat`: shows a surface-specific agent adaptation
- `ai_fields`: use this for AI-computed field scenarios instead of tool-based actions
- `ai_server_actions`: use this when the goal is AI-generated values in server actions, not reusable agent tools
- `ai_documents_source` and `ai_knowledge`: use these as references for source-backed RAG extensions

## Framework Module Map

Use this as a quick mental model when deciding where a customization belongs:

- `ai`: base framework for agents, topics, tools, sources, embeddings, chat channels, and tool execution
- `ai_app`: management UI for agents, topics, and related backend configuration
- `ai_auto_install`: conditional installation layer around the AI framework
- `ai_fields`: AI-computed field and property path
- `ai_server_actions`: AI-generated field values inside server actions
- `ai_livechat`: livechat-specific agent surface and access adaptations
- `ai_crm_livechat`: example of attaching a business topic to a surface-specific agent
- `ai_documents`: record-level AI orchestration with prompt context and bounded tools
- `ai_documents_account`: extension of the Documents orchestration pattern
- `ai_documents_source` and `ai_knowledge`: additional source types for answer-oriented RAG flows
- `ai_website` and `ai_website_livechat`: same agent/topic/tool model exposed through website-specific entrypoints
- `ai_account` and `ai_knowledge`: good references when the requirement is drafting or composer integration rather than new tool contracts

## Python Skeleton

```python
from odoo import models


class ExampleModel(models.Model):
    _name = "x.example"
    _description = "Example"

    def _ai_create_example(self, name, category=None):
        self.ensure_one()
        record = self.env["x.example.record"].create({
            "name": name,
            "category": category or False,
        })
        return f"Created example {record.display_name}."

    def _ai_get_create_example_available_params(self):
        self.ensure_one()
        return "Allowed categories: retail, services, internal."
```

## AI Tool XML Skeleton

```xml
<odoo>
    <record id="ir_actions_server_ai_create_example" model="ir.actions.server">
        <field name="name">AI Create Example</field>
        <field name="model_id" ref="model_x_example"/>
        <field name="state">code</field>
        <field name="use_in_ai">True</field>
        <field name="ai_tool_description">
            Create an example record when the user explicitly asks for one and the required values are known.
        </field>
        <field name="ai_tool_schema"><![CDATA[
{
  "type": "object",
  "properties": {
    "name": {"type": "string"},
    "category": {
      "type": "string",
      "enum": ["retail", "services", "internal"]
    }
  },
  "required": ["name"]
}
        ]]></field>
        <field name="code"><![CDATA[
ai["result"] = records._ai_create_example(
    name=params["name"],
    category=params.get("category"),
)
        ]]></field>
    </record>
</odoo>
```

## Topic Skeleton

```xml
<odoo>
    <record id="ai_topic_example_tools" model="ai.topic">
        <field name="name">Example Business Actions</field>
        <field name="instructions">
            Use these tools only for explicit example-management requests.
            Ask for missing required inputs before creating records.
            Never invent categories outside the allowed schema.
        </field>
        <field name="tool_ids" eval="[(4, ref('your_module.ir_actions_server_ai_create_example'))]"/>
    </record>
</odoo>
```

## Agent Binding Skeleton

```xml
<odoo>
    <record id="ai_agent_example" model="ai.agent">
        <field name="name">Example Agent</field>
        <field name="system_prompt">Handle example-management requests only.</field>
        <field name="topic_ids" eval="[(4, ref('your_module.ai_topic_example_tools'))]"/>
    </record>
</odoo>
```

## Parent AI Action Skeleton

```xml
<odoo>
    <record id="ir_actions_server_ai_process_example" model="ir.actions.server">
        <field name="name">AI Process Example</field>
        <field name="model_id" ref="model_x_example_record"/>
        <field name="state">ai</field>
        <field name="ai_action_prompt">
            Review the current record context and use only the allowed tools to process it.
        </field>
        <field name="ai_tool_ids" eval="[
            (4, ref('your_module.ir_actions_server_ai_create_example'))
        ]"/>
    </record>
</odoo>
```

## Validation

- Test the underlying `_ai_*` model methods directly.
- Test `_ai_tool_run()` on the server action for schema and execution behavior.
- Test access-sensitive paths with realistic users when portal, public, or internal behavior differs.
- Test at least one successful path and one refusal or validation path.
- Verify the XML ID based tool names are stable across upgrades.
- Verify the tool result is a short plain string and not an interactive action payload.

## Guardrails

- AI readiness is opt-in.
- Do not bolt AI onto a module because it sounds modern.
- Do not assume the systray Ask AI agent will automatically expose custom tools.
- Do not return normal UI actions from AI tools.
- Do not hide core business logic inside large server-action code blocks.
- Do not use RAG sources as a substitute for tool-based business actions.
- Keep the patch small, migration-safe, and easy to review.
