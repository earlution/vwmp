# Epic 01: VWMP Frontend Technology Stack Decision

**Status:** Approved
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Phase:** Phase 2.1 - Technology Stack Decision
**Created:** 2025-11-20
**Version:** v0.21.0.14+

---

## Decision

**Selected:** React + React Flow (via CDN initially)

---

## Options Evaluated

### Option 1: React + React Flow (✅ Selected)

**Technology Stack:**
- **React:** JavaScript library for building user interfaces
- **React Flow:** React library for building node-based editors and interactive diagrams
- **Integration:** CDN-based (no build step) initially, can migrate to npm/Webpack later

**Pros:**
- ✅ React Flow is mature and battle-tested (v11+)
- ✅ Excellent documentation and community support
- ✅ Built specifically for node-based visual editors
- ✅ Supports drag-and-drop, zoom, pan, mini-map out of the box
- ✅ Easy to integrate with FastAPI REST API via `fetch()`
- ✅ Can start with CDN (no build pipeline needed initially)
- ✅ Large ecosystem of React components and libraries
- ✅ Good TypeScript support (for future migration)

**Cons:**
- ❌ Requires React knowledge (but minimal for CDN approach)
- ❌ CDN version is less optimized (but fine for initial development)
- ❌ Production builds may need npm/Webpack setup (later optimization)

**CDN Approach:**
```html
<!-- React (v18) -->
<script crossorigin src="https://unpkg.com/react@18/umd/react.production.min.js"></script>
<script crossorigin src="https://unpkg.com/react-dom@18/umd/react-dom.production.min.js"></script>

<!-- React Flow (v11) -->
<script crossorigin src="https://unpkg.com/reactflow@11/dist/umd/index.js"></script>

<!-- Babel Standalone (for JSX in browser) -->
<script src="https://unpkg.com/@babel/standalone/babel.min.js"></script>
```

**Alternative (npm/Webpack) for Production:**
- Can migrate to npm/Webpack build pipeline later if needed
- Better tree-shaking and optimization
- TypeScript support

---

### Option 2: Vue.js + Vue Flow

**Technology Stack:**
- **Vue.js:** Progressive JavaScript framework
- **Vue Flow:** Vue.js port of React Flow

**Pros:**
- ✅ Simpler syntax and smaller learning curve
- ✅ Good for rapid prototyping
- ✅ Less boilerplate than React

**Cons:**
- ❌ Vue Flow is newer and less mature than React Flow
- ❌ Smaller ecosystem and fewer examples
- ❌ Less community support
- ❌ CDN integration more complex

**Verdict:** Not selected - React Flow is more mature and proven

---

### Option 3: Vanilla JavaScript + Custom Canvas

**Technology Stack:**
- **Vanilla JavaScript:** No frameworks
- **HTML5 Canvas or SVG:** Custom rendering

**Pros:**
- ✅ No dependencies
- ✅ Full control over implementation
- ✅ Smaller bundle size

**Cons:**
- ❌ Much higher development effort (200+ hours vs 50-100 hours)
- ❌ Need to implement drag-and-drop, zoom, pan, connections from scratch
- ❌ More maintenance burden
- ❌ Slower time to market
- ❌ No proven patterns to follow

**Verdict:** Not selected - Would significantly extend timeline and complexity

---

## Decision Rationale

### Primary Reasons

**Decision Drivers:**
1. **Purpose-Built for Visual Editors** - React Flow was specifically designed for node-based visual editors
2. **Maturity & Stability** - Proven track record in production, active community
3. **Rapid Development** - CDN approach enables immediate start without build pipeline
4. **Future Flexibility** - Can migrate to npm/Vite later for production optimization
5. **Standalone Ready** - Can be deployed as standalone SPA or integrated into any backend

### Detailed Rationale

1. **React Flow is Purpose-Built:**
   - React Flow was specifically designed for node-based visual editors
   - Includes drag-and-drop, zoom, pan, mini-map, connections, and more out of the box
   - This is exactly what we need for the visual workflow designer
   - **Alternative would require custom implementation** (200+ hours vs 50-100 hours)

2. **Maturity and Proven Track Record:**
   - React Flow has been used in production by many companies
   - Large, active community with regular updates
   - Comprehensive documentation and examples
   - Regular bug fixes and feature updates
   - **Lower risk** compared to building from scratch or using less mature alternatives

