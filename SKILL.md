---
name: api-contract-forge
description: "Universal API contract skill. Build new APIs with best contracting standards for any tech stack, infer types from JSON responses, detect inconsistencies, and generate contract artifacts (TypeScript, Zod, Pydantic, Dart, Kotlin, Swift, OpenAPI, JSON Schema). Triggers on: build api, new endpoint, api contract, generate types, infer schema, normalize API, scaffold api."
---

# API Contract Forge

A universal skill for API contracting. It does two things:

1. **BUILD** — Scaffold new API endpoints with proper contracting (validation, typed responses, documentation) for any tech stack
2. **ANALYZE** — Take raw JSON API responses, infer types, detect inconsistencies, and generate contract artifacts for any language

## When to Use

Activate this skill when the user:

- Wants to **build a new API endpoint** with proper contracting standards
- Says "build api", "new endpoint", "scaffold api", "create api for [resource]"
- Wants to generate a **full CRUD** for a resource (routes, controller, service, validation, docs)
- Pastes a raw JSON API response and wants types/schemas generated
- Wants to compare two or more API responses for the same endpoint to detect shape drift
- Asks to "create a contract" or "generate types" from an API response
- Wants to normalize inconsistent API response formats
- Needs to detect breaking changes between two versions of a response
- Wants contract artifacts in any language (TypeScript, Python, Dart, Kotlin, Swift, etc.)
- Says "api contract", "infer schema", "generate types from JSON", "normalize API"

## Core Workflow

The skill has two modes. Choose based on what the user needs:

- **BUILD mode** (Phase 0) — User wants to create a NEW API endpoint
- **ANALYZE mode** (Phases 1-6) — User has an EXISTING API response to analyze

---

## BUILD MODE — Create New API Endpoints with Contracting

### Phase 0: BUILD — Scaffold New API with Full Contracting

This phase generates production-ready API endpoints with validation, typed responses, and documentation baked in. It works with ANY framework by first detecting the project's tech stack and conventions.

#### Step 0.1: DETECT — Auto-Detect Project Tech Stack

Scan the project to determine the framework, language, and existing patterns. Check these files:

| File | Detects |
|------|---------|
| `package.json` | Node.js framework (Express, Fastify, NestJS, Hono, Koa) + validation lib (Zod, Joi, class-validator) + ORM (Prisma, TypeORM, Mongoose, Sequelize, Drizzle) |
| `requirements.txt` / `pyproject.toml` / `Pipfile` | Python framework (FastAPI, Django REST, Flask) + ORM (SQLAlchemy, Django ORM, Tortoise) |
| `build.gradle` / `pom.xml` | Java/Kotlin framework (Spring Boot, Ktor) |
| `go.mod` | Go framework (Gin, Echo, Fiber, Chi) |
| `Gemfile` | Ruby framework (Rails, Sinatra) |
| `pubspec.yaml` | Dart/Flutter (shelf, dart_frog) |
| `Cargo.toml` | Rust framework (Actix, Axum, Rocket) |
| `composer.json` | PHP framework (Laravel, Symfony) |

Also scan for:
- **Existing route files** — to detect routing patterns (file-based, decorator-based, explicit router)
- **Existing controllers/handlers** — to detect response patterns (helper functions, raw res.json, etc.)
- **Existing validation schemas** — to detect validation approach
- **Existing type/model files** — to detect type organization
- **Swagger/OpenAPI config** — to detect documentation approach
- **Database config** — to detect ORM and DB type

**Output a detection summary:**
```
Project Detection
=================
Language:    TypeScript
Framework:   Express 4.x
Validation:  Zod 3.x
ORM:         Prisma 5.x (PostgreSQL)
Docs:        Swagger via swagger-jsdoc
Response:    Custom helpers (sendSuccess, sendError, sendPaginatedResponse)
Structure:   routes/ → controllers/ → services/
Auth:        JWT middleware (authenticate)
Permissions: requirePermission("resource.action.scope")
i18n:        i18next (req.t)
```

If detection fails or is ambiguous, ASK the user to confirm.

#### Step 0.2: GATHER — Collect Resource Requirements

Ask the user for:

1. **Resource name** (e.g., "invoice", "meal-plan", "appointment")
2. **Fields** — name, type, required/optional, validation rules. Accept any of:
   - Natural language: "title (string, required), amount (number, min 0), status (enum: draft/sent/paid)"
   - JSON sample: `{ "title": "Invoice #1", "amount": 150.00, "status": "draft" }`
   - Existing database model: "use the Invoice model from Prisma schema"
3. **Operations** — which CRUD operations to generate:
   - `C` — Create (POST)
   - `R` — Read single (GET /:id)
   - `L` — List with pagination (GET /)
   - `U` — Update (PUT /:id or PATCH /:id)
   - `D` — Delete (DELETE /:id)
   - Custom operations (e.g., "POST /:id/send", "PATCH /:id/status")
4. **Relations** (optional) — belongs to company, has many items, etc.
5. **Permissions** (optional) — what permission pattern to use
6. **Special requirements** (optional) — file upload, soft delete, search/filter, etc.

**Handling partial generation:**

| User says | What to generate |
|-----------|-----------------|
| "create an invoice API" | Only `C` (POST) — single endpoint |
| "I need GET and POST for invoices" | Only `R`, `L`, `C` — specified endpoints |
| "build invoice CRUD" or just "invoice" | Full `CRLUD` — all 5 operations |
| "invoice with C, R, U" | Only `C`, `R`, `U` — specified subset |
| "POST /invoices/:id/send" | Only the custom operation — single endpoint |

