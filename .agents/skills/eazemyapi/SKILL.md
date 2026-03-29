---
name: eazemyapi
description: >
  Expert knowledge for building with EazeMyAPI — an AI-powered no-code backend builder that auto-generates REST APIs from database tables.
  Use this skill whenever the user mentions EazeMyAPI, asks how to integrate their frontend with EazeMyAPI, wants to write fetch/axios code
  against EazeMyAPI endpoints, needs help designing a database schema for an EazeMyAPI project, is writing custom SQL queries to expose as
  EazeMyAPI endpoints, wants to understand the URL structure or authentication pattern, or is building any SaaS/app on top of EazeMyAPI as
  their backend. Also trigger for questions about EazeMyAPI projects by name (e.g. "my-project"), or whenever X-API-SIGNATURE appears in context.
---

# EazeMyAPI Skill

EazeMyAPI is an AI-powered backend-as-a-service. You describe your app → AI designs the database → REST APIs are auto-generated instantly. No backend server code required.

**Base API URL:** `https://api.eazemyapi.com`  
**Docs:** `https://doc.eazemyapi.com`  
**Dashboard:** `https://app.eazemyapi.com`

---

## URL Structure

EazeMyAPI has **two distinct URL patterns** depending on the type of endpoint.

### CRUD Endpoints (auto-generated per table)

```
https://api.eazemyapi.com/{project-name}/{table}/{version}/{action}
```

| Part | Description |
|------|-------------|
| `project-name` | Your project slug (e.g. `soche-india`) |
| `table` | The table name (e.g. `posts`, `users`) |
| `version` | API version — `v1`, `v2`, `v3`, etc. |
| `action` | CRUD action — `list`, `show/:id`, `create`, `update/:id`, `delete/:id` |

**Examples:**
```
https://api.eazemyapi.com/soche-india/posts/v2/list
https://api.eazemyapi.com/soche-india/posts/v2/show/:id
https://api.eazemyapi.com/soche-india/posts/v2/create
https://api.eazemyapi.com/soche-india/posts/v2/update/:id
https://api.eazemyapi.com/soche-india/posts/v2/delete/:id
```

### Custom / Raw Query Endpoints

```
https://api.eazemyapi.com/{project-name}/{version}/{custom-endpoint}
```

| Part | Description |
|------|-------------|
| `project-name` | Your project slug |
| `version` | API version |
| `custom-endpoint` | Name you gave the custom query (e.g. `posts-by-category`) |

**Example:**
```
https://api.eazemyapi.com/soche-india/v2/posts-by-category
```

> **Key difference:** CRUD URLs have the **table name between project and version**. Custom query URLs go directly `project → version → endpoint` with no table in the path.

---

## Authentication

Every request **must** include the API signature header:

```
X-API-SIGNATURE: <your-secret-key>
```

Find your key: `Project → Settings → API Signature Key`

**Security rules:**
- Never expose this key in frontend code directly
- Store in environment variables (`.env`, Vercel env vars, etc.)
- For public frontend apps, proxy through a serverless function (e.g. Vercel API route)

---

## Auto-Generated CRUD Endpoints

When you create a table (e.g. `posts`), EazeMyAPI generates 5 endpoints automatically.
Pattern: `/{project}/{table}/{version}/{action}`

| Action | Full URL Example | Method | Notes |
|--------|-----------------|--------|-------|
| `list` | `.../posts/v2/list` | GET | Returns array of all records |
| `show/:id` | `.../posts/v2/show/1` | GET | ID in URL path |
| `create` | `.../posts/v2/create` | POST | JSON body |
| `update/:id` | `.../posts/v2/update/1` | POST | ID in URL, fields in JSON body |
| `delete/:id` | `.../posts/v2/delete/1` | DELETE | ID in URL path |

> **Note:** `update` uses **POST** (not PUT/PATCH). The record ID goes in the **URL path** for `show`, `update`, and `delete`.

---

## Response Format

All endpoints return this consistent shape:

```json
{
  "success": true,
  "message": "Data found.",
  "data": { ... }
}
```

- `success`: `true` or `false`
- `message`: human-readable status
- `data`: single object (show/create) or array (list)

**Error response:**
```json
{
  "success": false,
  "message": "Record not found.",
  "data": null
}
```

Always check `success` before using `data`.

---

## Code Patterns

Replace `my-project` / `posts` / `v2` with your actual project name, table, and version.

### List all records — GET
```js
const res = await fetch("https://api.eazemyapi.com/my-project/posts/v2/list", {
  headers: {
    "Content-Type": "application/json",
    "X-API-SIGNATURE": process.env.API_KEY
  }
});
const { success, data } = await res.json();
if (success) setPosts(data);
```

