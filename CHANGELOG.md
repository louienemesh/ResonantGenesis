# Changelog

All notable changes to ResonantGenesis will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.1.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
- Agent Creation Wizard with 5-step guided flow
- Marketplace agent templates (Research Assistant, Code Helper, Data Analyst)
- Enhanced Help Center with FAQ, keyboard shortcuts, reading time
- Resources dropdown with 9 navigation links
- Control Center section headers for better organization
- Architecture documentation with Mermaid diagrams
- Comprehensive API reference with Quick Start guide
- Security policy and vulnerability reporting process
- Contributing guidelines with development setup

### Changed
- Renamed "General Store" to "Marketplace"
- Renamed "Agents" to "Agent Studio"
- Reorganized Control Center into Monitoring, Analysis, Governance sections
- Enhanced mobile sidebar with Resources section

### Fixed
- Node API URL configuration for production
- Menu organization and navigation clarity

## [1.3.0] - 2026-02-21

### Added
- Agent Teams API for multi-agent collaboration
- Team rental functionality for temporary agent access
- Workflow orchestration endpoints
- Enhanced session streaming with more event types
- NetworkPanel integration for real-time status

### Changed
- Improved onboarding flow with guided scenarios
- Updated templates gallery with new categories

## [1.2.0] - 2026-01-15

### Added
- Custom tools API for user-defined integrations
- Webhook triggers for automated agent execution
- Template instantiation endpoint
- Rate limit headers in API responses

### Changed
- Improved error response consistency
- Enhanced API documentation

### Fixed
- Session timeout issues on mobile devices
- Memory leak in long-running sessions

## [1.1.0] - 2025-12-01

### Added
- Blockchain publishing for agents (`/agents/{id}/publish`)
- Network agent search and execution
- Node status endpoint
- DSID (Decentralized Semantic Identity) support
- Trust tier system (T0-T4)

### Changed
- Improved agent execution performance
- Enhanced security for API endpoints

### Security
- Added rate limiting on all endpoints
- Implemented JWT token rotation

## [1.0.0] - 2025-11-01

### Added
- Initial platform release
- Core agents CRUD operations
- Session management and execution
- Basic metrics and analytics
- User authentication with OAuth2
- Web frontend with React
- FastAPI backend
- PostgreSQL database integration
- Redis caching layer

### Security
- SSL/TLS encryption for all traffic
- Secure password hashing
- API key management

---

## Version History Summary

| Version | Date | Highlights |
|---------|------|------------|
| 1.3.0 | Feb 2026 | Agent Teams, workflow orchestration |
| 1.2.0 | Jan 2026 | Custom tools, webhooks |
| 1.1.0 | Dec 2025 | Blockchain integration, DSID |
| 1.0.0 | Nov 2025 | Initial release |

## Upgrade Guide

### From 1.2.x to 1.3.x

No breaking changes. New Agent Teams API is additive.

```bash
# Update dependencies
npm install @resonantgenesis/sdk@latest
pip install --upgrade resonantgenesis
```

### From 1.1.x to 1.2.x

No breaking changes. Custom tools API is additive.

### From 1.0.x to 1.1.x

**Breaking Changes:**
- Agent publish endpoint moved from `/agents/publish` to `/agents/{id}/publish`
- Trust tier required for network operations

```python
# Old (1.0.x)
client.publish_agent(agent_data)

# New (1.1.x)
client.agents.get(agent_id).publish()
```

---

[Unreleased]: https://github.com/louienemesh/ResonantGenesis/compare/v1.3.0...HEAD
[1.3.0]: https://github.com/louienemesh/ResonantGenesis/compare/v1.2.0...v1.3.0
[1.2.0]: https://github.com/louienemesh/ResonantGenesis/compare/v1.1.0...v1.2.0
[1.1.0]: https://github.com/louienemesh/ResonantGenesis/compare/v1.0.0...v1.1.0
[1.0.0]: https://github.com/louienemesh/ResonantGenesis/releases/tag/v1.0.0
