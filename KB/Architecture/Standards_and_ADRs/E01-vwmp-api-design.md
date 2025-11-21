# VWMP API Design Specification

**Status:** Draft - Design
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Created:** 2025-11-21
**Version:** v0.21.0.41+12

---

## Executive Summary

This document defines the REST API and WebSocket interface for the Visual Workflow Management Platform (VWMP). The API provides workflow management (CRUD), execution control, real-time monitoring, and plugin discovery capabilities.

**API Framework:** FastAPI
**API Versioning:** `/api/vwmp/v1/` (future), currently `/api/vwmp/`
**Authentication:** JWT or session-based (configurable)
**Content-Type:** `application/json`

---

## API Endpoints

### Base URL

```
/api/vwmp/
```

---

## Workflow Management Endpoints

### List Workflows

**GET** `/api/vwmp/workflows/`

**Description:** Retrieve list of all workflows with metadata.

**Query Parameters:**
- `type` (optional): Filter by workflow type (e.g., `release`, `kanban_review`)
- `limit` (optional): Maximum number of results (default: 50)
- `offset` (optional): Pagination offset (default: 0)

**Response:** `200 OK`

```json
{
  "count": 10,
  "results": [
    {
      "id": "53dd7364-c347-40de-9583-11b071dcaf95",
      "name": "Release Workflow",
      "version": "1.0.0",
      "type": "release",
      "description": "Automated release workflow",
      "created_at": "2025-11-20T10:00:00Z",
      "updated_at": "2025-11-21T15:30:00Z",
      "last_executed": "2025-11-21T14:00:00Z",
      "execution_count": 42
    }
  ]
}
```

---

### Get Workflow

**GET** `/api/vwmp/workflows/{workflow_id}/`

**Description:** Retrieve complete workflow definition.

**Path Parameters:**
- `workflow_id` (required): UUID of workflow

**Response:** `200 OK`

```json
{
  "id": "53dd7364-c347-40de-9583-11b071dcaf95",
  "name": "Release Workflow",
  "version": "1.0.0",
  "type": "release",
  "description": "Automated release workflow",
  "config": {
    "version_file": "VERSION",
    "changelog_dir": "KB/Changelog_and_Release_Notes/Changelog_Archive"
  },
  "steps": [
    {
      "id": "step-1",
      "name": "Bump Version",
      "type": "version_bump",
      "handler": "release.version_bump",
      "config": {
        "increment_type": "patch",
        "version_file": "${config.version_file}"
      },
      "required": true,
      "enabled": true,
      "dependencies": []
    }
  ],
  "parameters": [
    {
      "name": "summary",
      "type": "string",
      "required": true,
      "label": "Release Summary",
      "description": "Brief description of the release"
    }
  ],
  "created_at": "2025-11-20T10:00:00Z",
  "updated_at": "2025-11-21T15:30:00Z"
}
```

**Error Responses:**
- `404 Not Found`: Workflow not found

```json
{
  "error": "Workflow not found",
  "workflow_id": "53dd7364-c347-40de-9583-11b071dcaf95"
}
```

---

### Create Workflow

**POST** `/api/vwmp/workflows/`

**Description:** Create a new workflow definition.

**Request Body:**

```json
{
  "name": "New Workflow",
  "version": "1.0.0",
  "type": "release",
  "description": "Workflow description",
  "config": {},
  "steps": [],
  "parameters": []
}
```

**Response:** `201 Created`

```json
{
  "id": "new-uuid-here",
  "name": "New Workflow",
  "version": "1.0.0",
  "type": "release",
  "description": "Workflow description",
  "created_at": "2025-11-21T16:00:00Z"
}
```

**Error Responses:**
- `400 Bad Request`: Validation error

```json
{
  "error": "Validation failed",
  "errors": {
    "name": ["This field is required."],
    "steps": ["Workflow must have at least one step."]
  }
}
```

---

### Update Workflow

**PUT** `/api/vwmp/workflows/{workflow_id}/`

**Description:** Update existing workflow definition.

**Path Parameters:**
- `workflow_id` (required): UUID of workflow

**Request Body:** Same as Create Workflow

