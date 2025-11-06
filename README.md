# AFv2 Pattern #6: Hierarchy

Supervisor-orchestrated task delegation to specialist roles with review gates.

## Pattern Structure

```
Start → Supervisor → Checker → [Worker → Reviewer → loop back to Checker] → Final → Direct Reply
```

## Key Features

- Role-based architecture (Supervisor, Worker, Reviewer)
- Step iterator (hierarchy.current_step incremented by Reviewer)
- Loop-back edge (Reviewer → Checker, animated)
- Role-specific tool ACLs

## Files

- `06-hierarchy.json` - Complete Flowise workflow (877 lines)

## Quick Start

1. Import `06-hierarchy.json` into Flowise
2. Configure Anthropic API key for all agents
3. Test with task delegation scenario

## Use Cases

- Software development workflows (planning → coding → review)
- Content creation (research → write → edit)
- Project management (delegate → execute → validate)
- Multi-role task orchestration

## Documentation

See [Context Foundry Pattern Library](https://github.com/context-foundry/context-foundry/tree/main/extensions/flowise/templates/afv2-patterns) for complete documentation.

🤖 Built with Context Foundry
