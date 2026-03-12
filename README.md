# API Contract Forge

A Claude skill for API contracting. Build new APIs with best practices or analyze existing API responses to generate typed contracts.

## Features

### BUILD Mode
- Auto-detects your project's tech stack (Express, NestJS, FastAPI, Go, Django, Laravel, Spring Boot, Rails, and more)
- Scaffolds production-ready CRUD endpoints with validation, typed responses, Swagger docs, and RBAC permissions
- Runs a contract compliance checklist (OWASP, RFC 9457, pagination, i18n)

### ANALYZE Mode
- Paste a raw JSON API response and get typed contracts in any language
- Detects inconsistencies, nullable vs optional fields, and anti-patterns
- Compares two response versions to detect breaking changes
- Generates artifacts: TypeScript interfaces, Zod schemas, Pydantic models, OpenAPI 3.1, JSON Schema

## Supported Tech Stacks

| Framework | Language | What it generates |
|-----------|----------|-------------------|
| Express | TypeScript | Routes, Controller, Service, Zod types, Swagger JSDoc |
| NestJS | TypeScript | Module, Controller, Service, DTOs, Swagger decorators |
| FastAPI | Python | Router, Pydantic models, Service, SQLAlchemy model |
| Django REST | Python | Serializer, ViewSet, URL config, Model |
| Gin/Echo/Chi | Go | Handler, Model structs, Validation, OpenAPI comments |
| Spring Boot | Kotlin/Java | Controller, DTO, Service, Repository, Entity |
| Laravel | PHP | Controller, FormRequest, Resource, Model, Migration |
| Rails | Ruby | Controller, Model, Serializer, Migration, Route |

## Skill Structure

```
api-contract-forge/
├── SKILL.md                              # Core workflow (~470 lines)
├── references/
│   ├── build-express.md                  # Express + TypeScript templates
│   ├── build-nestjs.md                   # NestJS templates
│   ├── build-fastapi.md                  # FastAPI + Python templates
│   ├── build-django.md                   # Django REST templates
│   ├── build-go.md                       # Go (Gin/Echo/Chi) templates
│   ├── build-spring-boot.md              # Spring Boot Kotlin/Java templates
│   ├── build-laravel.md                  # Laravel PHP templates
│   ├── build-rails.md                    # Rails Ruby templates
│   ├── analyze-generators.md             # Backend language generators
│   └── industry-standards.md             # Standards 1-10 (OWASP, RFC 9457, etc.)
└── evals/
    └── evals.json                        # Test cases
```

## Usage

Just tell Claude what you need:

```
# BUILD mode
"Build a new API for notifications"
"Create CRUD for invoices"
"Add a new endpoint for appointments"
"Scaffold a users API with Express + Zod"

# ANALYZE mode
"Here's my API response: { ... }"
"Generate TypeScript types from this JSON"
"Compare these two API responses for breaking changes"
"Check my types against this API response"
```

## Trigger Keywords

`build api` | `new endpoint` | `api contract` | `generate types` | `infer schema` | `normalize API` | `scaffold api` | `create api for [resource]` | `type safety` | `breaking changes` | `API schema`

## License

MIT