**Rules:**
- If the user specifies exact operations → generate ONLY those operations, nothing more
- If the user says "just create" or "only POST" → generate a single POST endpoint
- If the user provides just a resource name with no operation hint → generate full CRUD (CRLUD) with sensible defaults
- If ambiguous (e.g., "I need an invoice API") → ASK: "Which operations do you need? All CRUD, or specific ones like just Create (POST)?"
- Users can always come back later and say "now add the update endpoint" to incrementally add more operations to the same resource

#### Step 0.3: GENERATE — Create Contract Layers for Requested Operations

Generate files following the DETECTED project conventions. The output must match the existing codebase patterns EXACTLY — same file naming, same folder structure, same code style, same import patterns.

**IMPORTANT — Partial generation rules:**
- Generate ONLY the operations the user requested in Step 0.2
- If user asked for only `C` (Create) → generate only the create schema, create route, create controller method, and create service method. Do NOT generate read/update/delete code.
- If user asked for `C` and `R` → generate only create + read-single schemas, routes, controller, and service methods
- Validation schemas: generate only the schemas needed for requested operations (e.g., skip `update{Resource}Schema` if no Update operation, skip `{resource}QuerySchema` if no List operation)
- Routes file: include only the route definitions for requested operations
- Controller & Service: include only the methods for requested operations
- Swagger docs: document only the generated endpoints
- Frontend types: generate only the types relevant to the generated endpoints
- The templates below show ALL operations for reference — extract only what's needed

Below are generation rules per framework. Use ONLY the detected framework's rules.

---

##### Express + TypeScript

**File 1: Validation Schema** — `src/types/{resource}.types.ts` (or detected pattern)

```typescript
import { z } from "zod"; // or Joi, depending on detection

// CREATE schema
export const create{Resource}Schema = z.object({
  // Fields with validation rules
  title: z.string().min(1).max(200),
  amount: z.number().min(0),
  status: z.enum(["draft", "sent", "paid"]).default("draft"),
  // Relations
  companyId: z.string().uuid(),
});

export type Create{Resource}Input = z.infer<typeof create{Resource}Schema>;

// UPDATE schema (all fields optional)
export const update{Resource}Schema = create{Resource}Schema.partial();
export type Update{Resource}Input = z.infer<typeof update{Resource}Schema>;

// QUERY schema (for list filtering/pagination)
export const {resource}QuerySchema = z.object({
  page: z.coerce.number().min(1).default(1),
  limit: z.coerce.number().min(1).max(100).default(10),
  search: z.string().optional(),
  status: z.enum(["draft", "sent", "paid"]).optional(),
  sortBy: z.string().default("createdAt"),
  sortOrder: z.enum(["asc", "desc"]).default("desc"),
});

export type {Resource}QueryInput = z.infer<typeof {resource}QuerySchema>;
```

Validation rules:
- Use `.min()` / `.max()` for strings and numbers
- Use `.email()`, `.uuid()`, `.url()` for format validation
- Use `.enum()` for fixed value sets
- Use `.default()` for fields with defaults
- Use `.optional()` for non-required fields
- Use `.nullable()` for explicitly nullable fields
- Use `.refine()` for custom validation logic
- Derive ALL TypeScript types from schemas with `z.infer<>`
- NEVER define types separately from schemas

**File 2: Routes** — `src/routes/{resource}.routes.ts`

```typescript
import { Router } from "express";
// Import from detected middleware patterns
import { authenticate } from "../middlewares/auth.middleware";
import { requirePermission } from "../middlewares/permission.middleware";
import { validateBody, validateQuery, validateParams } from "../middlewares/validation.middleware";
import { asyncHandler } from "../middlewares/error.middleware";
import { create{Resource}Schema, update{Resource}Schema, {resource}QuerySchema } from "../types/{resource}.types";
import { {resource}Controller } from "../controllers/{resource}.controller";

const router = Router();

// Apply auth middleware to all routes (if detected in project)
router.use(authenticate);

/**
 * @swagger
 * /api/v1/{resources}:
 *   post:
 *     summary: Create a new {resource}
 *     tags: [{Resources}]
 *     security:
 *       - bearerAuth: []
 *     requestBody:
 *       required: true
 *       content:
 *         application/json:
 *           schema:
 *             $ref: '#/components/schemas/Create{Resource}'
 *     responses:
 *       201:
 *         description: {Resource} created successfully
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/{Resource}Response'
 *       400:
 *         description: Validation error
 *       401:
 *         description: Unauthorized
 */
router.post(
  "/",
  requirePermission("{resources}.create"),
  validateBody(create{Resource}Schema),
  asyncHandler({resource}Controller.create{Resource})
);

/**
 * @swagger
 * /api/v1/{resources}:
 *   get:
 *     summary: List {resources} with pagination
 *     tags: [{Resources}]
 *     security:
 *       - bearerAuth: []
 *     parameters:
 *       - in: query
 *         name: page
 *         schema:
 *           type: integer
 *           default: 1
 *       - in: query
 *         name: limit
 *         schema:
 *           type: integer
 *           default: 10
 *       - in: query
 *         name: search
 *         schema:
 *           type: string
 *       - in: query
 *         name: status
 *         schema:
 *           type: string
 *           enum: [draft, sent, paid]
 *     responses:
 *       200:
 *         description: {Resources} retrieved successfully
 *         content:
 *           application/json:
 *             schema:
 *               $ref: '#/components/schemas/Paginated{Resource}Response'
 */
router.get(
  "/",
  requirePermission("{resources}.view.all"),
  validateQuery({resource}QuerySchema),
  asyncHandler({resource}Controller.get{Resources})
);

// GET /:id, PUT /:id, DELETE /:id — same pattern with swagger JSDoc

export default router;
```

Swagger rules:
- EVERY route MUST have `@swagger` JSDoc
- Include `tags`, `security`, `parameters`, `requestBody`, `responses`
- Reference `$ref` schemas under `components/schemas`
- Document ALL response codes (200, 201, 400, 401, 403, 404, 500)
- Include `description` for every parameter

