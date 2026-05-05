# mattiheino-newsletter

Static sign-up page for `newsletter.mattiheino.com`. Hosted on Cloudflare Pages, DNS at Cloudflare, form initially backed by Formspree, eventually by self-hosted Listmonk on Hetzner.

## Files

- `index.html` – sign-up page with three-checkbox form (blog posts, complexity course, civil-preparedness cohort)
- `privacy.html` – privacy notice published at `/privacy.html`
- No build step; everything is in those two files plus README

## Deployment

1. Push to GitHub: `heinonmatti/mattiheino-newsletter` (public).
2. Connect to Cloudflare Pages → Pages → Create project → Connect to Git → select this repo. Build command: empty. Output directory: `/` (root).
3. Cloudflare DNS for `mattiheino.com` → add CNAME: `newsletter` → `<project>.pages.dev`, proxied (orange cloud).
4. Cloudflare Pages → Custom domains → add `newsletter.mattiheino.com`. TLS provisions automatically.

## Form action – switch from Formspree to Listmonk

The form currently posts to a Formspree placeholder. Two changes will happen over the build:

### Phase 1: Formspree backup (Tuesday/Wednesday 5–6 May 2026)

In `index.html`, replace `YOUR_FORMSPREE_ID` with the real ID from a new Formspree form ("mattiheino.com newsletter signup", notification email `matti@mattiheino.com`). Submissions land in Formspree's dashboard and email Matti per submission.

### Phase 2: Listmonk live (Wednesday/Thursday 6–7 May 2026)

Once Listmonk is online at `lists.mattiheino.com`:

1. Three lists exist in Listmonk: `newsletter`, `complexity-course`, `civil-preparedness-cohort`. Capture their UUIDs.
2. In `index.html`, change the `<form action="...">` to `https://lists.mattiheino.com/subscription/form`.
3. Change each `<input type="checkbox" name="topic" value="…">` to `<input type="checkbox" name="l" value="<list-UUID>">` with the corresponding UUID.
4. Listmonk requires a hidden `nonce` field; add `<input type="hidden" name="nonce" value="">` inside the form.
5. Push, Cloudflare Pages auto-deploys.

Form HTML structure stays the same. Only the action URL and field names change.

## Privacy notice URL

`https://newsletter.mattiheino.com/privacy.html`. Linked from the sign-up form's footer and from the form-note line beneath the submit button.

## Local preview

Open `index.html` in a browser, or run `python -m http.server` from this directory and visit `http://localhost:8000/`.
