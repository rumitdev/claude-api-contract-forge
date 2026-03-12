# API Contracting Checklist

This checklist covers everything the API Contract Forge skill enforces when building APIs. Share this with your backend team so everyone follows the same standards.

---

## 1. Setup & Detection

- [ ] Identify framework, language, ORM, and validation library
- [ ] Follow existing project patterns (folder structure, code style, imports)
- [ ] Identify auth strategy (JWT, session, API key)
- [ ] Identify RBAC/permission patterns (if any)
- [ ] Identify i18n setup (if any)

## 2. Resource Design

- [ ] Resource naming follows singular/plural conventions
- [ ] All fields defined with correct types and validation rules
- [ ] Required vs optional fields clearly decided
- [ ] Relations defined (belongs to, has many)
- [ ] Only the needed CRUD operations are generated (not always all 5)

## 3. Validation Layer

- [ ] Input validation schema exists for every POST/PUT endpoint
- [ ] Validation middleware applied on every route
- [ ] Types derived from validation schemas (single source of truth, no duplication)
- [ ] Format validators applied (email, UUID, URL, datetime)
- [ ] Enum validation for status/category fields
- [ ] List endpoint `limit` param bounded (max 100) to prevent unbounded queries

## 4. Response Contract

- [ ] Standard response envelope used (`{ statusCode, message, data }`)
- [ ] Paginated response uses consistent shape (offset: page/limit/total/totalPages)
- [ ] All responses go through response helpers (no raw `res.json()`)
- [ ] Timestamps included in every resource (createdAt, updatedAt)
- [ ] Sensitive fields excluded from response (password, tokens, secrets)

## 5. Error Handling

- [ ] RFC 9457 error format used (type, title, status, detail, traceId)
- [ ] Field-level validation errors returned in `errors[]` array
- [ ] Consistent error shape across all endpoints
- [ ] Error traceability — request/trace ID included in error responses

## 6. API Documentation (Swagger/OpenAPI)

- [ ] Swagger/OpenAPI annotation on every route
- [ ] All response codes documented (200, 201, 400, 401, 403, 404, 500)
- [ ] API version included in route prefix and Swagger info
- [ ] 429 Too Many Requests response documented
- [ ] Request body and response body schemas fully documented
- [ ] Every endpoint has tags, summary, and security annotations

## 7. Security (OWASP API Top 10)

| OWASP Risk | Checklist Item |
|------------|---------------|
| API1: Broken Object Level Auth | [ ] GET /:id checks resource ownership, not just authentication |
| API2: Broken Authentication | [ ] Auth middleware on every protected route |
| API3: Broken Object Property Auth | [ ] Sensitive fields (password, token) excluded from responses |
| API4: Unrestricted Resource Consumption | [ ] Rate limiting in place, `limit` param bounded (max 100) |
| API5: Broken Function Level Auth | [ ] Admin-only endpoints protected with permission checks |
| API6: Sensitive Business Flows | [ ] Critical operations (delete, status change) require proper authorization |
| API7: SSRF | [ ] URL inputs validated and sanitized |
| API8: Security Misconfiguration | [ ] CORS configured, error details hidden in production |
| API9: Improper Inventory | [ ] Every endpoint documented in Swagger |
| API10: Unsafe Consumption | [ ] Third-party API responses validated before use |

## 8. Production Standards

- [ ] **API Versioning** — routes use `/api/v1/` prefix
- [ ] **Request Correlation** — X-Request-Id and X-Trace-Id headers in responses
- [ ] **Idempotency** — Idempotency-Key header supported on POST/PUT endpoints
- [ ] **Rate Limiting Headers** — X-RateLimit-Limit, X-RateLimit-Remaining, X-RateLimit-Reset
- [ ] **ETags** — Conditional requests (If-None-Match / 304) for GET /:id endpoints
- [ ] **Deprecation Strategy** — old endpoints marked `deprecated: true` with removal date
- [ ] **i18n** — all user-facing message strings use i18n keys (if i18n is in the project)
- [ ] **API Linting** — Spectral or equivalent in CI/CD pipeline

## 9. Special Patterns (When Needed)

### File Upload
- [ ] Multipart/form-data validation (file type, size limits)
- [ ] Framework-specific file handler (multer, UploadFile, etc.)
- [ ] File storage service layer
- [ ] Swagger docs for file upload endpoints

### Soft Delete
- [ ] `deletedAt` timestamp field on the model
- [ ] Auto-filtering — queries exclude soft-deleted records by default
- [ ] Restore endpoint (POST /:id/restore)
- [ ] Trash listing endpoint (GET /trash)

### Relationship Loading
- [ ] `?include=company,items` query parameter support
- [ ] Whitelist of allowed relations (no arbitrary includes)
- [ ] ORM-specific eager loading (Prisma include, SQLAlchemy joinedload, etc.)
- [ ] Response shape documented with included relations

### Advanced Filtering
- [ ] Date range filters (`createdAfter`, `createdBefore`)
- [ ] Numeric range filters (`amountMin`, `amountMax`)
- [ ] Multi-value enum filters (comma-separated: `status=draft,sent`)
- [ ] Null check filters (`assigneeIsNull=true`)
- [ ] Text search with configurable fields
- [ ] Database indexes on commonly filtered columns

## 10. Anti-Patterns to Avoid

- [ ] No mixed naming conventions (pick camelCase OR snake_case, not both)
- [ ] No inconsistent ID fields (use `id` everywhere, not `_id` in some and `uuid` in others)
- [ ] No boolean/number as strings (`true` not `"true"`, `1` not `"1"`)
- [ ] No inconsistent null representation (use `null`, not `""` or `{}` or missing field)
- [ ] No over-nested responses (data should not be buried 5+ levels deep)
- [ ] No inconsistent pagination shapes across list endpoints
- [ ] No inconsistent response envelopes across endpoints
- [ ] No exposed sensitive data (JWT tokens, passwords in responses)
- [ ] No missing timestamps on records
- [ ] No error responses without trace IDs
- [ ] No unbounded list queries (always enforce max limit)
- [ ] No breaking changes without deprecation notice

## 11. Contract Analysis (For Existing APIs)

When analyzing existing API responses:

- [ ] Deep recursive type inference from JSON
- [ ] Nullable vs optional vs nullable+optional correctly identified
- [ ] Formats detected (UUID, email, datetime, URL, JWT, ObjectId)
- [ ] Nested types extracted as named types (never inline 3+ levels)
- [ ] Pagination and envelope patterns auto-detected
- [ ] Breaking changes identified when comparing response versions
- [ ] Existing code validated against inferred contract

---

## How to Use This

1. **Before building a new API** — Walk through sections 1-8 as your planning checklist
2. **During development** — Use sections 3-6 to verify each endpoint as you build it
3. **Before code review** — Run through sections 7-10 as a final quality gate
4. **For existing APIs** — Use section 11 to audit and improve what's already deployed

The API Contract Forge skill automates all of this. Just say `build api for [resource]` and it handles everything above.