**Response:** `200 OK` (same format as Get Workflow)

**Error Responses:**
- `404 Not Found`: Workflow not found
- `400 Bad Request`: Validation error

---

### Delete Workflow

**DELETE** `/api/vwmp/workflows/{workflow_id}/`

**Description:** Delete a workflow definition.

**Path Parameters:**
- `workflow_id` (required): UUID of workflow

**Response:** `204 No Content`

**Error Responses:**
- `404 Not Found`: Workflow not found
- `409 Conflict`: Workflow is currently executing

```json
{
  "error": "Cannot delete workflow: currently executing",
  "workflow_id": "53dd7364-c347-40de-9583-11b071dcaf95",
  "execution_id": "execution-uuid"
}
```

---

## Execution Endpoints

### Execute Workflow

**POST** `/api/vwmp/workflows/{workflow_id}/execute/`

**Description:** Execute a workflow with provided parameters.

**Path Parameters:**
- `workflow_id` (required): UUID of workflow

**Request Body:**

```json
{
  "parameters": {
    "summary": "Documentation: Add API design specification",
    "type": "documentation",
    "kb_docs_ok": true
  },
  "async": true
}
```

**Query Parameters:**
- `async` (optional): Execute asynchronously and return immediately (default: `true`)

**Response:** `202 Accepted` (async) or `200 OK` (sync)

**Async Response:**

```json
{
  "execution_id": "execution-uuid",
  "workflow_id": "53dd7364-c347-40de-9583-11b071dcaf95",
  "status": "pending",
  "started_at": "2025-11-21T16:00:00Z",
  "websocket_url": "ws://localhost:8000/api/vwmp/executions/{execution_id}/"
}
```

**Sync Response:**

```json
{
  "execution_id": "execution-uuid",
  "workflow_id": "53dd7364-c347-40de-9583-11b071dcaf95",
  "status": "success",
  "started_at": "2025-11-21T16:00:00Z",
  "completed_at": "2025-11-21T16:00:30Z",
  "total_time": 30.5,
  "step_results": {
    "step-1": {
      "step_id": "step-1",
      "status": "success",
      "output": {"new_version": "0.21.0.41+12"},
      "execution_time": 0.5
    }
  }
}
```

**Error Responses:**
- `404 Not Found`: Workflow not found
- `400 Bad Request`: Parameter validation error

```json
{
  "error": "Parameter validation failed",
  "errors": {
    "summary": ["This field is required."],
    "kb_docs_ok": ["Must be boolean."]
  }
}
```

---

### Get Execution Status

**GET** `/api/vwmp/executions/{execution_id}/`

**Description:** Get execution status and results.

**Path Parameters:**
- `execution_id` (required): UUID of execution

**Response:** `200 OK`

```json
{
  "execution_id": "execution-uuid",
  "workflow_id": "53dd7364-c347-40de-9583-11b071dcaf95",
  "workflow_name": "Release Workflow",
  "status": "running",
  "progress": {
    "total_steps": 10,
    "completed_steps": 3,
    "failed_steps": 0,
    "percentage": 30
  },
  "started_at": "2025-11-21T16:00:00Z",
  "completed_at": null,
  "step_results": {
    "step-1": {
      "step_id": "step-1",
      "step_name": "Bump Version",
      "status": "success",
      "output": {"new_version": "0.21.0.41+12"},
      "execution_time": 0.5,
      "logs": ["Incremented version: 0.21.0.41+11 → 0.21.0.41+12"]
    },
    "step-2": {
      "step_id": "step-2",
      "step_name": "Create Detailed Changelog",
      "status": "running",
      "execution_time": 1.2,
      "logs": ["Creating changelog file..."]
    }
  }
}
```

**Error Responses:**
- `404 Not Found`: Execution not found

---

### Cancel Execution

**POST** `/api/vwmp/executions/{execution_id}/cancel/`

**Description:** Cancel a running execution.

**Path Parameters:**
- `execution_id` (required): UUID of execution

**Response:** `200 OK`

```json
{
  "execution_id": "execution-uuid",
  "status": "cancelled",
  "cancelled_at": "2025-11-21T16:05:00Z"
}
```