### Show single record — GET
```js
const id = 1;
const res = await fetch(`https://api.eazemyapi.com/my-project/posts/v2/show/${id}`, {
  headers: { "X-API-SIGNATURE": process.env.API_KEY }
});
const { success, data } = await res.json();
```

### Create record — POST
```js
const res = await fetch("https://api.eazemyapi.com/my-project/posts/v2/create", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "X-API-SIGNATURE": process.env.API_KEY
  },
  body: JSON.stringify({ title: "Hello World", content: "My first post" })
});
const { success, data } = await res.json();
```

### Update record — POST (with ID in URL)
```js
const id = 1;
const res = await fetch(`https://api.eazemyapi.com/my-project/posts/v2/update/${id}`, {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    "X-API-SIGNATURE": process.env.API_KEY
  },
  body: JSON.stringify({ title: "Updated Title" })
});
```

### Delete record — DELETE (with ID in URL)
```js
const id = 1;
const res = await fetch(`https://api.eazemyapi.com/my-project/posts/v2/delete/${id}`, {
  method: "DELETE",
  headers: { "X-API-SIGNATURE": process.env.API_KEY }
});
```

### Custom query endpoint — GET
```js
const res = await fetch("https://api.eazemyapi.com/my-project/v2/posts-by-category?category_id=abc", {
  headers: { "X-API-SIGNATURE": process.env.API_KEY }
});
// Note: custom queries use /{project}/{version}/{endpoint} — no table in path
```

### Vercel Serverless Proxy (secure API key for public frontends)
```js
// /api/proxy.js — handles both CRUD and custom endpoints
export default async function handler(req, res) {
  const { path, ...params } = req.query; // path = "posts/v2/list" or "v2/custom-endpoint"
  const qs = new URLSearchParams(params).toString();
  const url = `https://api.eazemyapi.com/my-project/${path}${qs ? "?" + qs : ""}`;

  const response = await fetch(url, {
    method: req.method,
    headers: {
      "Content-Type": "application/json",
      "X-API-SIGNATURE": process.env.EAZE_API_KEY
    },
    body: req.method === "POST" ? JSON.stringify(req.body) : undefined
  });

  const data = await response.json();
  res.status(response.status).json(data);
}
```

---

## Field Types

| Type | Use For | Example |
|------|---------|---------|
| `TEXT` | Short strings — names, titles | `"Romit"` |
| `PARAGRAPH` | Long text — bios, descriptions, content | `"Full post body..."` |
| `NUMBER` | Integers — age, count, ID | `42` |
| `DECIMAL` | Floats — price, rating | `4.8` |
| `DATE` | Calendar date | `"2026-03-29"` |
| `DATETIME` | Timestamp | `"2026-03-29T10:30:00Z"` |
| `EMAIL` | Email address | `"rm@eazemyapi.com"` |
| `PHONE` | Phone number | `"+91 9876543210"` |
| `PASSWORD` | Encrypted password | stored hashed |
| `BOOLEAN` | True/false flags | `true` |

---

## Custom Queries

For joins, aggregations, filters, or reports — write raw SQL in the dashboard and EazeMyAPI exposes it as an endpoint.

**Dashboard:** `Project → Custom Queries → Create Query`

**Example SQL:**
```sql
SELECT 
  categories.id AS category_id,
  categories.name AS category_name,
  posts.id AS post_id,
  posts.description
FROM categories
JOIN posts ON posts.category_id = categories.id
WHERE categories.name = 'Foods';
```

This auto-generates:
```
https://api.eazemyapi.com/my-project/v2/posts-by-category
```
Note: custom query URL has **no table segment** — `/{project}/{version}/{endpoint}` only.

**Parameterized queries** — use `:param` syntax in SQL:
```sql
SELECT * FROM posts WHERE category_id = :category_id;
```

Then call: `?category_id=abc-123`

---

## Schema Design Best Practices

When designing tables for EazeMyAPI:

1. **Always add `created_at` (DATETIME)** — useful for sorting and filtering
2. **Use `BOOLEAN` for status flags** — `is_active`, `is_published`, `is_verified`
3. **Use `TEXT` for slugs and IDs from external systems**
4. **Store foreign keys as `TEXT`** (UUIDs) or `NUMBER` (integer IDs)
5. **Use `PARAGRAPH` for any content > 255 chars**
6. **Name endpoints descriptively** — EazeMyAPI mirrors your table name, keep it clean

**Example `posts` table:**
| Field | Type |
|-------|------|
| id | NUMBER |
| title | TEXT |
| content | PARAGRAPH |
| author_id | NUMBER |
| category | TEXT |
| is_published | BOOLEAN |
| created_at | DATETIME |

---

## Common Gotchas

- **CRUD URLs include the table name**: `/{project}/{table}/{version}/{action}` — don't skip the table segment
- **Custom query URLs do NOT have a table**: `/{project}/{version}/{custom-endpoint}`
- **Update uses POST, not PUT/PATCH** — this is intentional in EazeMyAPI
- **ID goes in the URL path** for `show/:id`, `update/:id`, `delete/:id` — not as `?id=1`
- **Always check `success: true`** before reading `data` — don't assume
- **API key is project-scoped** — one key per project, shared across all tables
- **Versions are chosen by you** — `v1`, `v2`, etc., set in the dashboard when generating

---

## Integration Checklist

When helping a user integrate EazeMyAPI into their app:

- [ ] Confirm project name, table name, and version (e.g. `soche-india`, `posts`, `v2`)
- [ ] For CRUD: use `/{project}/{table}/{version}/{action}` pattern
- [ ] For custom queries: use `/{project}/{version}/{custom-endpoint}` pattern
- [ ] Actions are: `list`, `show/:id`, `create`, `update/:id`, `delete/:id`
- [ ] `update` uses **POST**, not PUT
- [ ] ID is in **URL path** for show/update/delete, never a query param
- [ ] Always include `X-API-SIGNATURE` header
- [ ] For React/Next.js frontend — store key in env vars, use Vercel proxy for public apps
- [ ] Handle `success: false` responses gracefully in the UI
- [ ] For joins/aggregations → use Custom Queries (different URL pattern)

---

## Reference Links
- Full docs: https://doc.eazemyapi.com
- API endpoints reference: https://doc.eazemyapi.com/docs/api-endpoints
- Field types: https://doc.eazemyapi.com/docs/field-types
- Custom queries: https://doc.eazemyapi.com/docs/custom-queries
- Authentication: https://doc.eazemyapi.com/docs/authentication
