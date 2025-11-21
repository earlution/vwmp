# Epic 01: Phase 2 - Visual Workflow Designer UI Implementation

**Status:** PLANNING
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Phase:** Phase 2 - Frontend Implementation
**Created:** 2025-11-20
**Version:** v0.21.0.13+

---

## Executive Summary

This document outlines the implementation plan for the **Visual Workflow Designer UI**, the missing frontend component of the VWMP platform. While the backend API, execution engine, and plugin system are complete, the actual drag-and-drop visual designer interface has not yet been implemented.

---

## Current State Analysis

### ✅ What's Complete

1. **Backend Infrastructure** (100% complete)
   - ✅ Django REST API endpoints (`/api/vwmp/workflows/`)
   - ✅ Workflow execution engine
   - ✅ Plugin system and registry
   - ✅ Workflow storage and versioning
   - ✅ Monitoring and progress tracking (backend)

2. **Architecture & Design** (100% complete)
   - ✅ Platform architecture (C4 views)
   - ✅ Plugin architecture design
   - ✅ UI/UX design document (T003)
   - ✅ Integration architecture

3. **Basic Dashboard** (Partial)
   - ✅ Simple HTML dashboard listing workflows
   - ✅ Statistics cards
   - ✅ Workflow cards with basic info
   - ❌ No visual designer interface
   - ❌ No drag-and-drop functionality
   - ❌ No workflow editor

### ❌ What's Missing (The Actual Visual Designer)

1. **Visual Workflow Canvas**
   - Drag-and-drop step placement
   - Visual step connections (arrows/edges)
   - Zoom and pan controls
   - Mini-map for navigation
   - Grid background and snap-to-grid

2. **Step Palette**
   - Browse available step types
   - Search/filter step types
   - Categorized step types
   - Template workflows
   - Drag-to-canvas functionality

3. **Step Configuration Panel**
   - Dynamic form generation from step schema
   - Parameter input and validation
   - Dependency selection UI
   - Advanced options (collapsible)
   - Real-time validation feedback

4. **Visual Execution Monitor**
   - Real-time step status visualization
   - Progress bars and execution logs
   - Error display and recovery
   - Execution history viewer

5. **Workflow Editor/Viewer**
   - Create new workflows visually
   - Edit existing workflows
   - Workflow preview mode
   - Save/load workflows
   - Export/import workflows

---

## Implementation Plan

### Phase 2.1: Technology Stack Decision (2-4 hours)

**Goal:** Choose frontend framework and libraries for visual designer

**Options:**
1. **React + React Flow** (Recommended)
   - React Flow is mature and feature-rich
   - Good community support
   - Extensive documentation
   - Works well with Django REST API

2. **Vue.js + Vue Flow**
   - Vue Flow is newer but capable
   - Vue.js has smaller learning curve
   - Less ecosystem maturity

3. **Vanilla JavaScript + Custom Canvas**
   - Full control, no dependencies
   - Higher development effort
   - More maintenance burden

**Recommendation:** React + React Flow
- Better fit for complex interactive UI
- React Flow has proven track record
- Can be integrated with Django via CDN or npm

**Deliverable:** Technology decision document with rationale

---

### Phase 2.2: Visual Canvas Foundation (8-16 hours)

**Tasks:**
1. Set up React environment (or CDN-based React)
2. Integrate React Flow library
3. Create basic canvas component
4. Implement zoom and pan controls
5. Add grid background
6. Implement mini-map
7. Create basic step node component

**Deliverable:** Working canvas with zoom/pan, grid, and basic nodes

**Dependencies:** Phase 2.1 complete

---

### Phase 2.3: Step Palette Component (6-12 hours)

**Tasks:**
1. Fetch available step types from API (`/api/vwmp/step-handlers/`)
2. Create step palette UI component
3. Implement step type categorization
4. Add search/filter functionality
5. Implement drag-to-canvas functionality
6. Add template workflows section
7. Style step type cards with icons

**Deliverable:** Functional step palette with drag-and-drop

**Dependencies:** Phase 2.2 complete (canvas must accept dropped steps)

---

### Phase 2.4: Step Configuration Panel (12-20 hours)

**Tasks:**
1. Create dynamic form generator from step schema
2. Implement form field types (text, select, checkbox, etc.)
3. Add parameter validation (client-side)
4. Create dependency selector UI
5. Implement advanced options (collapsible)
6. Add help text and tooltips
7. Create save/cancel actions
8. Connect to API for step updates

**Deliverable:** Functional step configuration panel

**Dependencies:** Phase 2.2 complete (panel opens when step selected)

---

### Phase 2.5: Workflow Editor Integration (10-18 hours)

**Tasks:**
1. Connect canvas to workflow API (CRUD operations)
2. Implement workflow loading from API
3. Implement workflow saving to API
4. Add workflow metadata editing (name, description, tags)
5. Create workflow validation UI
6. Add "New Workflow" functionality
7. Implement workflow deletion
8. Add undo/redo functionality (optional)

**Deliverable:** Full workflow CRUD operations from visual editor

**Dependencies:** Phase 2.2, 2.3, 2.4 complete

---

### Phase 2.6: Visual Execution Monitor (8-16 hours)