**File 3: Controller** — `src/controllers/{resource}.controller.ts`

```typescript
import { Request, Response, NextFunction } from "express";
// Use detected response helpers
import { sendSuccess, sendError, sendPaginatedResponse } from "../utils/response";
import { {resource}Service } from "../services/{resource}.service";

export class {Resource}Controller {
  async create{Resource}(req: Request, res: Response): Promise<void> {
    try {
      const result = await {resource}Service.create(req.body);
      sendSuccess(res, result, req.t("{RESOURCE}_CREATED"), 201);
    } catch (error: any) {
      sendError(res, req.t(error.message), error.statusCode || 500);
    }
  }

  async get{Resources}(req: Request, res: Response): Promise<void> {
    try {
      const { items, total } = await {resource}Service.findAll(req.query as any);
      const { page, limit } = req.query as any;
      sendPaginatedResponse(res, items, total, page, limit, req.t("DATA_RETRIEVED"));
    } catch (error: any) {
      sendError(res, req.t("REQUEST_FAILED"), 500, error.message);
    }
  }

  // getById, update, delete — same pattern
}

export const {resource}Controller = new {Resource}Controller();
```

Controller rules:
- Use the project's DETECTED response helpers — NEVER use raw `res.json()`
- Use the project's DETECTED i18n pattern for messages
- Wrap all handlers in try/catch (or use detected asyncHandler)
- Keep controllers THIN — delegate logic to service layer
- Use the project's DETECTED request type (AuthenticatedRequest, etc.)

**File 4: Service** — `src/services/{resource}.service.ts`

```typescript
// Use detected ORM
import { prisma } from "../config/database";
import { Create{Resource}Input, Update{Resource}Input, {Resource}QueryInput } from "../types/{resource}.types";

export class {Resource}Service {
  async create(data: Create{Resource}Input) {
    return prisma.{resource}.create({ data });
  }

  async findAll(query: {Resource}QueryInput) {
    const { page, limit, search, status, sortBy, sortOrder } = query;
    const where: any = {};

    if (search) {
      where.OR = [
        { title: { contains: search, mode: "insensitive" } },
      ];
    }
    if (status) where.status = status;

    const [items, total] = await Promise.all([
      prisma.{resource}.findMany({
        where,
        skip: (page - 1) * limit,
        take: limit,
        orderBy: { [sortBy]: sortOrder },
      }),
      prisma.{resource}.count({ where }),
    ]);

    return { items, total };
  }

  async findById(id: string) {
    const item = await prisma.{resource}.findUnique({ where: { id } });
    if (!item) throw { message: "{RESOURCE}_NOT_FOUND", statusCode: 404 };
    return item;
  }

  async update(id: string, data: Update{Resource}Input) {
    await this.findById(id); // throws if not found
    return prisma.{resource}.update({ where: { id }, data });
  }

  async delete(id: string) {
    await this.findById(id);
    return prisma.{resource}.delete({ where: { id } });
  }
}

export const {resource}Service = new {Resource}Service();
```

Service rules:
- Use the DETECTED ORM syntax (Prisma, Mongoose, TypeORM, Sequelize, etc.)
- All input parameters use types derived from Zod schemas
- Include pagination, search, and filter logic for list operations
- Throw structured errors with message and statusCode
- Keep DB queries in services, NEVER in controllers

---

##### NestJS

Generate: Module, Controller (with decorators), Service, DTOs (with class-validator), Entity, Swagger decorators.

```typescript
// {resource}.dto.ts
import { IsString, IsNumber, IsEnum, IsOptional, Min, Max } from 'class-validator';
import { ApiProperty, ApiPropertyOptional } from '@nestjs/swagger';

export class Create{Resource}Dto {
  @ApiProperty({ description: 'Title', minLength: 1, maxLength: 200 })
  @IsString()
  title: string;

  @ApiProperty({ description: 'Amount', minimum: 0 })
  @IsNumber()
  @Min(0)
  amount: number;

  @ApiPropertyOptional({ enum: ['draft', 'sent', 'paid'], default: 'draft' })
  @IsOptional()
  @IsEnum(['draft', 'sent', 'paid'])
  status?: string;
}
```

Follow NestJS conventions: `@Controller`, `@Get`, `@Post`, `@ApiTags`, `@ApiBearerAuth`, `@ApiResponse`.

---

##### FastAPI + Python

Generate: Router, Pydantic models, Service, SQLAlchemy model (if detected).

```python
# schemas/{resource}.py
from pydantic import BaseModel, Field
from enum import Enum
from typing import Optional

class {Resource}Status(str, Enum):
    draft = "draft"
    sent = "sent"
    paid = "paid"

class Create{Resource}(BaseModel):
    title: str = Field(..., min_length=1, max_length=200)
    amount: float = Field(..., ge=0)
    status: {Resource}Status = {Resource}Status.draft

class Update{Resource}(BaseModel):
    title: Optional[str] = Field(None, min_length=1, max_length=200)
    amount: Optional[float] = Field(None, ge=0)
    status: Optional[{Resource}Status] = None

# routes/{resource}.py
from fastapi import APIRouter, Depends, Query, HTTPException

router = APIRouter(prefix="/{resources}", tags=["{Resources}"])

@router.post("/", response_model=ApiResponse[{Resource}Response], status_code=201)
async def create_{resource}(data: Create{Resource}, db: Session = Depends(get_db)):
    ...

@router.get("/", response_model=PaginatedResponse[{Resource}Response])
async def list_{resources}(page: int = Query(1, ge=1), limit: int = Query(10, ge=1, le=100), db: Session = Depends(get_db)):
    ...
```

---

##### Go (Gin/Echo/Chi)

Generate: Handler, Model structs with json tags, Validation (go-playground/validator), OpenAPI comments.

