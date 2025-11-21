# VWMP Spin-Off Preparation Guide

**Status:** Preparation - In Progress
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Created:** 2025-11-21
**Version:** v0.21.0.41+10
**GitHub Repository:** https://github.com/earlution/vwmp

---

## Executive Summary

This document tracks the preparation status for spinning off VWMP as a standalone open-source project on GitHub. The repository has been created and is ready for initial setup.

**GitHub Repository:** https://github.com/earlution/vwmp

**Current Status:**
- ✅ GitHub repository created
- ⏳ Repository setup (README, LICENSE, structure)
- ⏳ Code extraction and refactoring
- ⏳ Initial package structure
- ⏳ Documentation migration

---

## Repository Information

### GitHub Repository

**URL:** https://github.com/earlution/vwmp
**License:** MIT (already set up)
**Status:** Empty repository, ready for initial setup
**Owner:** earlution
**Public:** Yes (open-source)

### Repository Structure (Target)

```
vwmp/
├── LICENSE                    # MIT License (already present)
├── README.md                  # Project README (to be created)
├── pyproject.toml            # Python package configuration
├── setup.py                  # Package setup (optional)
├── requirements.txt          # Dependencies
├── requirements-dev.txt      # Development dependencies
├── .github/
│   └── workflows/           # CI/CD workflows
├── vwmp/                    # Core package
│   ├── __init__.py
│   ├── core/               # Framework-agnostic core
│   ├── plugins/            # Generic plugins
│   └── api/                # FastAPI API layer (optional)
├── vwmp_frontend/           # React frontend (optional)
├── docs/                    # Documentation
│   └── architecture/       # Architecture ADRs
├── examples/                # Example workflows
└── tests/                   # Test suite
```

---

## Pre-Spin-Off Checklist

### Phase 1: Architectural Cleanup ⏳ **REQUIRED**

**Status:** Not Started

**Tasks:**
1. **Move Project Handlers** ⏳
   - Move `KanbanUpdateHandler` and `ValidatorExecutionHandler` from `plugins/release/handlers.py` to `plugins/project/handlers.py`
   - Update plugin registry imports
   - Update handler registrations

2. **Remove Hardcoded Script Paths** ⏳
   - Make script paths fully configurable in step configuration
   - Update `KanbanUpdateHandler` to use subprocess with configurable paths
   - Update `ValidatorExecutionHandler` to use configurable validator paths

3. **Improve Project Plugin Isolation** ⏳
   - Create proper plugin structure in `plugins/project/`
   - Extract Kanban updater logic to separate module
   - Extract validator execution to separate module

**Estimated Effort:** 1-2 weeks
**Priority:** High (blocks clean separation)

**Reference:** [Spin-Off Readiness Analysis](E01-vwmp-spin-off-readiness-analysis.md)

---

### Phase 2: FastAPI Migration ⏳ **REQUIRED FOR LIGHTWEIGHTNESS**

**Status:** Not Started

**Tasks:**
1. Create FastAPI application structure
2. Port FastAPI REST API endpoints to FastAPI
3. Port WebSocket functionality (FastAPI Channels → FastAPI WebSocket)
4. Update frontend API client
5. Test end-to-end
6. Remove FastAPI API layer

**Estimated Effort:** 2-3 weeks
**Priority:** High (required for lightweightness targets)

**Reference:** [FastAPI Migration Plan](E01-vwmp-fastapi-migration-plan.md)

---

### Phase 3: Package Structure ⏳

**Status:** Not Started

**Tasks:**
1. **Create Standalone Package Structure**
   - Create `vwmp/` package (rename from `src/vwmp/`)
   - Move core engine to `vwmp/core/`
   - Move generic plugins to `vwmp/plugins/`
   - Create FastAPI API layer as `vwmp/api/`

2. **Package Configuration**
   - Create `pyproject.toml` with package metadata
   - Define dependencies (minimal - FastAPI, Uvicorn, PyYAML, jsonschema)
   - Configure entry points (CLI, server)

3. **Entry Points**
   - CLI entry point (`vwmp.cli`)
   - Server entry point (`vwmp.server`)
   - Headless mode support

**Estimated Effort:** 1-2 weeks
**Priority:** Medium (required for standalone package)

---

### Phase 4: Frontend Migration ⏳

**Status:** Not Started

**Tasks:**
1. **Set up Vite Project**
   - Create `vwmp_frontend/` directory
   - Set up Vite with React + React Flow
   - Migrate React components from CDN to npm modules

