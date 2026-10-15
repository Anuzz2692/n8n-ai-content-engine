🤖 n8n AI Content Engine

AI Content Engine — A multi-agent n8n workflow that turns a simple request form into validated, human-reviewed content: orchestration, generation, medical and brand validation, an automatic refinement loop, and a final human approval step.

Status: MVP / work in progress. A local version using Ollama with Qwen2.5-Coder-7B is being built separately and is not part of this repository yet.

📌 Overview

AI Content Engine is an n8n workflow that uses OpenAI gpt-5-mini through n8n's AI Agent nodes. A user fills in a form (content type, topic, audience, platform, language, tone and more), and the workflow does the rest: it decides how to process the request, writes the content, checks it with two separate validators, refines it if needed, and finally asks a human reviewer to approve, reject or request rework.

The prompts are written for healthcare content (fertility and reproductive health), where unsupported claims and guarantees must be avoided. The organisation name in the prompts is a placeholder ("Example Fertility Clinic").

🏗️ Architecture
User fills the request form (n8n Form Trigger)
        │
        ▼
┌─────────────────────────────────────────────────┐
│                 ORCHESTRATOR                    │
│     (AI Agent + Structured Output Parser)       │
│                                                 │
│  1. Orchestrator Agent  → Analyse the request   │
│  2. Switch              → Route to specialist   │
│  3. Content Generation Agent → Write content    │
│  4. Medical Validator   → Score + issues        │
│  5. Brand Validator     → Score + issues        │
│  6. Quality Decision    → Approve or rework     │
│  7. Refinement loop     → Fix, re-validate      │
│                           (up to 3 passes)      │
│  8. Human Review form   → Approve / Rework /    │
│                           Reject                │
└─────────────────────────────────────────────────┘
        │
        ▼
 Final plain-text response
 (decision | reviewer comments | final content)

The workflow is made up of these agents and steps:

Agent / Step	Responsibility
Orchestrator Agent	Analyses the request and returns structured JSON: which specialist to use, and whether medical validation, brand validation and human review are required
Content Generation Agent	Writes the requested content for the chosen platform, type, language and tone
Medical Validator	Checks content for unsupported or misleading medical claims and returns a score and issues
Brand Validator	Checks tone, clarity, call to action and promotional language and returns a score and issues
Content Refinement Agent	Rewrites the content to fix every issue the validators reported
Re-Medical / Re-Brand Validators	Validate the refined content again inside the loop
Human Review (Wait form)	Shows the content and both validation results; the reviewer approves, requests rework or rejects
✨ Features
Form-driven input — Content types: Instagram Post, Instagram Reel, Blog, Website Content, Campaign. Platforms: Instagram, YouTube, LinkedIn, Twitter, Website, Google Ads. Plus campaign objective, language, tone, additional instructions and optional reference content.
Structured routing — The orchestrator's output is validated against a JSON schema (specialist, requires_medical_validation, requires_brand_validation, requires_human_review) and routed with a Switch node.
Platform-aware generation — Instagram posts use Hook / Caption / CTA / Hashtags, reels use Hook and Script, and blog or website content is long-form (2,500 to 3,000 words) with introduction, table of contents, FAQs, People Also Ask and conclusion.
Reference-content blueprinting — Existing content can be pasted in as a structural guide. The agent follows its flow and key points without copying it.
Two-validator quality gate — Separate Medical and Brand validators return passed, score (0 to 1), issues and critical_issues as structured JSON.
Automatic refinement loop — Content that does not meet the threshold is refined and re-validated, up to 3 passes.
Human in the loop — A review form shows the content and validation scores, with APPROVE, REQUEST_REWORK and REJECT options and a comments field.
Safety-focused prompts — The agents are told never to invent statistics or success rates, never to guarantee outcomes, and never to give individual medical advice.
Clean final output — The final response is converted to plain text (no HTML, JSON or markdown markers).
🖥️ Tech Stack
Layer	Technology
Workflow orchestration	n8n
LLM	OpenAI gpt-5-mini via n8n Chat Model nodes
Agents	n8n AI Agent nodes
Output validation	n8n Structured Output Parser (JSON schema)
Decision and loop logic	JavaScript in n8n Code nodes
User interface	n8n Form Trigger and Wait (form) nodes
📂 Project Structure
n8n-ai-content-engine/
│
├── README.md          # This file
├── workflow.json      # Exported n8n workflow (private details removed)
│
└── images/
    └── workflow.png   # Screenshot of the workflow on the n8n canvas