```go
// models/{resource}.go
type Create{Resource}Request struct {
    Title  string  `json:"title" binding:"required,min=1,max=200"`
    Amount float64 `json:"amount" binding:"required,min=0"`
    Status string  `json:"status" binding:"omitempty,oneof=draft sent paid"`
}

type {Resource}Response struct {
    ID        string    `json:"id"`
    Title     string    `json:"title"`
    Amount    float64   `json:"amount"`
    Status    string    `json:"status"`
    CreatedAt time.Time `json:"createdAt"`
}
```

---

##### Django REST Framework

Generate: Serializer, ViewSet, URL config, Model.

```python
# serializers.py
class {Resource}Serializer(serializers.ModelSerializer):
    class Meta:
        model = {Resource}
        fields = ['id', 'title', 'amount', 'status', 'created_at']
        read_only_fields = ['id', 'created_at']

# views.py
class {Resource}ViewSet(viewsets.ModelViewSet):
    queryset = {Resource}.objects.all()
    serializer_class = {Resource}Serializer
    permission_classes = [IsAuthenticated]
    filter_backends = [SearchFilter, OrderingFilter]
    search_fields = ['title']
    ordering_fields = ['created_at', 'amount']
```

---

##### Laravel (PHP)

Generate: Controller, FormRequest, Resource (API Resource), Model, Migration, Route.

```php
// app/Http/Requests/Create{Resource}Request.php
class Create{Resource}Request extends FormRequest {
    public function rules(): array {
        return [
            'title' => 'required|string|min:1|max:200',
            'amount' => 'required|numeric|min:0',
            'status' => 'nullable|in:draft,sent,paid',
        ];
    }
}
```

---

##### Ruby on Rails

Generate: Controller, Model, Serializer, Migration, Route.

---

##### Spring Boot (Kotlin/Java)

Generate: Controller, DTO (with Jakarta validation), Service, Repository, Entity.

```kotlin
@RestController
@RequestMapping("/api/v1/{resources}")
class {Resource}Controller(private val service: {Resource}Service) {

    @PostMapping
    @ResponseStatus(HttpStatus.CREATED)
    fun create(@Valid @RequestBody dto: Create{Resource}Dto): ApiResponse<{Resource}Response> {
        return ApiResponse.success(service.create(dto))
    }
}

data class Create{Resource}Dto(
    @field:NotBlank @field:Size(min = 1, max = 200)
    val title: String,
    @field:Min(0)
    val amount: BigDecimal,
    val status: {Resource}Status = {Resource}Status.DRAFT
)
```

---

#### Step 0.4: REGISTER — Wire Up the New Endpoint

After generating files, show the user what needs to be wired:

1. **Register route** — add import and `router.use("/{resources}", {resource}Routes)` in the main routes file
2. **Register i18n keys** — add translation keys for success/error messages
3. **Add permissions** — add new permission strings to the permission seed/config
4. **Database migration** — create table/collection (Prisma migrate, Alembic, Rails migration, etc.)

Provide the exact code snippets to paste, with file paths.

#### Step 0.5: FRONTEND TYPES — Generate Client-Side Contract

After backend is generated, also generate frontend types that match:

1. **TypeScript interfaces** for the new resource (for React/Vue/Angular/Svelte)
2. **Dart models** with fromJson/toJson (for Flutter)
3. **API hook/function** for calling the new endpoint (matching detected frontend pattern)

```typescript
// Frontend: types/{resource}.interface.ts
export interface {Resource}Type {
  id: string;
  title: string;
  amount: number;
  status: "draft" | "sent" | "paid";
  createdAt: string;
}

export interface {Resource}sAPIResponse {
  items: {Resource}Type[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}
```

#### Step 0.6: CHECKLIST — Contract Compliance Verification

After generation, run through this checklist and report pass/fail:

```
Contract Compliance Checklist
=============================

VALIDATION & TYPES:
  [PASS/FAIL] Validation schema exists for every POST/PUT endpoint
  [PASS/FAIL] Validation middleware applied on every route
  [PASS/FAIL] All types derived from validation schemas (no duplicate definitions)
  [PASS/FAIL] Frontend types match backend response shape

RESPONSE FORMAT:
  [PASS/FAIL] Response uses standard envelope (ApiResponse / sendSuccess)
  [PASS/FAIL] Paginated list uses standard pagination shape
  [PASS/FAIL] Error responses follow consistent format (RFC 9457 recommended)
  [PASS/FAIL] No raw res.json() — all responses through helpers

DOCUMENTATION:
  [PASS/FAIL] Swagger/OpenAPI JSDoc on every route
  [PASS/FAIL] All response codes documented (200, 201, 400, 401, 403, 404, 500)
  [PASS/FAIL] API version included in route prefix and Swagger info

SECURITY (OWASP):
  [PASS/FAIL] Auth middleware applied on protected routes
  [PASS/FAIL] Permission checks on every route (if RBAC detected)
  [PASS/FAIL] List endpoint limit param bounded (max 100)
  [PASS/FAIL] Sensitive fields excluded from response (password, tokens)
  [PASS/FAIL] 429 Too Many Requests documented in Swagger

PRODUCTION READINESS:
  [PASS/FAIL] Request correlation (X-Request-Id) supported or recommended
  [PASS/FAIL] Idempotency-Key documented for POST endpoints
  [PASS/FAIL] i18n message keys used (if i18n detected)
  [PASS/FAIL] Timestamps (createdAt/updatedAt) included in response
```

---

## ANALYZE MODE — Work with Existing API Responses

### Phase 1: RECEIVE — Accept Raw JSON Input

Ask the user to provide:

1. **One or more raw JSON API responses** (paste directly or provide file path)
2. **Endpoint info** (optional): HTTP method, URL path, description
3. **Target languages** (optional): Which languages to generate for. If not specified, ask.

