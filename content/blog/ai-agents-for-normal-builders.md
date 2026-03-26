---
title: "AI Agents for Normal Builders: A Practical Guide to Getting Real Work Done"
date: 2026-03-26
lastmod: 2026-03-26
draft: false
description: "A practical guide to building AI agents that actually ship. Covers agent maturity levels, bounded agency, real-world use cases, Claude Code examples, and a launch checklist for builders who want agents that do real work."
summary: "AI agents are having a moment. Most of the conversation is either hype or doom. Neither helps you ship. This is a practical guide for builders who want agents that do real work — with concrete examples using Claude Code."
keywords:
  - "AI agents"
  - "agentic AI"
  - "Claude Code"
  - "AI agent best practices"
  - "bounded agency"
  - "agent maturity"
  - "LLM agents"
  - "AI automation"
  - "agent guardrails"
  - "AI workflows"
tags:
  - "ai-agents"
  - "agentic-ai"
  - "claude-code"
  - "engineering-leadership"
  - "automation"
categories:
  - "AI"
  - "Engineering"
series:
  - "AI Agents"
author: "Kuldeep Chowhan"

cover:
  image: ""
  alt: ""
  caption: ""
  relative: true

ShowToc: true
TocOpen: false
ShowReadingTime: true
ShowShareButtons: true
ShowPostNavLinks: true

faq:
  - question: "What is an AI agent?"
    answer: "An AI agent is a system that combines a large language model with tools, memory, and a loop to complete outcomes autonomously. Unlike a chatbot that answers prompts, an agent interprets a goal, plans steps, takes actions by calling tools, observes results, corrects course, and stops when success criteria are met."
  - question: "What is bounded agency in AI agents?"
    answer: "Bounded agency is the practice of constraining an AI agent to a specific scope, toolset, limits, and stop rules. Instead of asking an agent to handle customer support, you ask it to draft replies for returns under $50 and route edge cases. Boundaries keep agents useful without making them dangerous."
  - question: "What is the best level of agent maturity to start with?"
    answer: "Level 2 (tool-using helper) plus Level 3 (human-in-the-loop operator) is the sweet spot for teams starting with AI agents. Level 2 handles research, analysis, and triage. Level 3 proposes actions that a human approves. Together they deliver real value without the risk of full autonomy."
  - question: "How do I build an AI agent with Claude Code?"
    answer: "Start by defining a clear outcome with success criteria. Use Claude Code's agent loop — it already implements the interpret-plan-act-observe-correct cycle. Give it specific tools via MCP servers, constrain its scope with a focused system prompt, require verification steps, and add explicit stop rules. Start read-only, then expand to write actions after you trust the logs."
  - question: "Why do most AI agents fail in production?"
    answer: "The five most common failure modes are: vague goals without measurable success criteria, no ground truth for the agent to verify its work against, too many tools causing unpredictable behavior, no escalation path when the agent is uncertain, and no monitoring or logging of what the agent actually did."
---

AI agents are having a moment. Most of the conversation is either hype or doom. Neither helps you ship. An AI agent is not magic — it is a workflow. If you strip it down, an agent is simply an LLM combined with tools, memory, a loop, and guardrails. When that combination is designed well, it compresses 30-to-90-minute tasks into 3-to-10 minutes of review. When designed poorly, it becomes a confident intern that breaks production.

This post is a practical guide for builders who want agents that do real work — with concrete examples using Claude Code.

## What Is an AI Agent, Really?

A chatbot answers prompts. An agent completes outcomes.

The difference is the loop:

1. Interpret the goal
2. Plan the steps
3. Take an action (call tools, read data, write output)
4. Observe the result
5. Correct and continue
6. Stop when success criteria are met

If your system cannot take actions and verify results, it is not an agent. It is autocomplete with extra steps.

Claude Code implements this loop natively. When you give it a task, it plans, calls tools (bash, file read/write, search), observes output, adjusts, and continues until done. That is why it works as both a practical coding tool and a reference architecture for how agents should behave. {{< cite source="Anthropic Claude Code Docs" url="https://docs.anthropic.com/en/docs/claude-code" >}}

## What Are the 4 Levels of Agent Maturity?

Most teams skip levels and get burned. Start here.

### Level 1: Assisted Drafting

The agent produces output, you paste it somewhere.

Examples: write an email, summarize notes, generate a spec, draft SQL, create an outline.

When it works: low risk, high speed. Failure mode: none, because you are the executor.

**Claude Code in practice — generating a spec:**