3. **Rapid Development (Time to Market):**
   - CDN approach allows immediate start (no build setup)
   - React Flow provides most features we need out of the box
   - Significantly faster than building from scratch (weeks vs months)
   - **Enable faster iteration** during development

4. **Standalone Deployment Support:**
   - React + React Flow can be deployed as standalone SPA
   - Works with any REST API backend (not tied to FastAPI)
   - Easy to migrate from CDN to npm/Vite for production
   - **Aligns with spin-off objective** - easy to extract frontend

5. **Future Flexibility:**
   - Can migrate to npm/Vite for production optimization if needed
   - Can add TypeScript later if desired
   - Can integrate other React libraries as needed
   - **Maintains development velocity** while allowing optimization later

6. **Alignment with Project Standards:**
   - Project already uses CDN libraries (Chart.js via CDN)
   - Progressive enhancement approach (CDN React fits this pattern)
   - Consistent with existing development patterns
   - **Familiar approach** for team

### Why Not Alternatives?

**Vue.js + Vue Flow:**
- ❌ Vue Flow is newer and less mature than React Flow
- ❌ Smaller ecosystem and fewer examples
- ❌ Less community support
- ❌ **Higher risk** for critical visual editor feature

**Vanilla JavaScript + Custom Canvas:**
- ❌ Much higher development effort (200+ hours vs 50-100 hours)
- ❌ Need to implement drag-and-drop, zoom, pan, connections from scratch
- ❌ More maintenance burden
- ❌ **Significantly slower time to market**

**Rationale:** React Flow provides purpose-built functionality, proven stability, and rapid development - all critical for delivering the visual workflow designer efficiently.

---

## Implementation Strategy

### Phase 1: CDN-Based (Initial Development)
- Use React and React Flow via CDN
- JSX via Babel Standalone in browser
- No build step required
- Fast iteration and prototyping

**Benefits:**
- ✅ Start immediately
- ✅ No npm/node dependencies
- ✅ Easy to test in browser
- ✅ Quick prototyping

### Phase 2: npm/Webpack (Production Optimization - Optional)
- Migrate to npm-based build if needed
- Webpack for bundling and optimization
- Tree-shaking for smaller bundles
- TypeScript support

**Benefits:**
- ✅ Smaller bundle size
- ✅ Better performance
- ✅ Type safety (TypeScript)
- ✅ Better development experience

**Decision:** Start with CDN, migrate to npm only if bundle size or performance becomes an issue.

---

## Integration with FastAPI

### Approach
1. **FastAPI Template:** Base template with React/React Flow CDN scripts
2. **React Components:** Inline JSX with Babel Standalone, or separate `.js` files
3. **API Integration:** Use `fetch()` API to connect to FastAPI REST API
4. **CSRF Protection:** Include FastAPI CSRF token in API requests
5. **Static Files:** Serve component JS files via FastAPI staticfiles if needed

### Example Structure
```
src/vwmp_web/vwmp/
├── templates/
│   └── vwmp/
│       ├── dashboard.html (existing)
│       └── designer.html (new - visual designer)
├── static/
│   └── vwmp/
│       ├── js/
│       │   ├── designer.js (React components)
│       │   └── api-client.js (API integration)
│       └── css/
│           └── designer.css (styles)
```

---

## Risk Assessment

### Low Risk ✅
- React Flow is mature and stable
- CDN approach is well-supported
- FastAPI + React integration is common pattern

### Mitigation
- Start with CDN for rapid prototyping
- Can migrate to npm/Webpack if needed
- Comprehensive testing at each phase

---

## Success Criteria

✅ **Technology Decision Complete When:**
- React + React Flow selected and documented
- Initial CDN integration working
- Basic canvas rendering in browser
- Ready for Phase 2.2 (Canvas Foundation)

---

## Next Steps

1. ✅ Document technology decision (this document)
2. Create initial React + React Flow integration
3. Set up FastAPI template with CDN scripts
4. Create basic canvas component
5. Test in browser
6. Proceed to Phase 2.2: Visual Canvas Foundation

---

## References

- [React Documentation](https://react.dev/)
- [React Flow Documentation](https://reactflow.dev/)
- [React Flow Examples](https://reactflow.dev/examples/)
- [FastAPI Static Files](https://fastapi.tiangolo.com/en/5.2/howto/static-files/)
- [Babel Standalone](https://babeljs.io/docs/en/babel-standalone)

---

## Approval

**Status:** ✅ Approved
**Date:** 2025-11-20
**Version:** v0.21.0.14