Accepted input formats:
- Raw JSON pasted in chat
- File path to a `.json` file
- URL to a live API endpoint (use curl/fetch to get response)
- Multiple responses for the same endpoint (for comparison)

### Phase 2: ANALYZE — Deep Recursive Inference

Walk the entire JSON tree recursively and build a type map. For EVERY field at EVERY nesting level:

#### 2.1 Type Detection Rules

| JSON Value | Inferred Type | Notes |
|---|---|---|
| `"string"` | `string` | Check format: email, date, datetime, uuid, url, iso8601 |
| `123` | `number` | Distinguish integer vs float if all values are whole numbers |
| `true/false` | `boolean` | |
| `null` | `nullable` | Mark parent field as `T \| null` |
| `[]` | `array<T>` | Infer T from array items. If empty, mark as `array<unknown>` and flag |
| `{}` | `object` | Recurse into nested object. If empty `{}`, flag as ambiguous |
| Mixed array | `union` | If array items have different shapes, create discriminated union |

#### 2.2 Format Detection

Automatically detect and annotate these string formats:
- **UUID**: matches `/^[0-9a-f]{8}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{4}-[0-9a-f]{12}$/i`
- **ObjectId**: matches `/^[0-9a-f]{24}$/i` (MongoDB)
- **Email**: matches `/@/` with domain
- **ISO DateTime**: matches ISO 8601 pattern
- **URL**: starts with `http://` or `https://`
- **JWT**: matches `xxxxx.xxxxx.xxxxx` base64 pattern
- **Enum candidate**: if a string field has few distinct values across samples, suggest enum

#### 2.3 Nullable vs Optional Detection

This is critical. Apply these rules:

- **Field exists with value `null`** → Type is `T | null` (nullable)
- **Field exists in some responses but missing in others** → Type is `T | undefined` (optional)
- **Field exists with value `null` in some, missing in others** → Type is `T | null | undefined` (nullable + optional)
- **Field exists with empty string `""`** → Flag: is this intentional or should it be null?
- **Field exists with empty object `{}`** → Flag: is this intentional or should it be null/omitted?
- **Field exists with empty array `[]`** → Type is `array<T>`, not nullable (empty is valid)

#### 2.4 Deeply Nested Object Handling

For objects nested more than 2 levels deep:
1. Create a **named sub-type** for each nested object (e.g., `MealSlot`, `NutritionDetail`)
2. Do NOT inline deep structures — always extract to named types
3. Track the full JSON path (e.g., `data.items[].mealSchedule[].days.monday.meals.breakfast`)
4. If a nested object has dynamic keys (like day names), detect and use `Record<string, T>` or equivalent

#### 2.5 Pagination Detection

Auto-detect common pagination patterns:

| Pattern | Fields |
|---|---|
| Offset-based | `page`, `limit`, `total`, `totalPages` |
| Cursor-based | `cursor`, `next_cursor`, `has_more` |
| Extended offset | `hasPrevPage`, `hasNextPage`, `prevPage`, `nextPage`, `pagingCounter` |
| Wrapper variations | `data.items[]`, `data.results[]`, `data.records[]`, `items[]`, `results[]` |

Extract pagination as a **separate generic type** (e.g., `PaginatedResponse<T>`).

#### 2.6 Response Envelope Detection

Detect the wrapper/envelope pattern:

```
Common patterns:
  { success, message, data }
  { statusCode, message, data }
  { status, data, error }
  { code, message, result }
  { ok, data, errors }
```

Extract envelope as a **separate generic type** (e.g., `ApiResponse<T>`), so the actual payload type is clean.

### Phase 3: REPORT — Inconsistency Detection

If the user provided multiple responses (or if a single response has repeated structures), compare and report:

#### 3.1 Inconsistency Categories

**CRITICAL (will cause runtime crashes):**
- Field exists in one response but completely missing in another
- Field is `null` in one response but an object in another
- Field type differs between responses (string vs number)
- ID field naming inconsistency (`_id` vs `id` vs `uuid`)
- Array item shapes differ across items in the same array

**WARNING (may cause silent bugs):**
- Optional field present in some items, absent in others (within same array)
- Empty object `{}` used where a populated object is expected
- Empty string `""` used where `null` might be more appropriate
- Numeric string `"1"` where a number `1` might be expected
- Inconsistent casing (`camelCase` mixed with `snake_case`)

**INFO (style recommendations):**
- Pagination field differences between endpoints
- Response envelope differences between endpoints
- Date format inconsistencies (ISO vs Unix timestamp)
- Boolean representation (`true/false` vs `1/0` vs `"yes"/"no"`)

#### 3.2 Report Format

Present the report as a clear table:

```
API Contract Analysis Report
============================

Endpoint: GET /shared/files
Nesting Depth: 6 levels
Total Fields: 47 unique fields
Named Types Needed: 8

CRITICAL Issues (2):
  [C1] Field 'hasLeftOver' — present in 4/7 meal slots, missing in 3
       Path: data.items[].mealPlan.mealSchedule[].days.*.meals.*.hasLeftOver
       Decision needed: Should this be required or optional?

  [C2] Field 'nutritionDetail' — full object in active days, empty {} in inactive
       Path: data.items[].mealPlan.mealSchedule[].days.*.nutritionDetail
       Decision needed: Should inactive days return null or omit the field?

WARNINGS (3):
  [W1] Field 'serving' — string type but contains numeric values ("1", "2")
       Suggestion: Consider using number type
  ...

For each issue, choose a resolution before generating types.
```

#### 3.3 Interactive Resolution

For each CRITICAL and WARNING issue, present options and ask the user to decide:

```
[C1] Field 'hasLeftOver' is inconsistent. Choose:
  A) Make it required (add to all meal slots)
  B) Make it optional (may be undefined)
  C) Make it nullable + optional (T | null | undefined)
```

