# PRD Examples

This directory contains real-world examples of using Product Requirements Documents (PRDs) to drive technical development projects. Each example demonstrates how to break down complex technical problems into structured requirements and actionable tasks.

## What is PRD-Based Development?

PRD-based development is a methodology that applies product management techniques to technical projects. Instead of jumping straight into code, you first create a comprehensive PRD that defines:

- **Goals & User Stories**: What you're trying to achieve and why
- **Functional Requirements**: Specific behaviors the system must have
- **Technical Considerations**: Architecture, tools, and implementation details
- **Success Metrics**: How you'll know when you're done

After the PRD, you create a detailed task breakdown that translates requirements into concrete, actionable steps.

## Directory Structure

Each project follows this consistent structure:

```
project-name/
├── prd-[project-name].md     # Main PRD document
└── tasks-prd-[project-name].md   # Detailed task breakdown
```

## Example Projects

### 1. AI Rules Sync MVP (`ai-rules-sync-mvp/`)

**Problem**: Manually copying AI assistant rule files across multiple Git repositories is tedious and leads to inconsistencies.

**Solution**: A lightweight CLI tool that syncs AI rule files from a central Git repository to local projects.

**Key Features**:

- One-command initialization: `ai-rule-sync init`
- Safe updates with drift detection: `ai-rule-sync pull`
- Easy publishing of changes: `ai-rule-sync publish`
- Cross-platform Go binary with automated releases

**Technologies**: Go, Cobra CLI, go-git, GoReleaser

### 2. AI Thoughts Digest (`ai-thoughts-digest/`)

**Problem**: Manually compiling weekly content from a Telegram channel into email digests is time-consuming.

**Solution**: A fully automated system that fetches Telegram posts, generates formatted digests, and publishes drafts to Substack.

**Key Features**:

- Real-time Telegram message ingestion using Telethon
- Weekly digest generation with Jinja2 templates
- Automated Substack publishing via unofficial GraphQL API
- Self-hosted deployment with Docker and Kamal
- Optional LLM summarization with OpenAI GPT-4

**Technologies**: Python, Telethon, SQLite, Jinja2, APScheduler, Docker, Kamal

### 3. PHP Forum Recovery (`php-forum-recovery/`)

**Problem**: A recovered phpBB forum needs to be deployed to production with zero data loss and proper backup procedures.

**Solution**: Production-ready deployment using modern containerization and infrastructure-as-code practices.

**Key Features**:

- Docker containerization of legacy phpBB application
- Zero-downtime deployments with Kamal
- Automated SSL certificate management with Let's Encrypt
- Comprehensive backup strategy with retention policies
- Email delivery integration with Oracle Cloud Infrastructure

**Technologies**: phpBB, Docker, Kamal, MariaDB, Apache, Let's Encrypt

## How to Use These Examples

### As Learning Materials

1. **Study the PRD Structure**: Notice how each PRD balances high-level goals with specific technical details
2. **Examine Task Breakdown**: See how complex requirements are decomposed into manageable tasks
3. **Follow the Process**: Observe how requirements drive implementation decisions

### As Templates

1. **Copy the Structure**: Use the PRD template format for your own projects
2. **Adapt the Sections**: Modify sections like "Goals", "User Stories", and "Technical Considerations" for your needs
3. **Customize Task Format**: Adjust the task breakdown style to match your workflow

### For AI-Assisted Development

These examples work particularly well with AI coding assistants:

1. **Clear Context**: PRDs provide comprehensive context for AI agents
2. **Structured Tasks**: Detailed task lists give AI clear instructions
3. **Success Criteria**: Well-defined requirements help validate AI-generated solutions

## Key Benefits of This Approach

### 1. **Clarity Before Coding**

- Forces you to think through requirements before implementation
- Identifies potential issues and edge cases early
- Creates shared understanding for team projects

### 2. **Better Project Management**

- Clear success metrics and milestones
- Easier to track progress and stay focused
- Helps prevent scope creep

### 3. **Documentation as a Byproduct**

- PRDs serve as project documentation
- Task lists provide implementation history
- Easy to onboard new team members

### 4. **AI-Friendly Format**

- Structured requirements work well with AI coding assistants
- Detailed context leads to better AI-generated code
- Clear tasks reduce back-and-forth with AI agents

## Best Practices

### Writing Effective PRDs

1. **Start with the Problem**: Always begin by clearly stating what you're trying to solve
2. **Define Success**: Include measurable goals and success criteria
3. **Be Specific**: Vague requirements lead to implementation confusion
4. **Consider Non-Goals**: Explicitly state what's out of scope

### Creating Good Task Breakdowns

1. **Make Tasks Actionable**: Each task should be something you can actually do
2. **Include Verification**: Add steps to validate that tasks are complete
3. **Follow Dependencies**: Order tasks logically based on dependencies
4. **Keep Tasks Small**: Break large tasks into smaller, manageable pieces

### Working with AI Assistants

1. **Provide Full Context**: Share both the PRD and current task list
2. **Be Explicit**: Include technical details and constraints in requirements
3. **Validate Outputs**: Use success criteria to verify AI-generated solutions
4. **Iterate Incrementally**: Complete tasks one at a time for better results

## Related Resources

- [PRD Task List Process](../prd-tasklist-process/) - Step-by-step guide for creating PRDs and task lists
- [SRS Sidekick Project Example](../srs-sidekick-project-example/) - Comprehensive example with additional documentation types

## Contributing

If you have suggestions for improving these examples or would like to contribute additional PRD examples, please feel free to submit a pull request or open an issue.

---

These examples demonstrate that taking time to write comprehensive requirements upfront leads to smoother development, better outcomes, and more maintainable projects. Whether you're working solo or with a team, the PRD-based approach provides structure and clarity that makes complex technical projects more manageable.