**Tasks:**
1. Connect to execution API (`/api/vwmp/workflows/<id>/execute/`)
2. Implement real-time step status updates (polling or WebSocket)
3. Visualize step status on canvas (color-coded borders)
4. Create execution log panel
5. Add progress bars
6. Implement error display
7. Add execution history viewer
8. Create "Stop Execution" functionality

**Deliverable:** Real-time execution monitoring UI

**Dependencies:** Phase 2.2 complete, backend execution engine working

---

### Phase 2.7: Polish & UX Improvements (6-12 hours)

**Tasks:**
1. Add keyboard shortcuts
2. Improve accessibility (WCAG 2.2 AA)
3. Add context menus (right-click)
4. Implement workflow templates UI
5. Add export/import functionality
6. Improve error messages and validation feedback
7. Add loading states and transitions
8. Responsive design for tablet devices

**Deliverable:** Polished, production-ready visual designer

**Dependencies:** All previous phases complete

---

## Effort Estimates

| Phase | Task | Estimated Hours | Priority |
|-------|------|----------------|----------|
| 2.1 | Technology Stack Decision | 2-4 | High |
| 2.2 | Visual Canvas Foundation | 8-16 | High |
| 2.3 | Step Palette Component | 6-12 | High |
| 2.4 | Step Configuration Panel | 12-20 | High |
| 2.5 | Workflow Editor Integration | 10-18 | High |
| 2.6 | Visual Execution Monitor | 8-16 | Medium |
| 2.7 | Polish & UX Improvements | 6-12 | Medium |
| **Total** | | **52-98 hours** | |

---

## Implementation Strategy

### Option A: React + React Flow (Recommended)
- **Approach:** Use React via CDN (no build step) or npm/Webpack
- **Pros:**
  - React Flow is mature and feature-rich
  - Good documentation and examples
  - Can start quickly with CDN
- **Cons:**
  - Requires React knowledge
  - Build setup needed for production (if not CDN)

### Option B: Progressive Enhancement
- **Approach:** Start with vanilla JS, enhance with React later
- **Pros:**
  - No framework dependency initially
  - Works immediately
- **Cons:**
  - More work to convert later
  - Harder to maintain complex UI

### Option C: Vue.js + Vue Flow
- **Approach:** Similar to React but Vue ecosystem
- **Pros:**
  - Simpler syntax
  - Smaller learning curve
- **Cons:**
  - Less mature than React Flow
  - Fewer examples

**Recommendation:** **Option A - React + React Flow via CDN initially**, migrate to npm/Webpack for production if needed.

---

## Technical Considerations

### 1. React Integration with Django

**Option 1: CDN-based React (Quick Start)**
```html
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>
<script crossorigin src="https://unpkg.com/reactflow@11/dist/umd/index.js"></script>
```
- ✅ No build step required
- ✅ Quick to start
- ❌ Less optimal for production

**Option 2: npm/Webpack Build (Production)**
- ✅ Optimized builds
- ✅ Tree shaking
- ✅ Better performance
- ❌ Requires build pipeline

**Recommendation:** Start with CDN for rapid prototyping, migrate to npm if needed.

### 2. API Integration

- Use `fetch()` API for all API calls
- Handle authentication (CSRF tokens for Django)
- Error handling and loading states
- Real-time updates via polling (WebSocket can be added later)

### 3. State Management

- Start with React state (`useState`, `useReducer`)
- Add context if needed for global state
- No need for Redux initially

### 4. Styling

- Use CSS-in-JS (styled-components) or CSS modules
- Or traditional CSS with BEM methodology
- Ensure responsive design for tablets

---

## Success Criteria

### MVP (Minimum Viable Product)
- ✅ Create workflow by dragging steps from palette
- ✅ Connect steps visually (dependencies)
- ✅ Configure step parameters
- ✅ Save workflow to backend
- ✅ Execute workflow from UI
- ✅ See basic execution status

### Full Implementation
- ✅ All MVP features
- ✅ Real-time execution monitoring
- ✅ Workflow templates
- ✅ Export/import workflows
- ✅ Keyboard shortcuts
- ✅ Accessibility (WCAG 2.2 AA)
- ✅ Responsive design
- ✅ Error handling and validation

---

## Next Steps

1. **Review and approve this plan**
2. **Make technology stack decision** (Phase 2.1)
3. **Set up development environment** (React + React Flow)
4. **Begin Phase 2.2: Visual Canvas Foundation**
5. **Iterate through remaining phases**

---

## Related Documents

- [Epic 01 Overview](../Epic-21.md)
- [Epic 01 Development Plan](Epic-21-Development-Plan.md)
- [VWMP UI/UX Design](../../../Architecture/Standards_and_ADRs/E01-vwmp-visual-designer-uiux.md)
- [VWMP Platform Architecture](../../../Architecture/Standards_and_ADRs/E01-vwmp-platform-architecture.md)

---

## Notes

- This phase focuses exclusively on the **frontend visual designer UI**
- Backend API is already complete and functional
- Can iterate quickly with React Flow CDN approach
- Testing should be done in browser as we build
- Consider creating a separate branch for this phase: `epic/21-workflow-platform-frontend`