Only proceed to Phase 4 after all critical issues are resolved.

### Phase 4: GENERATE — Multi-Language Contract Artifacts

Generate contract code for the user's chosen target languages. Always ask which targets they want if not specified.

#### 4.1 TypeScript Interfaces

```typescript
// Auto-generated by API Contract Forge
// Source: GET /endpoint
// Generated: {timestamp}

export interface ApiResponse<T> {
  statusCode: number;
  message: string;
  data: T;
}

export interface PaginatedData<T> {
  items: T[];
  total: number;
  page: number;
  limit: number;
  totalPages: number;
}

// Named sub-types extracted from nesting
export interface MealSlot {
  recipe: string | null;
  isLeftOver: boolean;
  hasLeftOver?: boolean;  // Optional — not present in all responses
  serving: string;
}

// ... all other types with JSDoc comments explaining decisions
```

Rules for TypeScript generation:
- Use `interface` for object shapes, `type` for unions/aliases
- Mark nullable fields with `| null`
- Mark optional fields with `?`
- Add JSDoc comments for format annotations (`@format uuid`, `@format email`)
- Extract deeply nested objects (3+ levels) as named interfaces
- Use `Record<string, T>` for dynamic-key objects
- Export everything

#### 4.2 Zod Schemas (TypeScript Runtime Validation)

```typescript
import { z } from "zod";

export const mealSlotSchema = z.object({
  recipe: z.string().nullable(),
  isLeftOver: z.boolean(),
  hasLeftOver: z.boolean().optional(),
  serving: z.string(),
});

export type MealSlot = z.infer<typeof mealSlotSchema>;
```

Rules for Zod generation:
- Use `.nullable()` for `T | null`
- Use `.optional()` for fields that may be missing
- Use `.nullish()` for `T | null | undefined`
- Add `.email()`, `.uuid()`, `.url()`, `.datetime()` for detected formats
- Use `z.record()` for dynamic-key objects
- Use `z.discriminatedUnion()` where applicable
- Always derive TypeScript types with `z.infer<>`

#### 4.3 OpenAPI 3.1 Specification

```yaml
openapi: "3.1.0"
info:
  title: "API Contract"
  version: "1.0.0"
paths:
  /endpoint:
    get:
      summary: "Description"
      responses:
        "200":
          content:
            application/json:
              schema:
                $ref: "#/components/schemas/EndpointResponse"
components:
  schemas:
    MealSlot:
      type: object
      required: [recipe, isLeftOver, serving]
      properties:
        recipe:
          type: [string, "null"]
        hasLeftOver:
          type: boolean
          description: "Optional — not present in all responses"
```

Rules for OpenAPI generation:
- Use OpenAPI 3.1.0 (supports `type: [string, "null"]` for nullable)
- Define all types under `components/schemas`
- Use `$ref` for nested type references
- Mark `required` arrays accurately
- Add `description` for any field that had an inconsistency resolution
- Include `example` values from the original JSON

#### 4.4 Python Pydantic Models

```python
from pydantic import BaseModel, Field
from typing import Optional
from datetime import datetime

class MealSlot(BaseModel):
    recipe: str | None
    is_left_over: bool = Field(alias="isLeftOver")
    has_left_over: bool | None = Field(None, alias="hasLeftOver")
    serving: str
```

Rules for Python generation:
- Use Pydantic v2 syntax
- Use `Field(alias="...")` when JSON uses camelCase but Python uses snake_case
- Use `str | None` for nullable, `Optional[str]` for optional with default None
- Add `model_config = ConfigDict(populate_by_name=True)` for alias support

#### 4.5 Dart Model Classes (Flutter)

```dart
class MealSlot {
  final String? recipe;
  final bool isLeftOver;
  final bool? hasLeftOver;
  final String serving;

  MealSlot({
    required this.recipe,
    required this.isLeftOver,
    this.hasLeftOver,
    required this.serving,
  });

  factory MealSlot.fromJson(Map<String, dynamic> json) => MealSlot(
    recipe: json['recipe'] as String?,
    isLeftOver: json['isLeftOver'] as bool,
    hasLeftOver: json['hasLeftOver'] as bool?,
    serving: json['serving'] as String,
  );

  Map<String, dynamic> toJson() => {
    'recipe': recipe,
    'isLeftOver': isLeftOver,
    if (hasLeftOver != null) 'hasLeftOver': hasLeftOver,
    'serving': serving,
  };
}
```

Rules for Dart generation:
- Use `final` fields with named constructor parameters
- `required` for non-optional fields, no keyword for optional
- Use `?` suffix for nullable types
- Always generate `fromJson` factory and `toJson` method
- Handle null checks in fromJson with `as Type?`
- Omit null optional fields in toJson

#### 4.6 Kotlin Data Classes (Android)

```kotlin
import kotlinx.serialization.Serializable
import kotlinx.serialization.SerialName

@Serializable
data class MealSlot(
    val recipe: String?,
    @SerialName("isLeftOver") val isLeftOver: Boolean,
    @SerialName("hasLeftOver") val hasLeftOver: Boolean? = null,
    val serving: String
)
```

Rules for Kotlin generation:
- Use `@Serializable` from kotlinx.serialization
- Use `@SerialName` for JSON field mapping
- Use `?` for nullable, `= null` default for optional
- Use `data class` for all models

#### 4.7 Swift Codable Structs (iOS)

```swift
struct MealSlot: Codable {
    let recipe: String?
    let isLeftOver: Bool
    let hasLeftOver: Bool?
    let serving: String
}
```

Rules for Swift generation:
- Use `struct` conforming to `Codable`
- Use `?` for both nullable and optional
- Add `CodingKeys` enum only if JSON keys differ from Swift naming
- Use `let` for immutable properties