2. **Standalone SPA**
   - Create standalone SPA bundle
   - Update API client for FastAPI endpoints
   - Test standalone deployment

3. **Integration**
   - Serve SPA from FastAPI static files (optional)
   - Or deploy separately as standalone SPA

**Estimated Effort:** 1-2 weeks
**Priority:** Medium (required for web UI)

---

### Phase 5: Documentation Migration ⏳

**Status:** Not Started

**Tasks:**
1. **Extract VWMP Architecture Docs**
   - Copy all `E01-vwmp-*.md` files from `KB/Architecture/Standards_and_ADRs/`
   - Remove `E01-` prefix from filenames
   - Update internal references
   - Place in `docs/architecture/`

2. **Create Project README**
   - Project overview and goals
   - Installation instructions
   - Quick start guide
   - Architecture overview
   - Contributing guidelines

3. **Create Documentation Structure**
   - User guides
   - API documentation
   - Plugin development guide
   - Migration guide (from FastAPI to FastAPI)

**Estimated Effort:** 1 week
**Priority:** Medium (important for adoption)

**Reference:** [Documentation Index](E01-vwmp-documentation-index.md)

---

### Phase 6: CI/CD Setup ⏳

**Status:** Not Started

**Tasks:**
1. **GitHub Actions Workflows**
   - Python package tests
   - Frontend build and tests
   - Linting and type checking
   - Security scanning
   - Package publishing to PyPI

2. **Testing**
   - Unit tests for core engine
   - Integration tests for API
   - Frontend tests
   - End-to-end tests

3. **Release Process**
   - Automated version bumping
   - Changelog generation
   - Tag creation
   - PyPI publishing

**Estimated Effort:** 1 week
**Priority:** Medium (important for maintenance)

---

## Repository Setup Tasks

### Initial Repository Setup

**Status:** Repository created, ready for content

**Next Steps:**
1. **Create README.md** ⏳
   - Project description
   - Installation instructions
   - Quick start guide
   - Links to documentation

2. **Create LICENSE** ✅
   - MIT License already present

3. **Create .gitignore** ⏳
   - Python ignores (venv, __pycache__, .pyc)
   - Node ignores (node_modules, dist)
   - IDE ignores (.vscode, .idea)
   - OS ignores (.DS_Store)

4. **Create Initial Structure** ⏳
   - Package directories
   - Documentation structure
   - Example workflows

5. **Add GitHub Templates** ⏳
   - Issue templates
   - Pull request template
   - Contributing guidelines

---

## Extraction Process

### Step 1: Extract Core Engine

**Source:** `src/vwmp/core/`
**Target:** `vwmp/core/`

**Commands:**
```bash
# In standalone repository
mkdir -p vwmp/core
cp -r src/vwmp/core/* vwmp/core/

# Update imports
# from src.vwmp.core import ...
# → from vwmp.core import ...
```

---

### Step 2: Extract Generic Plugins

**Source:** `src/vwmp/plugins/{git,release,kanban}/`
**Target:** `vwmp/plugins/{git,release,kanban}/`

**Note:** Extract only generic plugins, exclude Project-specific handlers.

**Commands:**
```bash
# Generic plugins only
mkdir -p vwmp/plugins
cp -r src/vwmp/plugins/git vwmp/plugins/
cp -r src/vwmp/plugins/release vwmp/plugins/  # But exclude Project handlers
cp -r src/vwmp/plugins/kanban vwmp/plugins/  # Review if generic enough
```

---

### Step 3: Extract API Layer (FastAPI)

**Source:** `vwmp/api/` (to be created)
**Target:** `vwmp/api/`

**Note:** Extract FastAPI API layer after migration from FastAPI.

---

### Step 4: Extract Frontend

**Source:** `src/vwmp_web/vwmp/templates/vwmp/` (React components)
**Target:** `vwmp_frontend/src/`

**Note:** Extract after migration to Vite/npm.

---

### Step 5: Extract Documentation

**Source:** `KB/Architecture/Standards_and_ADRs/E01-vwmp-*.md`
**Target:** `docs/architecture/`

**Commands:**
```bash
# Copy all VWMP architecture docs
mkdir -p docs/architecture
cp KB/Architecture/Standards_and_ADRs/E01-vwmp-*.md docs/architecture/

# Remove E01- prefix
cd docs/architecture
for file in E01-vwmp-*.md; do
  mv "$file" "${file#E01-vwmp-}"
done

# Update references in docs
# E01-vwmp-fastapi-decision.md → vwmp-fastapi-decision.md
```

---

## Package Metadata

### pyproject.toml (To Be Created)

