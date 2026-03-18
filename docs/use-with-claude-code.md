# Use with Claude Code

Claude Code provides the most integrated experience — skills load natively and the interviewer persona persists throughout the session.

## Prerequisites

- [Claude Code](https://claude.ai/code) installed
- This repository cloned locally

## Setup

```bash
git clone https://github.com/ps06756/The-Interview-Mentor.git
cd The-Interview-Mentor
```

## Start an interview

Load a skill category, then ask for a specific interviewer:

```bash
# System design (13 skills)
claude --plugin-dir ./agents/systems-design
> "Use the uber-interviewer skill and start my mock interview."

# Coding - entry level (5 skills)
claude --plugin-dir ./agents/swe-i
> "Use the arrays-hashmaps-interviewer skill and interview me."

# Coding - mid level (3 skills)
claude --plugin-dir ./agents/swe-ii
> "Use the dynamic-programming-interviewer skill."

# Data engineering (3 skills)
claude --plugin-dir ./agents/data-engineer

# DevOps / SRE (3 skills)
claude --plugin-dir ./agents/devops-sre

# ML engineering (2 skills)
claude --plugin-dir ./agents/ml-engineer

# AI product management (3 skills)
claude --plugin-dir ./agents/ai-pm

# Debugging & incident response (6 skills)
claude --plugin-dir ./agents/debugging

# Behavioral (1 skill)
claude --plugin-dir ./agents/behavioral

# Meta-skills (1 skill)
claude --plugin-dir ./agents/meta
```

## What to expect

Once loaded, the interviewer will:
1. Greet you and immediately start with a warm-up question
2. Adapt difficulty based on your answers
3. Provide hints when you ask (4 levels of progressively detailed help)
4. Generate a scorecard at the end with specific improvement areas

## Tips

- **One category at a time.** `--plugin-dir` loads all skills in that folder.
- **Say the exact skill name** (e.g., "uber-interviewer") so Claude picks the right one.
- **Ask for hints** naturally: "Can I get a hint?" or "I'm stuck, help me."
- **Request the scorecard** at the end: "Give me my evaluation."

## Alternative: Plugin Marketplace

You can also install via the marketplace:

```
/plugin marketplace add <path-to-cloned-repo>
/plugin install coding-interview-agent
```

Then start with: *"Help me prepare for a system design interview using coding-interview-agent."*