#### 4.8 JSON Schema (Universal)

```json
{
  "$schema": "https://json-schema.org/draft/2020-12/schema",
  "type": "object",
  "required": ["recipe", "isLeftOver", "serving"],
  "properties": {
    "recipe": { "type": ["string", "null"] },
    "isLeftOver": { "type": "boolean" },
    "hasLeftOver": { "type": "boolean" },
    "serving": { "type": "string" }
  }
}
```

### Phase 5: COMPARE — Breaking Change Detection (Optional)

If the user provides two versions of a response (old vs new), detect breaking changes:

#### Breaking Changes (will crash clients):
- Removed field that was previously required
- Type changed (string to number, object to array)
- Required field added (old clients won't send it)
- Enum value removed
- Nullable field became non-nullable
- Array item shape changed

#### Non-Breaking Changes (safe to deploy):
- New optional field added
- New enum value added
- Non-nullable field became nullable
- New endpoint added
- Description/example changed

#### Output Format:
```
Breaking Change Report: GET /endpoint
======================================
Version: v1 → v2

BREAKING (3):
  [B1] REMOVED field 'difficulty_level' from ScenarioType
       Impact: Frontend will receive undefined
       Action: Update all clients before deploying

  [B2] RENAMED 'items' to 'records' in pagination wrapper
       Impact: All list parsing will break
       Action: Use 'items' or version the endpoint

NON-BREAKING (2):
  [NB1] ADDED optional field 'category' to MealPlan
  [NB2] ADDED new enum value 'draft' to status field
```

### Phase 6: VALIDATE — Check Existing Code Against Contract (Optional)

If the user has an existing codebase, scan it to check if the code matches the contract:

1. Find TypeScript interfaces / Pydantic models / Dart classes in the project
2. Compare field names, types, and nullable/optional markers against the inferred contract
3. Report mismatches:
   - Frontend type says `string` but API returns `string | null`
   - Frontend type missing a field that API sends
   - Frontend type has extra field that API doesn't send

## Industry Standards — Production Best Practices

These standards MUST be followed in both BUILD and ANALYZE modes. They are based on current (2025-2026) industry best practices from OWASP, RFC 9457, OpenAPI Initiative, and API governance guidelines.

### Standard 1: Error Format — RFC 9457 (Problem Details)

All error responses MUST follow RFC 9457 (supersedes RFC 7807). This is the industry standard for HTTP API errors.

**Required error shape:**
```json
{
  "type": "https://api.example.com/errors/validation-error",
  "title": "Validation Error",
  "status": 400,
  "detail": "Field 'email' must be a valid email address",
  "instance": "/invoices",
  "traceId": "abc-123-def",
  "errors": [
    { "field": "email", "message": "Must be a valid email", "code": "INVALID_FORMAT" }
  ]
}
```

**Required fields:** `type` (URI reference), `title` (human-readable), `status` (HTTP status code)
**Recommended fields:** `detail` (specific explanation), `instance` (request path), `traceId` (correlation ID)
**Optional extension:** `errors[]` (field-level validation details)

In BUILD mode: If the project already has a custom error format (like `{ success: false, message, error }`), use THAT format but recommend RFC 9457 as a comment. Don't force migration — respect existing conventions.

In ANALYZE mode: Flag non-RFC-9457 error responses as INFO (not CRITICAL), since many projects use custom formats.

### Standard 2: API Versioning

Every API MUST have a versioning strategy. In BUILD mode, detect and follow the existing project's approach:

| Strategy | Pattern | When to Use |
|----------|---------|-------------|
| **URL path** (recommended) | `/api/v1/invoices` | Most common, clearest for consumers |
| **Header** | `Accept: application/vnd.api.v1+json` | When URL stability matters |
| **Query param** (discouraged) | `/api/invoices?version=1` | Only for legacy compat |

BUILD mode rules:
- Always include version in route prefix (detect from existing routes)
- Document version in Swagger JSDoc
- When generating Swagger, include `info.version` field
- Add `deprecated: true` annotation for deprecated endpoints/fields

### Standard 3: Request Correlation — Trace IDs

Every API response SHOULD include correlation headers for debugging and distributed tracing:

```
Response Headers:
  X-Request-Id: uuid-generated-per-request
  X-Trace-Id: uuid-or-from-incoming-header
```

BUILD mode: If the project has request logging middleware, ensure it generates/forwards a request ID. Add `X-Request-Id` to response headers.

ANALYZE mode: Check if responses include correlation headers. Flag as INFO if missing.

### Standard 4: Idempotency Keys

POST and PUT endpoints that create or modify resources SHOULD support idempotency:

```
Request Header:
  Idempotency-Key: client-generated-uuid
```

This prevents duplicate resource creation from network retries. In BUILD mode:
- Add `Idempotency-Key` as optional header in Swagger JSDoc
- If the project has idempotency middleware, wire it up
- If not, add a comment recommending it for production

### Standard 5: Rate Limiting Headers

All APIs SHOULD communicate rate limit status via standard headers:

```
Response Headers:
  X-RateLimit-Limit: 100
  X-RateLimit-Remaining: 95
  X-RateLimit-Reset: 1609459200
  Retry-After: 60  (only on 429 responses)
```

BUILD mode: Document 429 response in Swagger JSDoc. If the project has rate limiting middleware, reference it.

ANALYZE mode: Check if 429 responses are documented. Flag as INFO if missing.

### Standard 6: Deprecation Strategy

Fields, parameters, and endpoints MUST be deprecated before removal:

```yaml
# In OpenAPI spec
paths:
  /old-endpoint:
    get:
      deprecated: true
      description: "Use /new-endpoint instead. Removal date: 2026-06-01"
```

BUILD mode rules:
- Never generate deprecated patterns
- If replacing an existing endpoint, mark the old one as `deprecated: true` in Swagger
- Include deprecation timeline in description

### Standard 7: API Linting and CI Validation

Recommend Spectral (or equivalent) for automated API contract linting in CI/CD:

```yaml
# .spectral.yml example
extends: "spectral:oas"
rules:
  operation-operationId: error
  operation-description: warn
  oas3-valid-schema-example: error
```

In the compliance checklist (Step 0.6), add: "Is there an API linting step in CI?"

### Standard 8: ETags and Conditional Requests

GET endpoints returning single resources SHOULD support conditional requests:

```
Response Headers:
  ETag: "33a64df551425fcc55e4d42a148795d9f25f89d4"

Subsequent Request Headers:
  If-None-Match: "33a64df551425fcc55e4d42a148795d9f25f89d4"
  → Returns 304 Not Modified if unchanged
```

BUILD mode: Add ETag support as a comment/recommendation for GET /:id endpoints. Don't generate the middleware (that's infrastructure), but document it in Swagger.

