# Claude Code Integration

## Using The Mentor with Claude Code

### Quick Start

```bash
# Start an interview session
cat roles/swe-i/arrays-hashmaps-interviewer.md | claude

# Or use the --skill flag (if supported in your version)
claude --skill roles/swe-i/arrays-hashmaps-interviewer.md
```

### Settings Configuration

Add to your `.claude/settings.json`:

```json
{
  "skills": [
    {
      "name": "arrays-hashmaps-swe-i",
      "path": "/path/to/roles/swe-i/arrays-hashmaps-interviewer.md",
      "description": "Practice arrays and hashmaps for entry-level interviews"
    },
    {
      "name": "sql-optimization",
      "path": "/path/to/roles/data-engineer/sql-optimization-interviewer.md",
      "description": "Practice SQL optimization for data engineering roles"
    },
    {
      "name": "url-shortener",
      "path": "/path/to/roles/systems-design/url-shortener-interviewer.md",
      "description": "System design practice - URL shortener service"
    }
  ]
}
```

### Tips for Best Results

1. **Be explicit about loading the skill**: Start with "Please load the [skill name] skill and interview me"
2. **Use the hint system**: Say "I need a hint" or "Give me a level 2 hint" when stuck
3. **Request visualizations**: Ask "Can you show me a diagram of that?" for complex concepts
4. **Get feedback**: At the end, ask for a detailed evaluation using the rubric