```bash
claude "Read the codebase in ~/src/myapp and generate a technical spec
for adding rate limiting to the API layer. Include current architecture,
proposed changes, and migration steps."
```

Claude Code reads your files, understands the architecture, and produces a spec you review. Level 1. No risk.

### Level 2: Tool-Using Helper

The agent can call tools but cannot change important systems.

Examples: search docs, query analytics, read logs, generate reports, propose Jira tickets.

When it works: research, analysis, triage. Failure mode: wrong interpretation of data. Fix with citations and verification.

**Claude Code in practice — codebase analysis:**

```bash
claude "Analyze the error handling patterns across this repo. Find
inconsistencies, missing error boundaries, and cases where we swallow
exceptions silently. Produce a prioritized list with file paths and
line numbers."
```

This is read-only. Claude Code searches, reads, and reports. It cannot break anything. According to Anthropic's own benchmarks, Claude scores 72.7% on SWE-bench Verified — meaning it successfully resolves real GitHub issues nearly three-quarters of the time. {{< cite source="Anthropic Research" url="https://www.anthropic.com/research/swe-bench-sonnet" >}} That accuracy matters when you are using it to find bugs.

### Level 3: Human-in-the-Loop Operator

The agent proposes actions and you approve each one.

Examples: refund approvals, CRM updates, deployment checklists, vendor outreach, code changes.

When it works: operations, customer support, and engineering workflows. Failure mode: approval fatigue. Fix with batching and better summaries.

**Claude Code in practice — interactive code changes:**

```bash
claude "Refactor the authentication middleware to use JWT with
refresh tokens. Show me each file change before applying it.
Run the test suite after each change."
```

Claude Code proposes changes, you review diffs, approve each one. It runs tests after every modification. This is the pattern I use daily — the agent does the work, I make the decisions.

### Level 4: Limited Autonomy

The agent executes within strict boundaries.

Examples: reorder inventory within min/max, auto-label inbox, retry failed jobs, close low-risk tickets, run CI/CD pipelines.

When it works: repetitive, rules-based tasks with clear guardrails. Failure mode: silent drift. Fix with monitoring, audits, and hard stop conditions.

**Claude Code in practice — autonomous task execution with hooks:**

```bash
# CLAUDE.md project config that constrains the agent
# This file lives in your repo root

## Allowed actions
- Run tests with `npm test`
- Fix lint errors automatically
- Update snapshot files
- Create git commits for passing changes

## Prohibited actions
- Do NOT modify database migrations
- Do NOT change environment variables
- Do NOT push to remote
- Do NOT modify files in /config/production/

## Stop rules
- Stop and ask if any test fails after 2 retry attempts
- Stop and ask before modifying more than 5 files in a single change
- Stop and ask if a change touches authentication or billing logic
```

The `CLAUDE.md` file is your boundary specification. It tells Claude Code exactly what it can and cannot do before it starts. Combined with pre- and post-commit hooks, you get an agent that operates autonomously within a strict box. {{< cite source="Claude Code Configuration" url="https://docs.anthropic.com/en/docs/claude-code/settings" >}}

{{< callout type="summary" >}}
If you are starting today, Level 2 plus Level 3 is the sweet spot. Read-only analysis combined with human-approved actions gives you 80% of the value with near-zero risk.
{{< /callout >}}

## What Is Bounded Agency and Why Does It Matter?

Do not ask an agent to "handle customer support." Ask it to "draft replies for returns under $50 and route edge cases."

Bounded agency is how you keep agents useful without making them dangerous. A good boundary has four parts:

1. **Scope** — what it is allowed to touch
2. **Tools** — what it can call
3. **Limits** — time, budget, number of steps, rate limits
4. **Stop rules** — when to escalate to a human

This is boring by design. Boring is safe. Safe is scalable.

**How Claude Code implements bounded agency:**

Claude Code's architecture already enforces this pattern. MCP (Model Context Protocol) servers define exactly which tools the agent can access. {{< cite source="MCP Specification" url="https://modelcontextprotocol.io/" >}} You can give it a database read tool without a write tool. You can give it a Jira comment tool without a ticket-close tool. The permission model is the boundary.

```bash
# Example: agent that can read your database and Slack,
# but can only write to a markdown file
claude --mcp-config ./mcp-readonly.json \
  "Analyze our top 10 support tickets from last week.
   Cross-reference with error logs. Write a summary
   to weekly-support-digest.md"
```

## What Does an Agent Blueprint That Actually Ships Look Like?

Here is the template I use.