### Standard 9: Pagination — Support Both Patterns

BUILD mode should offer both pagination strategies:

**Offset-based** (default, for small-medium datasets):
```json
{ "items": [], "total": 100, "page": 1, "limit": 10, "totalPages": 10 }
```

**Cursor-based** (for large/real-time datasets):
```json
{ "items": [], "nextCursor": "eyJpZCI6MTAwfQ==", "hasMore": true }
```

Ask the user which pattern to use. Default to offset-based if not specified.

### Standard 10: OWASP API Security Compliance

In both BUILD and ANALYZE modes, check against OWASP API Security Top 10 (2023):

| OWASP Risk | What to Check |
|------------|--------------|
| **API1: Broken Object Level Auth** | Does GET /:id check ownership? Don't just check auth — check if the user owns the resource |
| **API2: Broken Authentication** | Is auth middleware on every protected route? |
| **API3: Broken Object Property Level Auth** | Are sensitive fields (password, token) excluded from responses? |
| **API4: Unrestricted Resource Consumption** | Is there rate limiting? Is `limit` param bounded (max 100)? |
| **API5: Broken Function Level Auth** | Are admin-only endpoints protected with permission checks? |
| **API6: Unrestricted Access to Sensitive Flows** | Are critical operations (delete, status change) protected? |
| **API7: Server-Side Request Forgery** | Are URL inputs validated? |
| **API8: Security Misconfiguration** | Is CORS configured? Are error details hidden in production? |
| **API9: Improper Inventory Management** | Is every endpoint documented in Swagger? |
| **API10: Unsafe API Consumption** | Are third-party API responses validated? |

BUILD mode: Ensure generated code handles API1 (ownership check in service layer), API3 (exclude sensitive fields in select/projection), API4 (bounded limit param).

---

## Anti-Patterns to Detect and Flag

Always check for these common API anti-patterns:

1. **Mixed naming conventions** — camelCase and snake_case in same response
2. **Inconsistent ID fields** — `_id`, `id`, `uuid`, `ID` in same API
3. **Boolean as string** — `"true"` instead of `true`
4. **Number as string** — `"1"` instead of `1` (especially `serving`, `page`, `limit`)
5. **Inconsistent null representation** — `null` vs `""` vs `{}` vs missing field
6. **Over-nested responses** — data buried 5+ levels deep without clear reason
7. **Inconsistent pagination** — different list endpoints using different pagination shapes
8. **Inconsistent envelope** — different endpoints wrapping data differently
9. **Sensitive data exposure** — JWT tokens, passwords in responses that shouldn't have them (OWASP API3)
10. **Missing timestamps** — created/updated records without `createdAt`/`updatedAt`
11. **No error traceability** — error responses without request/trace ID (Standard 3)
12. **Unbounded list queries** — list endpoints without max limit (OWASP API4)
13. **Missing 429 documentation** — no rate limit responses in Swagger (Standard 5)
14. **No deprecation path** — breaking changes without deprecation notice (Standard 6)

## Output Rules

1. **Always generate a header comment** with source endpoint, timestamp, and tool attribution
2. **Always extract nested types** — never inline objects deeper than 2 levels
3. **Always separate envelope/pagination** from payload types
4. **Always add comments** on fields where an inconsistency was resolved
5. **Always ask before generating** — confirm target languages with user
6. **Never guess on ambiguous types** — ask the user to decide
7. **Generate one file per language** — don't mix TypeScript and Dart in one output
8. **Use the language's conventions** — snake_case for Python, camelCase for TypeScript/Dart/Kotlin, etc.
9. **Include example values** as comments where helpful

## Quick Reference

| User Says | Mode | Phases |
|---|---|---|
| "Build a new invoice API" | BUILD | 0.1 → 0.2 → 0.3 → 0.4 → 0.5 → 0.6 |
| "Create CRUD for meal-plans" | BUILD | 0.1 → 0.2 → 0.3 → 0.4 → 0.5 → 0.6 |
| "Add a new endpoint for appointments" | BUILD | 0.1 → 0.2 → 0.3 → 0.4 → 0.5 → 0.6 |
| "Scaffold a users API with Express + Zod" | BUILD | 0.1 → 0.2 → 0.3 → 0.4 → 0.5 → 0.6 |
| "Here's my API response" + JSON | ANALYZE | 1 → 2 → 3 → 4 |
| "Compare these two responses" | ANALYZE | 1 → 2 → 3 → 5 |
| "Check my types against this response" | ANALYZE | 1 → 2 → 6 |
| "Generate TypeScript types" | ANALYZE | 4 (TypeScript only) |
| "What's wrong with this API response?" | ANALYZE | 2 → 3 (report only) |
| "Normalize this API" | ANALYZE | 2 → 3 → 4 (envelope standardization) |