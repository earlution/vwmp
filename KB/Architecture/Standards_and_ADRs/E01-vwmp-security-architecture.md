# VWMP Security Architecture

**Status:** Draft - Design
**Epic:** Epic 01 - Visual Workflow Management Platform (VWMP)
**Created:** 2025-11-21
**Version:** v0.21.0.41+12

---

## Executive Summary

This document defines the security architecture for VWMP, covering authentication, authorization, input validation, plugin security, and secure execution of workflows. The security model is designed to protect against common vulnerabilities while maintaining flexibility for plugin extensibility.

**Security Principles:**
- **Defense in Depth:** Multiple layers of security controls
- **Least Privilege:** Minimal permissions required for operations
- **Input Validation:** All inputs validated and sanitized
- **Audit Logging:** All security-relevant events logged
- **Plugin Isolation:** Plugins isolated from core system

---

## Security Model Overview

### Security Layers

1. **Authentication Layer:** Verify user identity
2. **Authorization Layer:** Control access to resources
3. **Input Validation Layer:** Validate and sanitize all inputs
4. **Execution Isolation Layer:** Isolate plugin execution
5. **Audit Layer:** Log all security-relevant events

---

## Authentication

### Authentication Methods

#### Option 1: JWT (Recommended for FastAPI)

**Token Format:**
```json
{
  "sub": "user_id",
  "exp": 1637510400,
  "iat": 1637506800,
  "roles": ["editor", "executor"]
}
```

**Token Storage:**
- **Access Token:** In-memory (client-side)
- **Refresh Token:** HttpOnly cookie (server-side)

**Token Expiry:**
- **Access Token:** 15 minutes
- **Refresh Token:** 7 days

#### Option 2: Session-Based

**Session Storage:**
- **Development:** Database-backed sessions
- **Production:** Redis-backed sessions (recommended)

**Session Configuration:**
- **Session Cookie:** HttpOnly, Secure, SameSite=Lax
- **Session Timeout:** 2 hours inactivity
- **CSRF Protection:** Enabled (CSRF token validation)

---

## Authorization

### Role-Based Access Control (RBAC)

**Roles:**

| Role | Permissions |
|------|-------------|
| **viewer** | List workflows, get workflow, view executions |
| **editor** | Create workflows, update workflows, delete workflows (own) |
| **executor** | Execute workflows, view execution logs |
| **admin** | Full access including system configuration |

### Permission Model

**Resource Permissions:**

1. **Workflow Permissions (Future):**
   - Owner: Full access
   - Collaborator: Execute and view
   - Public: View only

2. **Execution Permissions:**
   - Execute: Can trigger workflow execution
   - View: Can view execution status and logs
   - Cancel: Can cancel running executions (own)

3. **Plugin Permissions:**
   - Load: Can load and use plugins
   - Register: Can register custom plugins (admin only)

---

## Input Validation

### Validation Layers

#### 1. API Request Validation

**Pydantic Models (FastAPI):**

```python
from pydantic import BaseModel, Field, validator

class WorkflowCreateRequest(BaseModel):
    name: str = Field(..., min_length=1, max_length=200)
    version: str = Field(..., regex=r'^\d+\.\d+\.\d+')
    type: str = Field(..., regex=r'^[a-z_]+$')
    description: Optional[str] = Field(None, max_length=1000)
    config: Dict[str, Any] = Field(default_factory=dict)
    steps: List[StepDefinition] = Field(..., min_items=1)
    parameters: List[ParameterDefinition] = Field(default_factory=list)

    @validator('steps')
    def validate_steps(cls, v):
        if not v:
            raise ValueError('Workflow must have at least one step')
        return v
```

#### 2. Workflow Definition Validation

**JSON Schema Validation:**

- Workflow definition validated against JSON Schema
- Step dependencies validated (no circular dependencies)
- Handler references validated (handler must exist)
- Parameter schemas validated

#### 3. Parameter Validation

**Parameter Value Validation:**

- Parameters validated against parameter schema
- Type validation (string, number, boolean, enum)
- Range validation (min/max for numbers, length for strings)
- Format validation (regex patterns)

#### 4. File Path Validation

**Path Traversal Prevention:**

```python
def validate_path(path: str, workspace_root: Path) -> Path:
    """Validate file path to prevent directory traversal."""
    full_path = (workspace_root / path).resolve()
    if not str(full_path).startswith(str(workspace_root.resolve())):
        raise ValueError(f"Path outside workspace: {path}")
    return full_path
```

---

## Plugin Security

### Plugin Isolation

#### Thread Isolation (Initial)

**Current Approach:**
- Plugins execute in separate threads
- Shared memory space
- Trusted plugins only

**Isolation Level:**
- **Sandbox:** None (trusted plugins)
- **Resource Limits:** Configurable timeouts
- **File Access:** Restricted to workspace directory

#### Process Isolation (Future)

**Recommended for Untrusted Plugins:**
- Plugins execute in separate processes
- Isolated memory space
- Restricted file system access
- Restricted network access

**Implementation:**
```python
import subprocess
import multiprocessing

class ProcessIsolatedPluginExecutor:
    def execute(self, handler: StepHandler, context: StepContext) -> StepResult:
        # Execute plugin in separate process
        process = multiprocessing.Process(
            target=self._execute_plugin,
            args=(handler, context)
        )
        process.start()
        process.join(timeout=context.timeout)

        if process.is_alive():
            process.terminate()
            return StepResult(status="failed", error="Timeout")
```

### Plugin Validation

**Plugin Signature Validation (Future):**

