# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

This is the Shopify theme (based on Dawn) for **TapGlow**, a single-product DTC store selling an RGB mechanical-switch keychain ("clicker"). The theme is customized for a premium, Apple/Nothing/Ridge/Teenage-Engineering-style aesthetic on top of stock Dawn.

Business context (relevant when writing customer-facing copy — policies, FAQ, contact info, emails):
- Brand: TapGlow · Website: tapglow.se
- Legal entity: Swedish sole proprietorship (Enskild firma)
- Currency: SEK · Ships worldwide · Fulfillment via a third-party partner (Zopi, dropshipping)
- Payments: Shopify Payments, Visa, Mastercard, PayPal, Google Pay
- Support email: support@tapglow.se · Business address: Brovaktarvägen 15, 141 73 Segeltorp, Sweden

## Commands

Lint/validate the theme (must be run after any Liquid/JSON/CSS change):
```
npx -y @shopify/cli theme check
```
**Baseline: 8 warnings, 0 errors, across 7 files.** These pre-existing warnings are stock-Dawn issues unrelated to this project's customizations and are not expected to be fixed. Any change that introduces a *new* warning or error must be fixed before committing — the count should never exceed the baseline.

There is no local dev server, build step, or test suite in this repo (no `package.json`) — the Shopify CLI invoked via `npx` is only used for theme check.

## Git workflow — read before committing

This repo is connected to a live Shopify store via Shopify's GitHub integration. **Shopify's own bot (`shopify[bot]`) pushes commits directly to `main` whenever someone edits content in the theme customizer or Admin** (e.g. swapping a homepage image), independently of any work happening here. This means `origin/main` can move without warning between the start and end of a task.

Before every commit:
1. `git fetch origin` and diff `origin/main` against local `HEAD` (`git log --oneline origin/main -N` vs `git log --oneline -N`, or `git status --short`).
2. If they've diverged, inspect what the incoming commit(s) touched (`git show --stat <sha>`) before reconciling — so far these have always been auto-generated, single-file `templates/index.json` edits (image selection changes) with no overlap with in-progress theme-code changes.
3. Reconcile with `git rebase origin/main`, not merge — keeps history linear. Re-run theme check after rebasing.
4. Push, then verify with `git rev-parse HEAD` against `git ls-remote origin main` — don't consider a push done until the SHAs match.
5. Never force-push.

`git revert --no-edit HEAD` run twice in a row reverts the *first revert commit* on the second call, not the original second-back commit (since `HEAD` moves after the first revert) — pass explicit commit SHAs (`git revert --no-edit <sha>`) when reverting more than one commit in sequence.

JSON template/config files that Shopify's bot has touched get a `/* auto-generated */` comment block prepended, which breaks strict `JSON.parse`. Theme Check accepts this format natively — that's the authoritative validator, not a naive JSON parse.

## Architecture

**Homepage** (`templates/index.json`) is a linear stack of sections: `hero → benefits → showcase → editorial → faq → final_cta`. All copy lives inline in this file's section `settings`/`blocks` (no dependency on separate content resources) — this is the pattern to follow when adding homepage content: embed it directly in the template JSON rather than pulling from elsewhere.

**Hero** (`sections/hero.liquid` + `assets/section-hero.css`) is a custom section, not stock Dawn — full-bleed background image/video with overlaid text, rebuilt several times to reach the current design:
- Desktop: text is absolutely positioned near the upper-left (`left: 9%; top: 9%`), split into two flex groups — `.hero__content-top` (eyebrow + heading, `display: contents` so it behaves as part of the parent flow) and `.hero__content-bottom` (body text + CTAs, offset further right via `margin-left`). The heading uses sentence-split `<span>` elements instead of literal `<br>` tags (see gotcha below).
- Mobile: `.hero__content` spans the full section height and uses `justify-content: space-between` to pin the top group (label/heading) and bottom group (text/CTAs) to opposite ends, leaving the product visible in the middle — a deliberately different structure from desktop, not a scaled-down version of it.
- The hero image is "zoomed out" via `transform: scale()` on the `<img>`, which shrinks the already-cropped `object-fit: cover` image and reveals the section's background color (`#111315`, matching the photo's dark backdrop) at the edges — there is no way to reveal more of the actual photographed scene this way; that would require a differently-framed source image.
- **Never put `<br>` inside a JSON `heading`/text setting value** — Shopify's theme sync rejects invalid HTML embedded in JSON string values. Line breaks are done via the sentence-split span technique above, styled with `display: block`.
- Watch for Dawn's `.page-width` utility class (`max-width` + `margin: 0 auto`) leaking onto custom absolutely-positioned wrappers — it silently re-centers/narrows the containing block that percentage-based `left`/`right` values are computed against, and was the root cause of a hard-to-find "hero won't stop centering" bug.

**Header** (`sections/header.liquid` + `header-group.json`): stripped down to logo + cart icon only — no nav menu, search, account icon, or Buy Now button. Logo size is controlled by `.header__heading-link .h2 { font-size }` in the section's inline `<style>` block.

**Footer** (`sections/footer.liquid` + `footer-group.json`): `show_policy: true` makes Dawn auto-render a footer row of links for any of the four Shopify shop policies that has body content — no theme change needed once policy text exists in Admin. A `link_list` block ("Help Center") points at a **dedicated** navigation menu (handle `help-center`), deliberately separate from Shopify's default "Footer menu" to avoid inheriting unwanted default entries (e.g. a "Search" link).

**Product page**: stock Dawn `main-product.liquid`/`templates/product.json`, styled via `body.template-product` scoped rules in `assets/component-premium-polish.css` (spacing rhythm, Addто Cart dominance over Shop Pay via `--shopify-payment-button-color-scheme`, a fix for Dawn's `.icon-with-text--horizontal` having no mobile breakpoint).

**Contact / FAQ pages** (`templates/page.contact.json`, `templates/page.faq.json`): built as fully theme-embedded content using stock Dawn sections (`rich-text`, `collapsible-content`) rather than relying on the Shopify Page resource's own body field — same "embed content directly in the template JSON" pattern as the homepage. This means the *content* is fully version-controlled, but a Page record with a matching handle (`contact`, `faq`) still has to exist in Shopify Admin with its **Theme template** set to `page.contact` / `page.faq` respectively — if a page created in Admin shows empty, that template assignment is almost always the reason.

## Key constraint: store content vs. theme files

Shopify's four legal policies (Privacy, Terms, Shipping, Refund), Page body content, and navigation menu items are **store data**, not theme files — they live in Shopify's Admin/database and cannot be written via this git repo. This session has no Shopify Admin API credentials and creating/using a custom app was explicitly declined by the project owner — don't set one up or ask for API tokens/secrets unless directly instructed otherwise.

Draft copy for anything that has to go through Admin manually lives in `store-content/` (policies, page copy, and `README-store-content.md` with exact click-by-click paste/setup instructions). When asked to update policy or store-content copy, update the files in that folder rather than assuming it can be pushed live.

## Working style on this project

- Changes are requested and applied in small, iterative rounds — match the scope of the request exactly; don't redesign adjacent sections or "improve" things that weren't asked about.
- Prefer reusing existing stock Dawn section types (`rich-text`, `image-with-text`, `multicolumn`, `collapsible-content`) over inventing new custom sections.
- No lorem ipsum, no unnecessary bracket placeholders — if a real value is known (see business context above), use it.
- There is no browser/visual access in this environment — positioning/spacing changes are reasoned about from CSS/DOM mechanics, not visually confirmed. Say so explicitly rather than claiming a visual result, and give the user a concrete lever (an exact value/percentage) to request further adjustment.