```toml
[build-system]
requires = ["setuptools>=61.0", "wheel"]
build-backend = "setuptools.build_meta"

[project]
name = "vwmp"
version = "0.1.0"
description = "Visual Workflow Management Platform - A lightweight, extensible workflow engine"
readme = "README.md"
requires-python = ">=3.13"
license = {text = "MIT"}
authors = [
    {name = "Ruari Mears", email = "..."}
]
classifiers = [
    "Development Status :: 3 - Alpha",
    "Intended Audience :: Developers",
    "License :: OSI Approved :: MIT License",
    "Programming Language :: Python :: 3",
    "Programming Language :: Python :: 3.13",
]
dependencies = [
    "fastapi>=0.100.0",
    "uvicorn[standard]>=0.23.0",
    "pydantic>=2.0.0",
    "pyyaml>=6.0",
    "jsonschema>=4.0.0",
]

[project.optional-dependencies]
dev = [
    "pytest>=7.0.0",
    "pytest-asyncio>=0.21.0",
    "black>=23.0.0",
    "mypy>=1.0.0",
]

[project.scripts]
vwmp = "vwmp.cli:main"
vwmp-server = "vwmp.server:main"

[tool.setuptools]
packages = ["vwmp"]
```

---

## Initial Repository Content

### README.md (To Be Created)

**Content Outline:**
1. Project Overview
   - What is VWMP?
   - Key features
   - Use cases

2. Quick Start
   - Installation
   - Basic usage
   - Example workflow

3. Architecture
   - High-level overview
   - Components
   - Plugin system

4. Documentation
   - User guides
   - API documentation
   - Plugin development

5. Contributing
   - How to contribute
   - Development setup
   - Code of conduct

6. License
   - MIT License

---

## Migration Timeline

| Phase | Tasks | Duration | Status |
|-------|-------|----------|--------|
| **Phase 1** | Architectural Cleanup | 1-2 weeks | ⏳ Not Started |
| **Phase 2** | FastAPI Migration | 2-3 weeks | ⏳ Not Started |
| **Phase 3** | Package Structure | 1-2 weeks | ⏳ Not Started |
| **Phase 4** | Frontend Migration | 1-2 weeks | ⏳ Not Started |
| **Phase 5** | Documentation Migration | 1 week | ⏳ Not Started |
| **Phase 6** | CI/CD Setup | 1 week | ⏳ Not Started |
| **Total** | **All Phases** | **7-11 weeks** | ⏳ Not Started |

---

## Success Criteria

**VWMP is ready for spin-off when:**

1. ✅ **Architectural Cleanup Complete**
   - Project handlers moved to proper location
   - No hardcoded script paths
   - Project plugin properly isolated

2. ✅ **FastAPI Migration Complete**
   - All API endpoints ported to FastAPI
   - WebSocket working
   - Performance targets met (<50MB memory, <1s startup)

3. ✅ **Package Structure Ready**
   - Standalone `vwmp/` package structure
   - Entry points configured (CLI, server)
   - Headless mode working

4. ✅ **Documentation Migrated**
   - All architecture docs extracted
   - README.md created
   - Documentation structure complete

5. ✅ **CI/CD Working**
   - Tests passing
   - Automated releases
   - Package publishing ready

6. ✅ **GitHub Repository Ready**
   - Repository created ✅
   - Initial content added
   - License present ✅
   - README.md created
   - Contributing guidelines

---

## Next Steps

1. **Complete Architectural Cleanup** (Phase 1)
   - Move Project handlers
   - Remove hardcoded paths
   - Improve plugin isolation

2. **Begin FastAPI Migration** (Phase 2)
   - Create FastAPI structure
   - Port endpoints incrementally
   - Test with existing frontend

3. **Create Initial Repository Content**
   - README.md
   - .gitignore
   - Package structure
   - Example workflows

---

## Related Documents

- **[Spin-Off Readiness Analysis](E01-vwmp-spin-off-readiness-analysis.md)** - Current state assessment
- **[FastAPI Migration Plan](E01-vwmp-fastapi-migration-plan.md)** - FastAPI migration guide
- **[FastAPI Decision ADR](E01-vwmp-fastapi-decision.md)** - Technology choice rationale
- **[Documentation Index](E01-vwmp-documentation-index.md)** - All VWMP docs for extraction

---

## Repository Links

- **GitHub Repository:** https://github.com/earlution/vwmp
- **License:** MIT (already configured)
- **Status:** Empty, ready for initial setup

---

**Last Updated:** 2025-11-21
**Preparation Version:** 1.0.0
