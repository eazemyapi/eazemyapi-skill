---
name: eazemyapi
description: Use this skill whenever the user is building, designing, or extending EazeMyAPI(Backend Work Eliminated) an AI-powered backend-as-a-service (BaaS) platform. Trigger this skill for any of the following: building or designing EazeMyAPI dashboard UI or module pages, writing frontend integration code that calls EazeMyAPI APIs, designing database schemas and tables for an EazeMyAPI project, writing custom SQL queries to expose as endpoints, scaffolding or connecting SaaS products that use EazeMyAPI as their backend, or any mention of "EazeMyAPI", "eaze", "my BaaS", or "my backend". Always use this skill even if the request feels like a general UI, API, or architecture task — if EazeMyAPI is the context, this skill applies.
---

# EazeMyAPI Skill

EazeMyAPI is an **AI-powered backend builder** that lets developers generate databases and REST APIs instantly — no backend server code needed. Users describe their app (or define tables manually), and EazeMyAPI generates the full REST API automatically.

> Full documentation: https://doc.eazemyapi.com

This skill covers:
1. **Frontend / UI** — EazeMyAPI dashboard pages, module UIs, components
2. **Backend integration** — API calls, auth, data flows connecting products to EazeMyAPI
3. **Schema design** — tables, field types, relationships, custom queries

---

## How EazeMyAPI Works

**Workflow:**
```
Idea → Create Project → Define Tables → Generate APIs → Connect Frontend
```

1. Create a **Project** (your app's namespace)
2. Define **Database Tables** (with fields and types)
3. EazeMyAPI **auto-generates REST endpoints** for all CRUD operations
4. Add **Custom Queries** for joins, filters, aggregations
5. Integrate via the **API** using `X-API-SIGNATURE` header

---

## API Reference

### Base URL Structure
```
https://api.eazemyapi.com/{project-name}/{version}/{endpoint}
```

| Component | Description |
|---|---|
| `api.eazemyapi.com` | Base API server |
| `{project-name}` | Your project identifier (e.g. `demo-project`) |
| `{version}` | API version (e.g. `v2`) |
| `{endpoint}` | Auto-generated or custom endpoint name |

**Example:**
```
https://api.eazemyapi.com/demo-project/v2/get-single-user
```

---

### Authentication

Every request must include the API signature header:
```
X-API-SIGNATURE: <your-secret-key>
```

Find your key at: **Project → Settings → API Signature Key**

**Security rules:**
- Never expose the key in frontend/public code
- Store in environment variables
- Rotate if compromised
- If user-level access control is needed, implement auth on top

---

### Auto-Generated Endpoints

For every table, EazeMyAPI generates these endpoints automatically:

| Operation | Endpoint pattern | Method |
|---|---|---|
| Get all records | `get-{table}s` | GET |
| Get one record | `get-single-{table}` | GET + `?id=` |
| Create record | `create-{table}` | POST |
| Update record | `update-{table}` | PUT |
| Delete record | `delete-{table}` | DELETE + `?id=` |

**Examples for a `user` table:**
```
GET    https://api.eazemyapi.com/my-project/v2/get-users
GET    https://api.eazemyapi.com/my-project/v2/get-single-user?id=1
POST   https://api.eazemyapi.com/my-project/v2/create-user
PUT    https://api.eazemyapi.com/my-project/v2/update-user
DELETE https://api.eazemyapi.com/my-project/v2/delete-user?id=1
```

---

### Response Format

All responses follow this consistent JSON shape:

```json
{
  "success": true,
  "message": "Data found.",
  "data": []
}
```

| Field | Description |
|---|---|
| `success` | `true` or `false` |
| `message` | Human-readable result description |
| `data` | Object (single) or Array (multiple) |

**Single record:**
```json
{
  "success": true,
  "message": "Data found.",
  "data": { "id": 1, "name": "Romit", "email": "rm@eazemyapi.com" }
}
```

**Error:**
```json
{
  "success": false,
  "message": "Data not found",
  "data": []
}
```

**Always check `success` before processing `data`.**

---

### Custom Queries

For joins, filters, aggregations, or complex logic — write SQL and EazeMyAPI exposes it as an endpoint.

**Example query:**
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

**Generated endpoint:**
```
https://api.eazemyapi.com/my-project/v2/posts-by-category
```

**With parameters (`:param` syntax):**
```sql
SELECT * FROM posts WHERE category_id = :category_id;
```
```
GET /posts-by-category?category_id=abc123
```

**Create via dashboard:** Project → Custom Queries → Create Query → Write SQL → Save → endpoint auto-generated.

---

## Field Types

| Type | Use for |
|---|---|
| `TEXT` | Short text: names, titles, usernames |
| `NUMBER` | Integers: age, quantity, IDs |
| `PARAGRAPH` | Long text: descriptions, bios, blog content |
| `DECIMAL` | Prices, ratings, percentages |
| `DATE` | Calendar dates: birth date, event date |
| `DATETIME` | Timestamps: created_at, updated_at |
| `PASSWORD` | Stored encrypted |
| `EMAIL` | Valid email addresses |
| `PHONE` | Phone numbers |
| `BOOLEAN` | true/false: is_active, is_verified |

**Example users table:**
```
id          → NUMBER
name        → TEXT
email       → EMAIL
phone       → PHONE
bio         → PARAGRAPH
is_active   → BOOLEAN
created_at  → DATETIME
```

---

## Frontend Integration Patterns

### JavaScript / React (fetch)
```javascript
const res = await fetch(
  "https://api.eazemyapi.com/{project}/{version}/{endpoint}",
  {
    method: "GET",
    headers: {
      "Content-Type": "application/json",
      "X-API-SIGNATURE": process.env.EAZEMYAPI_SECRET_KEY
    }
  }
);
const { success, data, message } = await res.json();
if (!success) throw new Error(message);
```

### Create a record (POST)
```javascript
const res = await fetch(
  "https://api.eazemyapi.com/my-project/v2/create-user",
  {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
      "X-API-SIGNATURE": process.env.EAZEMYAPI_SECRET_KEY
    },
    body: JSON.stringify({ name: "Romit", email: "rm@eazemyapi.com" })
  }
);
```

### React custom hook pattern
```javascript
function useEazeAPI(endpoint) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    fetch(`https://api.eazemyapi.com/${PROJECT}/${VERSION}/${endpoint}`, {
      headers: { "X-API-SIGNATURE": process.env.REACT_APP_EAZE_KEY }
    })
      .then(r => r.json())
      .then(({ success, data, message }) => {
        if (!success) throw new Error(message);
        setData(data);
      })
      .catch(err => setError(err.message))
      .finally(() => setLoading(false));
  }, [endpoint]);

  return { data, loading, error };
}
```

### Vercel serverless function (safe key handling)
```javascript
// /api/get-users.js
export default async function handler(req, res) {
  const response = await fetch(
    "https://api.eazemyapi.com/my-project/v2/get-users",
    {
      headers: { "X-API-SIGNATURE": process.env.EAZEMYAPI_SECRET_KEY }
    }
  );
  const data = await response.json();
  res.status(200).json(data);
}
```
Use serverless functions to proxy EazeMyAPI calls when the key must stay server-side.

---

## Design System (EazeMyAPI Dashboard UI)

Apply when building EazeMyAPI's own dashboard. For external SaaS products powered by EazeMyAPI, use that product's own design system or follow the user's direction.

> Full tokens and component CSS: `references/design-tokens.md`

**Core palette:**
- Sidebar: `#0f1729` (dark navy)
- Background: `#f8f9fc` (light grey)
- Cards: `#ffffff` with `border: 1px solid #e5e7eb`
- Primary accent: `#6366f1` (indigo/purple)
- Success: `#10b981` | Warning: `#f59e0b` | Error: `#ef4444`