### 1. Define the Outcome and Success Criteria

Bad: "Improve onboarding."
Good: "Reduce time-to-first-value from 20 minutes to under 8 minutes."

Write success criteria like a QA spec:

- Must produce X
- Must not do Y
- Must cite sources for Z
- Must ask for approval when uncertain

### 2. Give It a Small Set of Tools

Agents fail when they have too many buttons. A study by Microsoft Research found that agent performance degrades significantly when tool count exceeds 10 — the model spends more tokens deciding which tool to use than actually using it. {{< cite source="Microsoft Research" url="https://www.microsoft.com/en-us/research/blog/magentic-one-a-generalist-multi-agent-system-for-solving-complex-tasks/" >}}

Common tools that work:

- Search docs and knowledge base
- Read-only analytics queries
- Ticket creation (draft mode)
- Email draft creation
- Calendar scheduling
- Database read for customer context

Start read-only. Move to write actions only after you have logs and confidence.

**Claude Code example — minimal tool configuration:**

```json
{
  "mcpServers": {
    "postgres-readonly": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-postgres"],
      "env": {
        "DATABASE_URL": "postgresql://readonly_user@localhost/myapp"
      }
    },
    "docs-search": {
      "command": "npx",
      "args": ["-y", "@anthropic/mcp-docs-server", "./docs"]
    }
  }
}
```

Two tools. Read-only database. Documentation search. That is enough to build a powerful analysis agent.

### 3. Add Memory Only Where It Matters

Memory is not "store everything." Store only stable, high-value facts:

- Customer tier, plan limits, preferences
- Product configuration
- Common decisions and policies

If memory is noisy, the agent becomes inconsistent.

In Claude Code, the `CLAUDE.md` file is your agent's long-term memory. Project-specific context, coding conventions, architectural decisions — things that do not change between sessions. Keep it focused. I have seen teams dump 2,000 lines into their config and wonder why the agent ignores half of it.

### 4. Force Verification

Make the agent prove it is right. This is the step most builders skip, and it is the one that separates demos from production systems.

Verification patterns:

- Quote and cite exact policy sections
- Run a query and include the query text and result
- Produce a checklist and mark each item as verified or assumed
- Do a second-pass "critic" step that tries to break the answer

**Claude Code example — built-in verification:**

```bash
claude "Fix the failing test in tests/auth.test.js. After fixing it,
run the full test suite. If all tests pass, run the linter.
Only commit if both pass. Show me the diff before committing."
```

This is a verification chain: fix, test, lint, diff review, then commit. Each step must succeed before the next one runs. Claude Code does this naturally — it checks tool output and adjusts.

### 5. Use a Clean Handoff Format

Humans should not read agent thoughts. They should read decisions.

A reliable handoff looks like:

- **Summary** (3 bullets max)
- **Proposed action**
- **Evidence** (links, citations, logs)
- **Risks and edge cases**
- **Approval needed** (yes / no / escalate)

## What Agent Use Cases Work for Almost Any Team?

### Use Case 1: Support Triage Agent

Goal: reduce response time without sacrificing quality.

What it does: reads the ticket, classifies intent (refund, shipping, bug, billing), pulls relevant policy and order context, drafts a response, and flags risk (chargeback risk, angry customer, legal language).

Boundaries: cannot send without approval, cannot offer discounts beyond a threshold, and escalates for safety, legal, threats, or PR risk.

### Use Case 2: Growth Research Agent

Goal: compress research and competitive analysis.

What it does: monitors competitor updates, extracts changes in pricing, positioning, and feature announcements, produces a weekly memo with diffs and links, and suggests experiments.

Boundaries: read-only, must include sources for every claim, and "unknown" is allowed and encouraged.

### Use Case 3: Engineering Ops Agent

Goal: make the team run with fewer meetings.

**Claude Code in practice — weekly engineering digest:**

```bash
claude "Pull the last 7 days of merged PRs from our GitHub repo.
Summarize what shipped, what's still in review, and any PRs
with failing checks. Cross-reference with our Jira board to
flag tickets that are marked done but don't have a merged PR.
Write the digest to weekly-engineering-update.md"
```

Boundaries: creates drafts only, no production changes, no customer outreach without approval.

## Why Do Most Agents Fail in Production?

**Failure 1: Vague goals.** "Handle marketing" is not a goal. It is a department. Fix: reduce scope until success is measurable.

**Failure 2: No ground truth.** If the agent cannot check its work, it will hallucinate with confidence. A Stanford study found that LLM-based agents with access to verification tools reduced error rates by 40% compared to agents without them. {{< cite source="Stanford HAI" url="https://hai.stanford.edu/" >}} Fix: give it access to data and require citations.

