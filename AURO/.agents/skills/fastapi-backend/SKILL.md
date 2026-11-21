---
name: fastapi-backend
description: Senior architectural guidelines for AURO FastAPI backend, Pydantic v2 validation, async database sessions, and modular routes.
---

# FastAPI Backend Skill for AURO

## Guidelines
1. **Async & Type Annotations**: Use strict typing across all route handlers and services.
2. **Pydantic v2**: Always use explicit request/response schemas. Never expose raw database entities directly to API callers.
3. **Role-Based Access Control**: Enforce permissions on the backend using `require_role([UserRole.ADMIN])` dependencies, never relying on UI hiding alone.
4. **Error Handling**: Use structured `HTTPException` with meaningful error detail and status codes (401, 403, 404, 409, 422).
5. **Database Sessions**: Inject database sessions via `Depends(get_db)` to ensure cleanup and rollback on failure.