- Plugins signed with cryptographic signature
- Signature verified before loading
- Unsigned plugins require admin approval

**Plugin Metadata Validation:**

- Plugin metadata validated against schema
- Plugin version checked for compatibility
- Plugin dependencies verified

---

## Secure Execution

### Workspace Isolation

**Workspace Root Restriction:**

- All file operations restricted to workspace root
- Path validation prevents directory traversal
- Workspace root configurable per user/tenant

### Command Execution Security

**Subprocess Execution:**

```python
import subprocess
from pathlib import Path

def execute_command_safely(
    command: List[str],
    workspace_root: Path,
    timeout: int = 300
) -> subprocess.CompletedProcess:
    """Execute command with security restrictions."""
    # Validate command
    if not command or not command[0]:
        raise ValueError("Invalid command")

    # Restrict to workspace
    env = os.environ.copy()
    env["PWD"] = str(workspace_root)

    # Execute with timeout
    result = subprocess.run(
        command,
        cwd=str(workspace_root),
        env=env,
        timeout=timeout,
        capture_output=True,
        text=True
    )
    return result
```

**Git Command Validation:**

- Only whitelisted git commands allowed
- Git commands validated before execution
- Git operations restricted to workspace repository

### Resource Limits

**Execution Limits:**

- **Timeout:** Configurable per step (default: 300 seconds)
- **Memory:** Configurable per workflow (default: 512MB)
- **CPU:** Configurable per workflow (default: 1 core)
- **File Size:** Maximum file size limits (default: 100MB)

---

## Secure Storage

### Workflow Definition Storage

**File Permissions:**

- Workflow files: `0600` (owner read/write only)
- Directory permissions: `0700` (owner access only)

**Encryption at Rest (Future):**

- Sensitive workflow configurations encrypted
- Encryption key managed separately
- Key rotation support

### Execution History Storage

**Log Sanitization:**

- Sensitive data (passwords, tokens) redacted from logs
- PII data anonymized in logs
- Execution history retention policy (30 days default)

---

## Network Security

### API Security

**HTTPS Required:**

- All API requests over HTTPS in production
- TLS 1.2+ required
- Certificate validation enforced

**CORS Configuration:**

- CORS restricted to allowed origins
- Credentials: include (for cookies)
- Preflight requests validated

### WebSocket Security

**WebSocket Authentication:**

- Authentication token validated on connection
- Connection rejected if token invalid
- Connection timeout: 5 minutes inactivity

**WebSocket Message Validation:**

- All messages validated against schema
- Message size limits (1MB default)
- Rate limiting per connection (100 messages/minute)

---

## Audit Logging

### Security Events Logged

1. **Authentication Events:**
   - Login success/failure
   - Token refresh
   - Session creation/destruction

2. **Authorization Events:**
   - Permission denied
   - Role changes
   - Access granted/revoked

3. **Workflow Events:**
   - Workflow creation/update/deletion
   - Workflow execution start/completion
   - Workflow execution failures

4. **Plugin Events:**
   - Plugin loading
   - Plugin execution
   - Plugin errors

5. **Administrative Events:**
   - Configuration changes
   - User management
   - System updates

### Log Format

**Structured Logging:**

```json
{
  "timestamp": "2025-11-21T16:00:00Z",
  "level": "INFO",
  "event_type": "workflow_execution_started",
  "user_id": "user-123",
  "workflow_id": "53dd7364-c347-40de-9583-11b071dcaf95",
  "execution_id": "execution-uuid",
  "ip_address": "192.168.1.100",
  "user_agent": "Mozilla/5.0..."
}
```

**Log Storage:**

- **Development:** File-based logging
- **Production:** Centralized logging (ELK, Splunk, etc.)
- **Retention:** 90 days default

---

## Vulnerability Management

### Known Vulnerabilities

**Dependency Scanning:**

- Regular dependency scans (weekly)
- Automated security updates
- CVSS score tracking

**Penetration Testing:**

- Quarterly penetration tests
- Bug bounty program (future)
- Security audit reports

### Incident Response

**Security Incident Response:**

1. **Detection:** Automated detection via monitoring
2. **Containment:** Isolate affected systems
3. **Investigation:** Analyze incident details
4. **Remediation:** Fix vulnerabilities
5. **Post-Incident:** Review and improve

---

## Security Best Practices

### Development

1. **Input Validation:** Validate all inputs at API boundary
2. **Output Encoding:** Encode outputs to prevent XSS
3. **SQL Injection:** Use parameterized queries (if database)
4. **CSRF Protection:** CSRF tokens for state-changing operations
5. **Secret Management:** Never commit secrets, use environment variables

### Deployment

1. **Secrets Management:** Use secret management service (AWS Secrets Manager, HashiCorp Vault)
2. **Network Security:** Firewall rules, VPN for admin access
3. **Monitoring:** Security event monitoring and alerting
4. **Backups:** Encrypted backups with tested restore procedures
5. **Updates:** Regular security updates and patches

---

## Compliance

### Security Standards

**Future Compliance Targets:**

- **SOC 2:** Security controls and auditing
- **GDPR:** Data protection and privacy
- **OWASP Top 10:** Web application security
- **CWE Top 25:** Common weakness enumeration

---

## Related Documents

- **[API Design](E01-vwmp-api-design.md)** - API security details
- **[Platform Architecture](E01-vwmp-platform-architecture.md)** - Overall architecture
- **[Plugin Architecture](E01-vwmp-plugin-architecture.md)** - Plugin security model

---

**Last Updated:** 2025-11-21
**Document Version:** 1.0.0