**Failure 3: Tool chaos.** Too many tools yields unpredictable behavior. Fix: start with 2 to 4 tools, expand slowly.

**Failure 4: No escalation path.** Agents need a clear "I am not sure" behavior. Fix: define stop rules and route to a human. In Claude Code, this is the `CLAUDE.md` stop rules section — explicit instructions on when to ask instead of act.

**Failure 5: No monitoring.** If you cannot see what it did, you cannot trust it. Fix: log prompts, tool calls, outputs, and approvals. Sample and audit weekly.

{{< callout type="tip" >}}
Claude Code logs every tool call and its output by default. Use `claude --verbose` to see the full chain of reasoning, tool calls, and decisions. This is your audit trail.
{{< /callout >}}

## What Does a Minimum Viable Agent Stack Look Like?

If you are building an agent system today, keep it straightforward:

- **Agent runtime**: an orchestrator that runs the loop (Claude Code, LangGraph, or a custom loop)
- **Tools**: your APIs wrapped with strict schemas and permissions
- **Memory**: a small context store plus a retrieval index for docs
- **Observability**: logs for every run, plus a dashboard for failures
- **Human approval UI**: where the agent proposes actions and you approve

If you cannot see the tool calls and the evidence, you do not have an agent. You have a demo.

For builders who want to skip the infrastructure, Claude Code with MCP servers is the fastest path to a real agent. You define tools as MCP servers, configure boundaries in `CLAUDE.md`, and the agent loop, memory, and tool calling are handled for you. I have used this pattern to build internal tools that went from idea to production in a single afternoon.

## A Launch Checklist You Can Use This Week

- [ ] Define one workflow with clear success criteria
- [ ] Restrict tool access to read-only at first
- [ ] Add explicit stop rules and escalation paths
- [ ] Require citations or evidence for key claims
- [ ] Add rate limits, step limits, and timeouts
- [ ] Log everything and sample 20 runs for review
- [ ] Ship Level 2 or Level 3 before trying autonomy
- [ ] Write a `CLAUDE.md` that encodes your boundaries
- [ ] Run the agent 10 times and review every output before trusting it

## The Pragmatic Take

AI agents are not a replacement for teams. They are a force multiplier for workflows.

If you build them with bounded scope, verification, and clean handoffs, they pay back fast. If you build them as "do everything" bots, they become a liability.

Start small. Pick one workflow you repeat weekly. Give the agent a tight box. Make it prove its work. Then expand.

The builders who win with agents in 2026 will not be the ones with the most sophisticated architectures. They will be the ones who defined the clearest boundaries.

---

## Frequently Asked Questions

{{< faq question="What is an AI agent?" >}}
An AI agent is a system that combines a large language model with tools, memory, and a loop to complete outcomes autonomously. Unlike a chatbot that answers prompts, an agent interprets a goal, plans steps, takes actions by calling tools, observes results, corrects course, and stops when success criteria are met.
{{< /faq >}}

{{< faq question="What is bounded agency in AI agents?" >}}
Bounded agency is the practice of constraining an AI agent to a specific scope, toolset, limits, and stop rules. Instead of asking an agent to handle customer support, you ask it to draft replies for returns under $50 and route edge cases. Boundaries keep agents useful without making them dangerous.
{{< /faq >}}

{{< faq question="What is the best level of agent maturity to start with?" >}}
Level 2 (tool-using helper) plus Level 3 (human-in-the-loop operator) is the sweet spot for teams starting with AI agents. Level 2 handles research, analysis, and triage. Level 3 proposes actions that a human approves. Together they deliver real value without the risk of full autonomy.
{{< /faq >}}

{{< faq question="How do I build an AI agent with Claude Code?" >}}
Start by defining a clear outcome with success criteria. Use Claude Code's agent loop — it already implements the interpret-plan-act-observe-correct cycle. Give it specific tools via MCP servers, constrain its scope with a focused system prompt or CLAUDE.md file, require verification steps, and add explicit stop rules. Start read-only, then expand to write actions after you trust the logs.
{{< /faq >}}

{{< faq question="Why do most AI agents fail in production?" >}}
The five most common failure modes are: vague goals without measurable success criteria, no ground truth for the agent to verify its work against, too many tools causing unpredictable behavior, no escalation path when the agent is uncertain, and no monitoring or logging of what the agent actually did.
{{< /faq >}}