**Error Responses:**
- `404 Not Found`: Execution not found
- `400 Bad Request`: Execution cannot be cancelled (already completed/failed)

---

## Plugin Discovery Endpoints

### List Step Handlers

**GET** `/api/vwmp/step-handlers/`

**Description:** List all available step handlers from registered plugins.

**Query Parameters:**
- `category` (optional): Filter by handler category
- `workflow_type` (optional): Filter by workflow type

**Response:** `200 OK`

```json
{
  "handlers": [
    {
      "path": "release.version_bump",
      "name": "Bump Version",
      "description": "Increment version in version file",
      "category": "version",
      "icon": "🔢",
      "workflow_types": ["release"],
      "config_schema": {
        "type": "object",
        "properties": {
          "version_file": {
            "type": "string",
            "description": "Path to version file"
          }
        }
      }
    }
  ]
}
```

---

### Get Step Handler Schema

**GET** `/api/vwmp/step-handlers/{handler_path}/schema/`

**Description:** Get JSON Schema for step handler configuration.

**Path Parameters:**
- `handler_path` (required): Handler path (e.g., `release.version_bump`)

**Response:** `200 OK`

```json
{
  "handler_path": "release.version_bump",
  "name": "Bump Version",
  "description": "Increment version in version file",
  "config_schema": {
    "type": "object",
    "properties": {
      "version_file": {
        "type": "string",
        "description": "Path to version file relative to workspace root",
        "default": "VERSION"
      },
      "increment_type": {
        "type": "string",
        "enum": ["patch", "minor", "major", "feature", "feature_set"],
        "default": "patch",
        "description": "Version increment type"
      }
    },
    "required": ["version_file"]
  },
  "output_schema": {
    "type": "object",
    "properties": {
      "new_version": {"type": "string"},
      "old_version": {"type": "string"}
    }
  }
}
```

**Error Responses:**
- `404 Not Found`: Handler not found

---

### List Workflow Types

**GET** `/api/vwmp/workflow-types/`

**Description:** List all registered workflow types.

**Response:** `200 OK`

```json
{
  "types": [
    {
      "name": "release",
      "version": "1.0.0",
      "description": "Release workflow for version bump, changelog, commit, tag, push",
      "icon": "🚀",
      "templates": [
        {
          "name": "Standard Release",
          "description": "Full release workflow with all steps"
        }
      ]
    },
    {
      "name": "kanban_review",
      "version": "1.0.0",
      "description": "Kanban documentation review workflow",
      "icon": "📊",
      "templates": []
    }
  ]
}
```

---

## WebSocket Protocol

### WebSocket Connection

**Endpoint:** `ws://localhost:8000/api/vwmp/executions/{execution_id}/`

**Purpose:** Real-time execution updates, step status changes, log streaming.

**Connection Lifecycle:**

1. **Connect:** Client connects to WebSocket endpoint
2. **Subscribe:** Server sends initial execution state
3. **Updates:** Server streams events as execution progresses
4. **Disconnect:** Client disconnects or execution completes

---

### WebSocket Messages

#### Server → Client Messages

**Execution Started:**

```json
{
  "type": "execution_started",
  "execution_id": "execution-uuid",
  "workflow_id": "53dd7364-c347-40de-9583-11b071dcaf95",
  "timestamp": "2025-11-21T16:00:00Z"
}
```

**Step Started:**

```json
{
  "type": "step_started",
  "execution_id": "execution-uuid",
  "step_id": "step-2",
  "step_name": "Create Detailed Changelog",
  "timestamp": "2025-11-21T16:00:05Z"
}
```

**Step Completed:**

```json
{
  "type": "step_completed",
  "execution_id": "execution-uuid",
  "step_id": "step-2",
  "step_name": "Create Detailed Changelog",
  "status": "success",
  "output": {"changelog_file": "CHANGELOG_v0.21.0.41+12.md"},
  "execution_time": 2.5,
  "timestamp": "2025-11-21T16:00:07.5Z"
}
```

**Step Failed:**