⚙️ Installation & Setup
Prerequisites
An n8n instance (n8n Cloud, Docker, or npx n8n with Node.js installed)
Access to the OpenAI API (set up as a credential in n8n)
1. Clone the Repository
bash
git clone https://github.com/Anuzz2692/n8n-ai-content-engine.git
cd n8n-ai-content-engine
2. Import the Workflow

In n8n, create a new workflow, open the menu (⋯) and choose Import from file, then select workflow.json.

3. Add Your OpenAI Credentials

The workflow has 7 OpenAI Chat Model nodes. Open each one and select your own OpenAI credential. No keys are stored in this repository.

4. Run It

Open the On form submission node, copy its test URL into your browser, and submit a request.

🚀 Usage
Submit a request — Choose a content type and platform, enter a topic, language and tone, and optionally add a target audience, objective, instructions or reference content.
Let the workflow run — The orchestrator routes the request, the content is written, and both validators score it.
Watch the quality gate — If the scores are below the threshold or issues are found, the content is refined and re-validated.
Review as a human — Open the review form, read the content and the validation results, and choose APPROVE, REQUEST_REWORK or REJECT. Add comments if needed.
Get the result — The final response shows the decision, your comments and the final content.
🧠 Agent Details
Orchestrator Agent

Reads the form data and returns structured JSON. In this version only one specialist (social_agent) exists, so every supported content type is routed to it. The orchestrator is explicitly told not to select website, campaign or SEO agents because they are not implemented yet. Any fertility or medical request is flagged for medical validation and human review.

Content Generation Agent

Writes the content only. It does not validate or approve anything. It follows the platform rules above, uses a warm, patient-friendly tone, and treats any reference content as a blueprint rather than a source to copy.

Medical Validator

Reviews content for incorrect medical information, unsupported claims, treatment or pregnancy guarantees, unsupported statistics, misleading or exaggerated claims, unsafe recommendations, unrealistic expectations, and missing medical context. It flags issues but never rewrites the content.

Brand Validator

Reviews only brand and communication quality: tone, patient-friendly language, clarity, audience and platform fit, and call to action. It flags aggressive sales language, fear-based messaging, sensationalism and overpromising. It does not judge medical accuracy.

Quality Decision and Refinement Loop

Content is approved for human review only when both scores are at least 0.9 and no issues are reported. Otherwise the Content Refinement Agent fixes the issues and both validators check the result again, for up to 3 passes.

Human Review

An n8n Wait node pauses the workflow and shows a form with the content, both validation results and the three decision options.

🔧 Configuration

The main settings live inside the workflow's Code nodes:

javascript
// Quality Decision Logic and Loop Quality Check
const bothMet = medicalScore >= 0.9 && brandScore >= 0.9;

// Loop Quality Check
const maxReached = refinementRuns >= 3;
Model: change gpt-5-mini on any OpenAI Chat Model node to use a different model.
Score threshold: edit 0.9 in the two Code nodes above.
Maximum refinement passes: edit 3 in the Loop Quality Check node.
⚠️ Known Limitations
Only the social_agent path is implemented. Website, campaign and SEO specialists are not built yet.
After the third refinement pass, the loop sets the validation results to "passed" so it can end. The human review form may therefore show PASSED even when issues remain, so reviewers should read the content carefully.
REQUEST_REWORK in the review form records the decision but does not re-run the workflow.
The workflow has not been deployed or tested at scale.
🔮 Roadmap
A local version using Ollama with Qwen2.5-Coder-7B (in progress)
ChromaDB and retrieval-based generation (RAG)
Keep the real validator scores after the refinement limit is reached
Test cases and documented results
📝 Disclaimer

This is a learning and prototyping project. AI-generated health-related content must be reviewed by a qualified person before use. Nothing here is medical advice.

👤 Author

Anuj Sharma — @Anuzz2692

Built with n8n and OpenAI.
