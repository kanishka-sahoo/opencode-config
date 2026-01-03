# OpenCode Configuration: Agent Orchestration Setup

This directory contains a specialized OpenCode configuration optimized for coordinated multi-agent development workflows with external documentation and code search capabilities.

## Overview

This configuration disables OpenCode's default agents in favor of a **development orchestrator pattern** where specialized subagents handle specific tasks under coordinated management. The setup integrates two MCP (Model Context Protocol) servers to provide access to external documentation and real-world code examples.

## Configuration Details

### Disabled Default Agents

The following OpenCode default agents are disabled in `opencode.json`:

- **plan**: Default planning agent
- **build**: Default build/implementation agent  
- **general**: General-purpose agent

**Rationale**: These agents are replaced by a more sophisticated multi-agent system where the development orchestrator coordinates specialized subagents (code-implementer, error-debugger, security-auditor, test-runner) to ensure comprehensive, quality-assured development workflows.

### MCP Servers

#### context7
- **URL**: `https://mcp.context7.com/mcp`
- **Purpose**: Provides access to up-to-date documentation and library references for various programming frameworks and libraries
- **Use Cases**: 
  - Looking up API documentation
  - Checking library references
  - Verifying framework conventions
  - Researching best practices

#### gh_grep
- **URL**: `https://mcp.grep.app`
- **Purpose**: Searches real-world code examples from over a million public GitHub repositories
- **Use Cases**:
  - Finding implementation patterns
  - Seeing how others solved similar problems
  - Learning idiomatic code patterns
  - Reference implementations

## Agent Architecture

### Primary Agents

**development-orchestrator**
- **Mode**: Primary
- **Role**: Central coordinator for complex development workflows
- **Responsibilities**:
  - Analyze requirements and plan execution strategy
  - Delegate tasks to appropriate subagents
  - Monitor progress and validate outputs
  - Ensure security reviews and testing are performed
  - Provide frequent progress updates to users
- **Workflow**: Implementation → Security Review → Testing (with debugging loops as needed)

**codebase-investigator**
- **Mode**: Primary
- **Role**: Deep code analysis and architecture explanation
- **Responsibilities**:
  - Investigate codebases thoroughly
  - Explain architecture and design patterns
  - Trace processing pipelines and flows
  - Provide detailed implementation explanations
- **Constraints**: Read-only (no code modifications)

### Subagents

**code-implementer**
- **Mode**: Subagent
- **Role**: Write clean, production-quality code
- **Responsibilities**:
  - Implement features based on specifications
  - Modify existing code (bug fixes, improvements)
  - Ensure code follows best practices and SOLID principles
  - Include proper error handling and documentation

**error-debugger**
- **Mode**: Subagent
- **Role**: Diagnose and fix bugs, errors, and unexpected behavior

**security-auditor**
- **Mode**: Subagent
- **Role**: Review code for vulnerabilities and security best practices
- **Trigger Points**:
  - Before deploying authentication/authorization features
  - After changes to security-critical areas
  - When handling user input, external APIs, or sensitive data

**test-runner**
- **Mode**: Subagent
- **Role**: Execute tests, generate coverage, verify functionality

**system-design-architect**
- **Mode**: Subagent
- **Role**: System architecture and design planning

## Skills

### backend-engineering
- **Path**: `skill/backend-engineering/SKILL.md`
- **Focus**: Production-grade backend systems
- **Principles**:
  - Correctness first, performance second
  - Explicit over clever
  - Defensive programming
  - Full observability

### frontend-design
- **Path**: `skill/frontend-design/SKILL.md`
- **Focus**: Production-grade frontend interfaces
- **Approach**: High design quality with creative, polished code

## Workflow Pattern

### Standard Development Workflow

1. **User Request**: User describes desired functionality or problem
2. **Orchestration**: development-orchestrator analyzes requirements
3. **Delegation**: Orchestrator delegates to appropriate subagents:
   - code-implementer for new code/changes
   - error-debugger if issues reported
   - security-auditor for sensitive features
   - test-runner for verification
4. **Validation**: Orchestrator validates subagent outputs
5. **Iteration**: If issues found, orchestrator re-delegates with feedback
6. **Completion**: Orchestrator verifies all requirements met and provides summary

### Investigation Workflow

1. **User Query**: User asks about codebase architecture or behavior
2. **Investigation**: codebase-investigator analyzes relevant code
3. **Research**: Uses MCP servers for documentation and patterns
4. **Reporting**: Provides structured explanation with code references

### Using MCP Servers

**context7 Usage**:
```
Query: "How to set up authentication with JWT in Express.js"
```

**gh_grep Usage**:
```
Query: "useState loading state example TypeScript"
```

Both servers are automatically available to all agents for research and reference.

## Directory Structure

```
opencode/
├── opencode.json              # Configuration (disabled defaults, MCP servers)
├── agent/
│   ├── development-orchestrator.md  # Primary orchestrator agent
│   ├── codebase-investigator.md     # Primary investigation agent
│   ├── code-implementer.md          # Subagent: implementation
│   ├── error-debugger.md            # Subagent: debugging
│   ├── security-auditor.md          # Subagent: security review
│   ├── test-runner.md               # Subagent: testing
│   └── system-design-architect.md   # Subagent: architecture
├── skill/
│   ├── backend-engineering/SKILL.md  # Backend development skill
│   └── frontend-design/SKILL.md      # Frontend development skill
└── README.md                        # This file
```

## Key Benefits

1. **Quality Assurance**: Security reviews and testing are mandatory, not optional
2. **Specialization**: Each agent focuses on what it does best
3. **Coordination**: Orchestrator ensures nothing falls through the cracks
4. **External Knowledge**: MCP servers provide current documentation and real-world patterns
5. **Comprehensive Coverage**: Investigation, implementation, security, testing, and debugging all covered

## Interaction Pattern

When working with this setup:

1. **For new features/bug fixes**: The development-orchestrator will coordinate the full workflow
2. **For code questions**: The codebase-investigator will analyze and explain
3. **For implementation details**: The orchestrator delegates to code-implementer
4. **For security-sensitive work**: Security-auditor is automatically invoked
5. **For verification**: Test-runner validates functionality

All agents have access to context7 (documentation) and gh_grep (code examples) for research.

## Notes

- This configuration is designed for complex, production-quality development workflows
- The orchestrator pattern ensures comprehensive quality gates
- MCP servers are read-only and used for research purposes only
- All agents follow the principles defined in the skill files for their domain