```json
{
  "type": "step_failed",
  "execution_id": "execution-uuid",
  "step_id": "step-3",
  "step_name": "Update Main Changelog",
  "status": "failed",
  "error": "File not found: CHANGELOG.md",
  "execution_time": 0.1,
  "timestamp": "2025-11-21T16:00:10Z"
}
```

**Log Message:**

```json
{
  "type": "log",
  "execution_id": "execution-uuid",
  "step_id": "step-2",
  "level": "info",
  "message": "Creating changelog file...",
  "timestamp": "2025-11-21T16:00:05.5Z"
}
```

**Execution Progress:**

```json
{
  "type": "progress",
  "execution_id": "execution-uuid",
  "progress": {
    "total_steps": 10,
    "completed_steps": 3,
    "failed_steps": 0,
    "percentage": 30
  },
  "timestamp": "2025-11-21T16:00:10Z"
}
```

**Execution Completed:**

```json
{
  "type": "execution_completed",
  "execution_id": "execution-uuid",
  "status": "success",
  "total_time": 30.5,
  "timestamp": "2025-11-21T16:00:30.5Z"
}
```

---

#### Client → Server Messages

**Ping (Keep-Alive):**

```json
{
  "type": "ping",
  "timestamp": "2025-11-21T16:00:00Z"
}
```

**Pong (Response to Ping):**

```json
{
  "type": "pong",
  "timestamp": "2025-11-21T16:00:00.1Z"
}
```

---

## Error Responses

### Standard Error Format

```json
{
  "error": "Error message",
  "error_code": "ERROR_CODE",
  "details": {
    "field": "Additional error details"
  },
  "timestamp": "2025-11-21T16:00:00Z"
}
```

### Error Codes

| Code | HTTP Status | Description |
|------|-------------|-------------|
| `WORKFLOW_NOT_FOUND` | 404 | Workflow not found |
| `EXECUTION_NOT_FOUND` | 404 | Execution not found |
| `HANDLER_NOT_FOUND` | 404 | Step handler not found |
| `VALIDATION_ERROR` | 400 | Request validation failed |
| `EXECUTION_ERROR` | 500 | Workflow execution error |
| `AUTHENTICATION_ERROR` | 401 | Authentication required |
| `AUTHORIZATION_ERROR` | 403 | Insufficient permissions |
| `RATE_LIMIT_ERROR` | 429 | Rate limit exceeded |

---

## Authentication & Authorization

### Authentication Methods

**Option 1: JWT (Recommended for FastAPI)**

```http
Authorization: Bearer <jwt_token>
```

**Option 2: Session-Based**

```http
Cookie: sessionid=<session_id>
```

### Authorization

**Roles:**
- `viewer`: Read-only access (list, get workflows)
- `editor`: Create and update workflows
- `executor`: Execute workflows
- `admin`: Full access including deletion

**Permission Model:**
- Workflow-level permissions (future)
- Global permissions (initial)

---

## Rate Limiting

**Limits:**
- Execution requests: 10 per minute per user
- API requests: 100 per minute per user
- WebSocket connections: 5 per user

**Response Headers:**

```http
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 95
X-RateLimit-Reset: 1637510400
```

**Rate Limit Exceeded Response:**

```json
{
  "error": "Rate limit exceeded",
  "error_code": "RATE_LIMIT_ERROR",
  "retry_after": 60,
  "timestamp": "2025-11-21T16:00:00Z"
}
```

---

## Pagination

**Query Parameters:**
- `limit`: Number of results per page (default: 50, max: 100)
- `offset`: Number of results to skip (default: 0)

**Response Format:**

```json
{
  "count": 150,
  "next": "/api/vwmp/workflows/?limit=50&offset=50",
  "previous": null,
  "results": [...]
}
```

---

## Related Documents

- **[FastAPI Migration Plan](E01-vwmp-fastapi-migration-plan.md)** - FastAPI implementation details
- **[Platform Architecture](E01-vwmp-platform-architecture.md)** - Overall architecture
- **[Container View](../../container/overview/epic01-vwmp-container.md)** - Container architecture

---

**Last Updated:** 2025-11-21
**Document Version:** 1.0.0
