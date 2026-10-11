# WJIH Opportunities Board

A bilingual Arabic/English jobs and internships platform for Wadi Jeddah Innovation Hub. Companies publish opportunities, and candidates apply without creating an account.

The backend uses TypeScript, NestJS, Prisma, and PostgreSQL, with pg-boss for background jobs and Socket.IO for realtime updates.

## Database development

Database changes are delivered through the Prisma schema, version-controlled SQL migrations, and targeted validation scripts.

| Stage | Scope | Status |
|---|---|---|
| Stage one | Active-membership uniqueness, optional applicant city and major, company contact and headquarters fields, opportunity city and requirements | Implemented and committed in `d5821cd` |
| Stage two | Industry catalog and company relationship, `website_url` rename, guarded removal of `required_skills` | Local changes under review; completion and validation are not yet confirmed |

The overall database redesign is in progress. Database implementation and backend integration are tracked separately: adding or changing database fields does not automatically update API validation, persistence, or responses.

### Review references

| Reference | Purpose |
|---|---|
| [Design overview](docs/database-design/README.md) | Scope, conventions, and document navigation |
| [Prisma schema](apps/api/prisma/schema.prisma) | Schema represented by this branch |
| [Migrations](apps/api/prisma/migrations/) | Version-controlled database changes |
| [Relationships](docs/database-design/relationships.md) | Target relationships, including deferred elements |
| [Data dictionary](docs/database-design/data-dictionary.md) | Field definitions and requirements |
| [Change map](docs/database-design/change-map.md) | Implementation status, migration planning, and backend handoff |
| [Validation](docs/database-design/validation/) | Stage-specific database checks |

Design documents distinguish approved requirements, implemented changes, and deferred proposals. Target diagrams do not imply that every illustrated component has been implemented.

## Repository structure

| Path | Contents |
|---|---|
| `apps/api/` | NestJS backend |
| `apps/api/prisma/` | Prisma schema and SQL migrations |
| `packages/shared/` | Shared schemas, enums, bilingual content utilities, and API contracts |
| `docs/database-design/` | Database design, implementation handoff, and validation |
| `docs/` | API and project documentation |

The React frontend is not included in this repository's current application structure.

## Local development

Requires Node.js 22.12 or later and PostgreSQL. The Docker configuration uses PostgreSQL 17.

The commands below describe the existing backend setup. Before using the updated database schema, review the [backend handoff](docs/database-design/change-map.md) for compatibility requirements. Existing seed scripts and application tests may require adaptation.

From the repository root:

```sh
npm ci
```

Create `apps/api/.env` from `apps/api/.env.example`, then configure the intended development database, application URL, and API port.

To start the configured PostgreSQL service:

```sh
docker compose up -d postgres
```

From `apps/api`, after verifying that `DATABASE_URL` points to the intended development database:

```sh
npx prisma generate
npx prisma migrate deploy
npm run start:dev
```

The default API address is `http://localhost:4100`, with Swagger documentation at `/docs`.

### Development authentication

Authentication uses magic links. For an existing development account:

```sh
npm run cli -- create-magic-link <email>
```

Use development-only login tools in the local environment.

## Validation

Run database validation scripts only against their documented target database and schema stage. A script for an earlier stage may intentionally depend on fields removed later.

| Command | Purpose |
|---|---|
| `npx prisma validate` | Validate the Prisma schema |
| `npx prisma generate` | Generate the Prisma client |
| `npx prisma migrate status` | Check migration status for the configured database |
| `npm run lint` in `apps/api` | Run the backend type check |
| `npm test` in `apps/api` | Run application tests against an isolated test database |
| `npm run check` from the repository root | Run the project build and test checks |

Application test helpers can clear database tables. Use a dedicated test database and verify its connection before execution.

Prisma validation and successful database migrations do not establish that unchanged backend code supports every new database requirement.

## Frontend and backend integration

- Use the shared request schemas and enums from `packages/shared`, updating them alongside API changes.
- Preserve Arabic/English content and RTL support.
- Follow the authentication and realtime contracts documented in [API contracts](docs/api-contract.md).
- Coordinate renamed or removed fields across inputs, persistence, responses, and frontend consumers.
- Review older API and workflow documentation against the current [database change map](docs/database-design/change-map.md); some documents describe previous product behavior.

## Database tools

- **PostgreSQL:** database server.
- **Prisma schema and migrations:** version-controlled structural changes.
- **pgAdmin:** inspection, SQL queries, and validation.

Manual structural changes in pgAdmin do not replace migrations.

## Deployment

Coordinate database migrations with the backend changes that depend on them. Verify existing data against migration preconditions before deployment.

Configure production database, mail, storage, application URL, and proxy settings for the target environment. Keep credentials out of version control.

Apply reviewed migrations with:

```sh
npx prisma migrate deploy
```

Database changes that rename or remove fields require compatible backend code before application traffic resumes.
