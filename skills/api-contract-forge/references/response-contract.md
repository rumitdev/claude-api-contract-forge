# Response Contract — Single Source of Truth

Every framework template MUST produce responses matching these exact shapes. No exceptions. This ensures any API consumer (frontend, mobile, other microservices) can use one response handler regardless of which backend framework generated the API.

Read this file before generating any controller/handler/router code.

---

## 1. Success — Single Resource

Used by: GET /:id, POST, PUT/PATCH

```json
{
  "statusCode": 200,
  "message": "Resource retrieved",
  "data": {
    "id": "uuid-here",
    "title": "Example",
    "amount": 150.00,
    "status": "draft",
    "createdAt": "2025-01-15T10:30:00Z",
    "updatedAt": "2025-01-15T10:30:00Z"
  }
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `statusCode` | `number` | Yes | HTTP status code (200, 201) |
| `message` | `string` | Yes | Human-readable message (use i18n key if detected) |
| `data` | `object` | Yes | The resource object |

**Status codes:**
- `200` — GET /:id, PUT/PATCH (existing resource returned)
- `201` — POST (new resource created)

---

## 2. Success — Paginated List

Used by: GET / (list endpoints)

```json
{
  "statusCode": 200,
  "message": "Data retrieved",
  "data": {
    "items": [
      { "id": "uuid-1", "title": "Example 1" },
      { "id": "uuid-2", "title": "Example 2" }
    ],
    "total": 50,
    "page": 1,
    "limit": 10,
    "totalPages": 5
  }
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `statusCode` | `number` | Yes | Always `200` |
| `message` | `string` | Yes | Human-readable message |
| `data.items` | `array` | Yes | Array of resources. Empty array `[]` if no results — never `null` |
| `data.total` | `number` | Yes | Total count of all matching records (not just this page) |
| `data.page` | `number` | Yes | Current page number (1-based) |
| `data.limit` | `number` | Yes | Items per page |
| `data.totalPages` | `number` | Yes | Calculated: `ceil(total / limit)` |

**Critical rules:**
- `items` is ALWAYS an array — empty `[]`, never `null`, never omitted
- `total` is the count across ALL pages, not just current page
- `page` is 1-based (first page = 1, not 0)
- `totalPages` is calculated by the backend, not the consumer

---

## 3. Success — No Content

Used by: DELETE

```
HTTP 204 No Content
(empty body)
```

No JSON body. Just the status code. Every framework supports this natively.

---

## 4. Error — Validation Error

Used by: POST, PUT/PATCH when input fails validation

```json
{
  "type": "validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "One or more fields failed validation",
  "traceId": "req-abc-123",
  "errors": [
    { "field": "email", "message": "Must be a valid email address", "code": "INVALID_FORMAT" },
    { "field": "amount", "message": "Must be greater than 0", "code": "MIN_VALUE" }
  ]
}
```

| Field | Type | Required | Notes |
|-------|------|----------|-------|
| `type` | `string` | Yes | Error category identifier |
| `title` | `string` | Yes | Human-readable error title |
| `status` | `number` | Yes | HTTP status code |
| `detail` | `string` | Recommended | Specific error description |
| `traceId` | `string` | Recommended | Request correlation ID for debugging |
| `errors` | `array` | For validation | Field-level error details |
| `errors[].field` | `string` | Yes | Field name that failed |
| `errors[].message` | `string` | Yes | Human-readable error message |
| `errors[].code` | `string` | Yes | Machine-readable error code |

---

## 5. Error — Not Found

```json
{
  "type": "not-found",
  "title": "Resource Not Found",
  "status": 404,
  "detail": "Invoice with id 'abc-123' was not found",
  "traceId": "req-abc-123"
}
```

---

## 6. Error — Unauthorized / Forbidden

```json
{
  "type": "unauthorized",
  "title": "Unauthorized",
  "status": 401,
  "detail": "Invalid or expired authentication token",
  "traceId": "req-abc-123"
}
```

```json
{
  "type": "forbidden",
  "title": "Forbidden",
  "status": 403,
  "detail": "You do not have permission to perform this action",
  "traceId": "req-abc-123"
}
```

---

## 7. Error — Server Error

```json
{
  "type": "internal-error",
  "title": "Internal Server Error",
  "status": 500,
  "detail": "An unexpected error occurred",
  "traceId": "req-abc-123"
}
```

**Never expose stack traces or internal error messages in production.** The `detail` field should be generic. Use `traceId` to look up the full error in server logs.

---

## Nested Resources

### Level 1 — Simple Include

```
GET /api/v1/users/user-001?include=profile
```

```json
{
  "statusCode": 200,
  "message": "User retrieved",
  "data": {
    "id": "user-001",
    "name": "Rumit",
    "email": "rumit@example.com",
    "profileId": "prof-001",
    "profile": {
      "id": "prof-001",
      "bio": "Full stack developer",
      "experience": 5
    },
    "createdAt": "2025-01-15T10:30:00Z"
  }
}
```

Without `?include`:
```json
{
  "statusCode": 200,
  "message": "User retrieved",
  "data": {
    "id": "user-001",
    "name": "Rumit",
    "email": "rumit@example.com",
    "profileId": "prof-001",
    "createdAt": "2025-01-15T10:30:00Z"
  }
}
```

### Level 2 — Nested Include (Dot Notation)

Use dot notation to include nested relations: `?include=profile.skills`

```
GET /api/v1/users/user-001?include=profile.skills
```

```json
{
  "statusCode": 200,
  "message": "User retrieved",
  "data": {
    "id": "user-001",
    "name": "Rumit",
    "email": "rumit@example.com",
    "profileId": "prof-001",
    "profile": {
      "id": "prof-001",
      "bio": "Full stack developer",
      "experience": 5,
      "skills": [
        { "id": "skill-001", "name": "Node.js", "level": "expert" },
        { "id": "skill-002", "name": "Python", "level": "intermediate" }
      ]
    },
    "createdAt": "2025-01-15T10:30:00Z"
  }
}
```

**Important:** `?include=profile.skills` automatically includes `profile` too — you don't need to say `?include=profile,profile.skills`.

### Multiple Includes

Combine with commas: `?include=profile.skills,company`

```
GET /api/v1/users/user-001?include=profile.skills,company
```

```json
{
  "statusCode": 200,
  "message": "User retrieved",
  "data": {
    "id": "user-001",
    "name": "Rumit",
    "profileId": "prof-001",
    "companyId": "comp-001",
    "profile": {
      "id": "prof-001",
      "bio": "Full stack developer",
      "skills": [
        { "id": "skill-001", "name": "Node.js", "level": "expert" }
      ]
    },
    "company": {
      "id": "comp-001",
      "name": "Acme Corp"
    },
    "createdAt": "2025-01-15T10:30:00Z"
  }
}
```

### Null and Empty Rules for Nested Data

Every nesting level follows the same rules:

| Scenario | Response | Never Do |
|----------|----------|----------|
| User has no profile (has-one, unassigned) | `"profile": null` | Don't omit the field |
| Profile has no skills (has-many, empty) | `"skills": []` | Don't use `"skills": null` |
| User has no company (belongs-to, unassigned) | `"company": null, "companyId": null` | Don't omit either field |
| Include requested but parent is null | `"profile": null` (skip nested loading) | Don't error or crash |

**Example — user with no profile, skills requested:**
```
GET /api/v1/users/user-002?include=profile.skills
```
```json
{
  "statusCode": 200,
  "message": "User retrieved",
  "data": {
    "id": "user-002",
    "name": "New User",
    "profileId": null,
    "profile": null,
    "createdAt": "2025-03-01T08:00:00Z"
  }
}
```
Profile is `null`, so skills are not loaded — the response stops at the null parent. No error.

### Nested Include Rules

1. **Without `?include`** — return foreign key IDs only: `"profileId": "prof-001"`
2. **With `?include=profile`** — return the full nested object at level 1. Nested relations inside profile (like skills) are NOT included unless explicitly requested
3. **With `?include=profile.skills`** — load profile AND skills inside it. The parent is always included when a child is requested
4. **Relations are nullable, not optional** — `"profile": null` when unassigned, never omit the field
5. **Arrays are empty, never null** — `"skills": []` when empty, never `"skills": null`
6. **Max 3 levels of nesting** — `user.profile.skills` is fine (3 levels). `user.profile.skills.endorsements` should be a separate endpoint
7. **Only whitelisted includes allowed** — reject unknown include values with a 400 error
8. **Dot notation for nested** — `profile.skills` means "include profile and inside it include skills"
9. **Include on lists** — same rules apply for list endpoints: `GET /users?include=profile.skills&page=1&limit=10`

---

## Field Naming Rules

| Rule | Correct | Wrong |
|------|---------|-------|
| Use camelCase for JSON fields | `createdAt` | `created_at` |
| Use consistent ID field name | `id` | `_id`, `uuid`, `ID` |
| Use ISO 8601 for dates | `2025-01-15T10:30:00Z` | `2025-01-15`, `1705312200` |
| Booleans are booleans | `true` | `"true"`, `1` |
| Numbers are numbers | `150.00` | `"150.00"` |
| Null means null | `null` | `""`, `{}`, `"null"` |

**Exception:** Backend languages use their own conventions internally (snake_case in Python/Ruby, camelCase in Java/TypeScript). The JSON output MUST be camelCase regardless of the backend language. Use serializer aliases, JSON tags, or response transformers to achieve this.

---

## Framework Implementation Guide

Each framework needs response helpers that produce the exact shapes above. Here's how:

### Frameworks with natural JSON control (Express, FastAPI, Go, Spring Boot)
Create response helper functions/classes that wrap data in the standard envelope.

### Frameworks with opinionated response formats (Django REST, Laravel, Rails)
Override the default serialization to match the standard:
- **Django**: Custom pagination class + custom renderer that wraps responses in the envelope
- **Laravel**: Override `JsonResource` wrapper + custom pagination in the collection
- **Rails**: Custom render helper that wraps serialized output

The key principle: the framework's internal conventions don't matter — the JSON output to the client MUST match this contract. Use the framework's extension points (custom renderers, middleware, response transformers) to normalize the output.

---

## Quick Validation Checklist

Before marking any generated endpoint as complete, verify:

- [ ] Single item response has `statusCode`, `message`, `data` wrapper
- [ ] List response has `statusCode`, `message`, `data.items`, `data.total`, `data.page`, `data.limit`, `data.totalPages`
- [ ] Delete returns `204 No Content` with empty body
- [ ] Validation errors return `type`, `title`, `status`, `errors[]` with `field`, `message`, `code`
- [ ] Not Found errors return `type`, `title`, `status`, `detail`
- [ ] All JSON fields use camelCase
- [ ] Dates are ISO 8601
- [ ] Null relations are `null`, not missing
- [ ] Empty arrays are `[]`, not `null`
- [ ] No raw framework-default responses leak through
