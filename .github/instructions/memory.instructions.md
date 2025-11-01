---
applyTo: "*"
---

# Kraken Mode Memory Index

This file serves as the master index for persistent memory across sessions. Each section references a dedicated submemory file for detailed context.

## Memory Structure

### user-preferences
→ `.github/instructions/user-preferences.md`
User-specified preferences for autonomous operation mode, communication style, and environmental constraints.

### style-avoidances
→ `.github/instructions/style-avoidances.md`
Anti-patterns to avoid, such as permission-seeking phrases and session termination patterns.

### tool-priorities
→ `.github/instructions/tool-priorities.md`
Preferred tools in order of priority for task execution.

### project-constraints
→ `.github/instructions/project-constraints.md`
Hard constraints including language versions, dependencies, service architecture, and network configuration.

### architecture-decisions
→ `.github/instructions/architecture-decisions.md`
Major architectural choices with rationale, including external access solutions, VPN integration, hardware limitations, and automation workflows.

### critical-troubleshooting
→ `.github/instructions/critical-troubleshooting.md`
Known issues and their verified solutions for Windows networking, Docker configuration, and Cloudflare Tunnel setup.

### configuration-files
→ `.github/instructions/configuration-files.md`
Key configuration files, their purpose, and current state (docker-compose.yml, .env, README.md, etc.).

### pending-tasks
→ `.github/instructions/pending-tasks.md`
Active tasks requiring completion, monitoring, or manual intervention.

### tdarr-behavior
→ `.github/instructions/tdarr-behavior.md`
Documented behavior patterns for Tdarr transcoding, including error handling, library scope, and synchronization with Radarr/Sonarr.
