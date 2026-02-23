# ChatChat Community Skills

Community-contributed skills for ChatChat AI agents. Try at chatchat.com. 
Skills are natural language instruction sets that guide agent behavior for specific tasks.

## Using Skills

Skills from this repository are automatically synced to the ChatChat Skills Store. To use a skill:

1. Open your agent's profile page
2. Click **Edit** next to the Skills section
3. Browse the Skills Store
4. Click **Install** on any skill

The installed skill will be available as a tool the LLM can automatically invoke when relevant.

## Contributing

See [CONTRIBUTING.md](CONTRIBUTING.md) for detailed instructions on how to create and submit skills.

## Structure

```
skills/
  prep-meeting/
    SKILL.md
  contract-review/
    SKILL.md
  draft-reply/
    SKILL.md
  ...
index.json          # Auto-generated manifest (do not edit manually)
```

## Categories

Each skill declares a `category` in its frontmatter. Categories help users browse the Skills Store and should reflect the user-facing purpose of the skill.

Common categories you’ll see:
- Productivity
- Communication
- Business
- Lifestyle
- Content & Media
- Development
- Data & AI
- Research
- Documentation
- Design
- DevOps
- Testing & Security
- Tools

Use the most specific, user-friendly category that matches the skill’s primary intent. If a skill spans multiple domains, pick the category users would most likely search.
