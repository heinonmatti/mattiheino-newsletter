# mattiheino-newsletter

Static sign-up page for `news.mattiheino.com`. Hosted on Cloudflare Pages, DNS at Cloudflare, form posts to a self-hosted Listmonk instance at `lists.mattiheino.com` running on the Hetzner VPS.

Brand follows the ajatuspää käytäytymisarkkitehtuuri identity from mattiheino.com (sky-blue hero band with the two-heads logo, deep red as the title and CTA accent, warm cream page).

## Files

- `index.html` – sign-up page with three-checkbox form (blog posts, complexity course, civil-preparedness cohort) plus a conditional textarea ("If you took part in such a course, what would you like to learn?") that appears when either course checkbox is ticked.
- `privacy.html` – privacy notice published at `/privacy.html`.
- `assets/ajatuspaa.jpg` – the logo used in the hero band on both pages.
- No build step; static HTML + inline CSS + ~25 lines of vanilla JavaScript at the bottom of `index.html`.

## Form architecture

**One Listmonk list with topic preferences as subscriber attributes**, rather than three separate lists. Form fields:

- `name` (first name, optional)
- `email` (required)
- `attribs[wants_blog]=true` (when blog checkbox ticked)
- `attribs[wants_complexity_course]=true` (when complexity-course checkbox ticked)
- `attribs[wants_civil_preparedness_cohort]=true` (when civil-preparedness checkbox ticked)
- `attribs[complexity_course_interest_note]=...` (free-text answer for the complexity course; present only when that course is ticked)
- `attribs[civil_preparedness_interest_note]=...` (free-text answer for the civil-preparedness cohort; present only when that course is ticked)
- `l=<list-UUID>` (hidden, the single list's UUID)
- `nonce` (hidden, Listmonk's CSRF token; left empty for public form)

This collapses the email flow to one confirmation + one merged conditional welcome. Per-topic broadcasts use Listmonk segment filtering on `attribs->>'wants_<topic>'`.

## Deployment

1. Push to GitHub: `heinonmatti/mattiheino-newsletter` (public).
2. Connect to Cloudflare Pages → Pages → Create project → Connect to Git → select this repo. Build command: empty. Output directory: `/` (root).
3. Cloudflare DNS for `mattiheino.com` → add CNAME: `news` → `<project>.pages.dev`, proxied (orange cloud).
4. Cloudflare Pages → Custom domains → add `news.mattiheino.com`. TLS provisions automatically.

## Pre-deploy: substitute the Listmonk list UUID

Before pushing the repo for the first deployment, replace `REPLACE_WITH_LIST_UUID` in `index.html` with the UUID of the single Listmonk list (created during Listmonk install, Phase 2 of the build-plan in `personal-assistant/projects/Personal/Marketing/expression-of-interest/build-plan.md`).

The form submits to `https://lists.mattiheino.com/subscription/form` regardless; only the hidden `l` value needs to be filled in.

## Local preview

Open `index.html` in a browser, or run `python -m http.server` from this directory and visit `http://localhost:8000/`.
