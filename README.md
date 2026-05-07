# mattiheino-newsletter

Static sign-up page for `news.mattiheino.com`. Hosted on Cloudflare Pages with DNS at Cloudflare. The form posts to a Cloudflare Worker at `submit.mattiheino.com` that bridges to a self-hosted Listmonk instance at `lists.mattiheino.com` (running on a Hetzner VPS).

Brand follows the ajatuspää käytäytymisarkkitehtuuri identity from mattiheino.com (sky-blue hero band with the two-heads logo, deep red as the title and CTA accent, warm cream page).

## Files

- `index.html` – sign-up page with a three-checkbox form (blog posts, complexity course, civil-preparedness cohort) plus a conditional textarea ("If you took part in such a course, what would you like to learn?") that appears under each course when its checkbox is ticked.
- `thanks.html` – Worker redirects subscribers here after a successful submission ("Check your inbox").
- `error.html` – Worker redirects here when the admin-API call fails.
- `privacy.html` – privacy notice published at `/privacy.html`.
- `assets/site.css` – shared baseline styles (brand palette, hero band, container, layout). Each HTML page links this and supplies its page-specific rules inline.
- `assets/ajatuspaa.jpg` – the logo used in the hero band on every page.
- No build step; static HTML + inline page-specific CSS + a shared stylesheet + ~25 lines of vanilla JavaScript at the bottom of `index.html`.

## Sign-up flow

```
+-----------------------------+        +-----------------------+
| news.mattiheino.com         |        | lists.mattiheino.com  |
| (Cloudflare Pages: this)    |        | (Listmonk on Hetzner) |
|                             |        |                       |
|   form action -->           |        |                       |
+-----------------------------+        +-----------------------+
              |                                  ^
              | POST                             | POST /api/subscribers
              v                                  | (Basic auth, server-only)
+-----------------------------+                  |
| submit.mattiheino.com       | ------------------
| (Cloudflare Worker)         |
+-----------------------------+
```

The Worker:

1. Validates the `Origin`/`Referer` against `news.mattiheino.com`.
2. Reads `email`, `name`, three `attribs[wants_*]` checkbox flags, and two `attribs[*_interest_note]` textarea fields from the form post.
3. Builds a subscriber payload: master list always; topic lists added per ticked checkbox; attribs include the topic flags and any interest notes.
4. Translates list UUIDs to numeric IDs (one cached `GET /api/lists` per Worker isolate) and POSTs the payload to Listmonk's admin API using a stored API token.
5. Master list is double-opt-in (Listmonk sends a confirm email). Topic lists are single-opt-in (added immediately). Net effect: one confirm email per sign-up regardless of how many topics were ticked.
6. Returns a 303 redirect to `thanks.html` on success or `error.html` on failure.

Why a Worker rather than posting straight to Listmonk's public form: Listmonk v6.1.0's `/subscription/form` endpoint silently drops `attribs[…]` fields, and its admin API requires authentication that we don't want exposed to the browser.

Worker source lives in the personal-assistant repo at `projects/Personal/Marketing/expression-of-interest/worker/`.

## Lists in Listmonk

| List name                              | Type | Opt-in | Used for                          |
|----------------------------------------|------|--------|------------------------------------|
| `Matti's notes` (master)               | Public | Double | Confirm-email gate, baseline list |
| `Matti's notes – news`                 | Public | Single | Blog-update broadcasts             |
| `Matti's notes – complexity behchange course` | Public | Single | Complexity-course audience    |
| `Matti's notes – preparedness cohort`  | Public | Single | Civil-preparedness audience        |

Per-topic campaigns target the relevant single-opt-in list directly. Subscribers can edit their topic memberships via the "Manage subscription" link Listmonk includes in every email it sends.

## Form fields

The form sends:

- `name` (first name, optional – used in the opt-in email greeting)
- `email` (required)
- `attribs[wants_blog]=true` when the blog checkbox is ticked
- `attribs[wants_complexity_course]=true` when the complexity-course checkbox is ticked
- `attribs[wants_civil_preparedness_cohort]=true` when the civil-preparedness checkbox is ticked
- `attribs[complexity_course_interest_note]=...` free-text, only when the complexity-course checkbox is ticked
- `attribs[civil_preparedness_interest_note]=...` free-text, only when the civil-preparedness checkbox is ticked

The Worker maps each `wants_*` flag to a Listmonk list UUID (hardcoded in the Worker source) and adds membership in addition to the master list.

## Deployment

1. Push to GitHub: `heinonmatti/mattiheino-newsletter` (public).
2. Cloudflare Pages → Create project → Connect to Git → select this repo. Build command: empty. Output directory: `/` (root).
3. Cloudflare DNS for `mattiheino.com` → add `CNAME: news → <project>.pages.dev`, proxied (orange cloud).
4. Cloudflare Pages → Custom domains → add `news.mattiheino.com`. TLS provisions automatically.

The Worker (`submit.mattiheino.com`) is deployed separately from this repo; see `personal-assistant/projects/Personal/Marketing/expression-of-interest/worker/README.md`.

## Local preview

Open `index.html` in a browser, or run `python -m http.server` from this directory and visit `http://localhost:8000/`. Form submissions from a local preview will be rejected by the Worker's origin check; that's expected.