**Module page layout pattern:**
1. Page header — title + description + primary action button
2. Stats row (optional) — 3–4 small stat cards
3. Main table/list card — with search/filter
4. Empty state — shown when no data exists

---

## Beta UX Principles

EazeMyAPI is in beta. Keep UI simple and approachable:

1. **Plain language** — "Run every day at 9am" not "0 9 * * *"
2. **Progressive disclosure** — Hide advanced options behind toggles
3. **Sensible defaults** — Pre-fill reasonable values
4. **Inline feedback** — Toast notifications for all actions
5. **Empty states** — Always include with a helpful CTA

---

## Code Output Guidelines

- **React**: Functional components + hooks. Tailwind preferred. Lucide icons. No `<form>` tags.
- **HTML**: Single-file with inline `<style>` + CSS variables.
- **Node.js / Vercel**: Default for server-side API proxying.
- **Other stacks**: Match the user's stack (Python, Flutter, etc.).
- Always check `success` field — don't just rely on HTTP status.
- Never hardcode `X-API-SIGNATURE` — always use env vars.

---

## Reference Files

- `references/modules.md` — EazeMyAPI dashboard module specs
- `references/design-tokens.md` — Full CSS variables and component snippets

---

## Quick-Start Checklist

- [ ] Identify task: UI / API integration / Schema design
- [ ] For API calls: use `/{project}/{version}/{endpoint}` URL structure
- [ ] Always include `X-API-SIGNATURE` header from env var
- [ ] Always check `success` field before using `data`
- [ ] For complex queries: use Custom Queries (SQL → auto endpoint)
- [ ] For UI: apply design system only for EazeMyAPI dashboard; otherwise follow product style
- [ ] Include loading states, error handling, and empty states