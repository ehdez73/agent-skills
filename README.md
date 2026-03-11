# Agent Skills

A collection of custom Agent Skills to enhance your coding workflow with specialized capabilities for requirements gathering, BDD test generation, and more.

## Available Skills

### 1. 3-Amigos Skill
Guide a structured conversation between a Product Owner, Developer, and QA to define functionality before development.

**Use this skill when:**
- Facilitating a 3-amigos meeting to define functionality
- Guiding discussions through business objectives, use cases, and acceptance criteria
- Extracting clear features from meeting discussions for implementation

### 2. Gherkin Generator Skill
Generates high-quality Gherkin (BDD) scenarios from functional requirements using a two-agent iterative cycle.

**Use this skill when:**
- Converting functional requirements to Gherkin/BDD test cases
- Generating Feature/Scenario/Given/When/Then specifications
- Creating executable acceptance criteria
- Transforming requirements documents into behavior-driven tests

## Installation

To install these skills in your VS Code or GitHub Copilot environment:

```bash
npx skills add https://github.com/ehdez73/agent-skills --agent universal
```

This command will:
- Download the skills repository
- Register both skills with the specified agent
- Make them available in your coding environment

### Specifying the Target Agent

You can specify which agent(s) should have access to these skills using the `--agent` flag. Replace `universal` with the target agent name:

```bash
npx skills add https://github.com/ehdez73/agent-skills --agent <agent-name>
```

For a complete list of supported agents and how to configure them, see the [Vercel Skills documentation on supported agents](https://github.com/vercel-labs/skills?tab=readme-ov-file#supported-agents).

### Alternative: Manual Installation

If you prefer to manually manage the skills, you can clone the repository and copy the skill folders:

```bash
# Clone the repository
git clone https://github.com/ehdez73/agent-skills.git

# Navigate to your Claude or Copilot skills directory
# (Location varies depending on your setup)
cd ~/.claude/skills
# or
cd ~/.copilot/skills

# Copy the skill folders
cp -r /path/to/cloned/agent-skills/skills/3-amigos-skill .
cp -r /path/to/cloned/agent-skills/skills/gherkin-generator-skill .
```

After copying, restart your VS Code or GitHub Copilot environment for the changes to take effect.

## Usage

Once installed, the skills will be automatically invoked when you mention relevant keywords:

- **3-Amigos**: "3-amigos meeting", "facilitate meeting", "extract features"
- **Gherkin**: "generate Gherkin", "BDD scenarios", "requirements to Gherkin", "Feature/Scenario/Given/When/Then"

## Repository Structure

```
skills/
├── 3-amigos-skill/
│   ├── SKILL.md
│   └── references/
│       ├── feature-extractor.md
│       └── meeting-facilitator.md
└── gherkin-generator-skill/
    ├── SKILL.md
    ├── agents/
    │   ├── generator-agent.md
    │   └── reviewer-agent.md
    └── references/
        └── gherkin-best-practices.md
```


## Other skills

### Spring Boot Skill 

From https://github.com/sivaprasadreddy/sivalabs-agent-skills/

```bash
# Install Spring Boot Skill
npx skills add https://github.com/sivaprasadreddy/sivalabs-agent-skills --agent universal --skill spring-boot-skill

```

### Git Commit Skill

From https://github.com/github/awesome-copilot
```bash
# Install Git Commit Skill
npx skills add https://github.com/github/awesome-copilot --agent universal --skill git-commit
```