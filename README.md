# n8n AI Content Engine

AI content generation workflow built with n8n and a local LLM (Ollama, Qwen2.5-Coder-7B). Work in progress.

## What it does
Generates content using a locally hosted language model, with the workflow orchestrated in n8n.

## Implemented
- Prompt configuration
- Conditional routing
- Retry handling
- Human review step
- Content validation
- E-E-A-T-aligned content-evaluation rubric (documented)

## Tech stack
n8n, Ollama, Qwen2.5-Coder-7B

## Status
In progress. Planned for the next version: ChromaDB and retrieval-based generation (RAG).

## How to run
1. Install n8n and Ollama.
2. Pull the model: `ollama pull qwen2.5-coder:7b`
3. In n8n, import the workflow JSON file from this repo (Workflows, then Import from file).
4. Set the Ollama URL in the workflow (default: http://localhost:11434).

## Note
This repository contains no private or clinic data.
