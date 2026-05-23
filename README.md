# Playbook

> Two parallel study tracks — **modern applied AI** and **Cloud/AWS** — for software developers who want to understand, decide, and build.

[![License: CC BY 4.0](https://img.shields.io/badge/License-CC%20BY%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by/4.0/)

**Written in Italian.** The content is structured around problems to solve, not around tools or service lists. Tools change fast; the underlying decisions don't.

---

## What's inside

Two self-contained learning tracks, meant to be studied in parallel:

- **AI Playbook** — how LLMs work, RAG, agents, evaluation, system design, security, production
- **Cloud Playbook** — cloud fundamentals, networking, containers, IaC, AWS in practice, cloud for AI, operations

Both tracks share the same voice, the same lesson template, and the same quality bar. Each lesson includes retrieval questions, a misconception table, and a glossary.

**Target audience**: developers with a software background (Python, full-stack, backend). No formal ML required.

---

## Contents

### AI Playbook (`ai/`)

| Part | Topics |
|---|---|
| 0 — Foundations | How LLMs work, embeddings, prompt engineering, reasoning models |
| 1 — Building | RAG, fine-tuning, agents, MCP, structured output, context engineering |
| 2 — Multimodal | Vision, audio, image/video generation, voice agents |
| 3 — Evaluation | Evals, hallucination, LLM-as-judge, observability, evaluating agents |
| 4 — System design | Anatomy, speed/cost/quality triangle, patterns, caching, data |
| 5 — Architecture | Build vs buy, reference architecture, decision drill |
| 6 — Production | Serving, FinOps, monitoring, canary/A-B rollout |
| 7 — Security | Prompt injection, privacy, EU AI Act, agent security |

### Cloud Playbook (`cloud/`)

| Part | Topics |
|---|---|
| 0 — Foundations | What cloud really is, IaaS/PaaS/SaaS, regions, storage, Linux essentials |
| 1 — Networking & security | VPC, DNS/TLS/LB/CDN, IAM concepts, secrets management |
| 2 — Compute & runtime | Containers, Kubernetes (concepts), serverless, async, API design, streaming |
| 3 — IaC & automation | Terraform, policy-as-code, CI/CD |
| 4 — AWS in practice | EC2/ECS/Lambda, SQS/SNS/EventBridge, S3/RDS/DynamoDB, networking, IaC |
| 5 — Cloud for AI | GPUs in the cloud, Bedrock vs self-hosted, inference costs, K8s for AI, RAG on AWS |
| 6 — Operations | FinOps, monitoring, distributed tracing, security, resilience, compliance |
| 7 — Synthesis | Reference architecture, end-to-end capstone, portfolio |

---

## How to use it

**Read online** (recommended): browse the published site.

**Run locally**:
```bash
npm install
npm start       # dev server at http://localhost:3000
npm run build   # static build in /build
```

**As plain Markdown**: every lesson is a standalone `.md` file — works directly in Obsidian, VS Code, or any Markdown editor.

### Study method

Every lesson ends with a **Retrieval check** — open-ended questions to answer from memory *after* reading. Review the ones you got wrong the next day. That's it. That's the whole method.

---

## Repo structure

```
ai/           AI Playbook — Markdown lessons
cloud/        Cloud Playbook — Markdown lessons
src/          Docusaurus homepage
prompts/      Prompts for writing new lessons with AI assistance
AGENTS.md     Contribution guide (lesson template, voice, conventions)
```

---

## Contributing

Lessons follow a precise template defined in [`AGENTS.md`](AGENTS.md): voice, structure, status badges, term conventions. Read it before adding or editing a lesson.

Pull requests welcome for:
- Factual corrections
- Service/pricing updates (with source)
- New lessons following the template

---

## License

[CC BY 4.0](LICENSE) — free to use, share, and adapt with attribution.
