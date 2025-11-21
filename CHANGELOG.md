# Changelog

All notable changes to this project will be documented in this file.

The format is based on [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
and this project adheres to [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

## [0.0.0+5] - 2025-11-21

### Added
- API Design Specification (E01-vwmp-api-design.md)
  - REST API endpoints for workflow management (CRUD)
  - Execution endpoints (execute, status, cancel)
  - Plugin discovery endpoints
  - WebSocket protocol for real-time updates
  - Authentication, authorization, rate limiting, pagination
- Security Architecture (E01-vwmp-security-architecture.md)
  - Authentication methods (JWT, Session-based)
  - Authorization (RBAC with roles)
  - Input validation layers
  - Plugin security and isolation
  - Secure execution (workspace isolation, command validation)
  - Network security (HTTPS, CORS, WebSocket security)
  - Audit logging and vulnerability management

### Changed
- Updated API Design document: removed Django references, updated file paths
- Updated Security Architecture document: removed Django-specific references

## [0.0.0+4] - 2025-11-21

### Changed
- Updated all documentation references project-wide to be contextualized to vwmp project
- Removed all Confidentia references from documentation
- Updated file paths: `src/confidentia/` → `src/vwmp/`, `src/confidentia_django/` → `src/vwmp_web/`
- Updated handler names: `confidentia.*` → `vwmp.*`
- Updated GitHub repository links: `github.com/earlution/confidentia` → `github.com/earlution/vwmp`
- Updated Epic references: `Epic-21` → `Epic-01`
- Removed Django references except in tech stack evaluation documents
- Updated framework references: Django → FastAPI (except in evaluation contexts)
- Updated main README with KB documentation section and links

## [0.0.0+3] - 2025-11-21

### Added
- Knowledge Base (KB/) directory with comprehensive documentation
- Architecture documentation (19 files):
  - Standards & ADRs (12 files): Platform architecture, plugin architecture, tech stack decisions, FastAPI migration plan, spin-off preparation
  - C4 Architecture views (4 files): Context, Container, Component views and runtime sequences
  - Integration Architecture (3 files): Integration architecture, gap analysis, completion summary
- Developer documentation (10 files):
  - Documentation hub and tutorials (release workflow, kanban review workflow)
  - Reference guides and user guides
  - Best practices, configuration examples, troubleshooting
- Project Management documentation (7 files):
  - Epic 01 documentation (3 files): Main epic, development plan, phase 2 plan
  - Story documentation (4 files): All Epic 01 stories

## [0.0.0+2] - 2025-11-21

### Added
- Project description and contextualization
- GitHub topics reference file
- Updated README with VWMP project details
- Updated Built With section (Python, FastAPI, React)
- Updated Getting Started and Usage sections

## [0.0.0+1] - 2025-11-21

### Added
- Initial project setup
- README template from Best-README-Template
- Shields.io badges integration
- Keep a Changelog format
- Version file (VERSION)

## [0.0.0+0] - 2025-01-XX

### Added
- Initial project structure
- MIT License
- Git repository setup

[Unreleased]: https://github.com/earlution/vwmp/compare/v0.0.0+5...HEAD
[0.0.0+5]: https://github.com/earlution/vwmp/compare/v0.0.0+4...v0.0.0+5
[0.0.0+4]: https://github.com/earlution/vwmp/compare/v0.0.0+3...v0.0.0+4
[0.0.0+3]: https://github.com/earlution/vwmp/compare/v0.0.0+2...v0.0.0+3
[0.0.0+2]: https://github.com/earlution/vwmp/compare/v0.0.0+1...v0.0.0+2
[0.0.0+1]: https://github.com/earlution/vwmp/compare/v0.0.0+0...v0.0.0+1
[0.0.0+0]: https://github.com/earlution/vwmp/releases/tag/v0.0.0+0

