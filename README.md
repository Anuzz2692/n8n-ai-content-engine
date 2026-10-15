# n8n AI Content Engine

An AI content generation workflow built in n8n. A request form feeds an orchestrator agent, a content generation agent, separate medical and brand validator agents, a refinement loop, and a human review step.

**Status:** MVP / work in progress. A local version using Ollama with Qwen2.5-Coder-7B is being built separately and is not part of this repo yet.

## How it works

1. **Request form** collects: content type, topic, target audience, platform, campaign objective, language, tone, additional instructions, and optional reference content.
2. **Orchestrator agent** analyses the request and returns structured output (which specialist to use, and whether medical validation, brand validation and human review are required).
3. **Switch node** routes the request to the content generation agent.
4. **Content Generation Agent** writes the content for the chosen platform and type. Blog and website content follows a long-form structure (introduction, table of contents, main content, FAQs, People Also Ask, conclusion). If reference content is supplied, it is used as a structural blueprint, not copied.
5. **Medical Validator** and **Brand Validator** each return structured JSON: `passed`, `score` (0 to 1), `issues` and `critical_issues`.
6. **Quality decision** approves the content only if both scores are at least 0.9 and no issues are reported. Otherwise the content goes to rework.
7. **Refinement loop**: a refinement agent fixes the reported issues, then both validators check it again. The loop stops after 3 refinement passes and sends the content to human review.
8. **Human review form** (n8n Wait node) shows the content and both validation results, with the options `APPROVE`, `REQUEST_REWORK` or `REJECT` and a comments field.
9. **Final response** is cleaned into plain text with the decision and reviewer comments.

## Tech stack

- n8n (form trigger, Switch, Set, Code, Wait, and AI agent nodes)
- OpenAI `gpt-5-mini` through n8n's chat model nodes
- Structured output parsers for validated JSON results
- JavaScript in n8n Code nodes for the decision and loop logic

## Run it

1. Install n8n (self-hosted or cloud).
2. Create a new workflow and import `workflow.json` (menu, then Import from file).
3. On each OpenAI Chat Model node, select your own OpenAI credentials.
4. Open the form trigger, copy its test URL, submit a request and follow the run in the editor.

The prompts use a placeholder organisation name ("Example Fertility Clinic"). Replace it with your own.

## Current limitations and next steps

- Only the `social_agent` path is implemented. The orchestrator is told not to select website, campaign or SEO agents because they do not exist yet.
- `REQUEST_REWORK` in the human review form records the decision but does not re-run the workflow.
- Planned: a local version with Ollama (Qwen2.5-Coder-7B), and ChromaDB for retrieval-based generation (RAG).

## Disclaimer

This is a learning and prototyping project. AI-generated health-related content must be reviewed by a qualified person before use. Nothing here is medical advice.
