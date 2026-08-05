# Store Content — Manual Setup Guide

This folder contains the final, ready-to-use copy for TapGlow's legal policies and info pages. None of this can be pushed to your live store through git — Shopify stores policies, pages, and navigation menus as store data (not theme files), so they have to be entered through the Shopify Admin UI. This guide gives you the exact clicks.

No API tokens or credentials are needed for any of this — it's all manual copy/paste in the Admin.

## Files in this folder

| File | What it is | What you do with it |
|---|---|---|
| `privacy-policy.md` | Privacy Policy | Copy the body text into Settings → Policies |
| `terms-of-service.md` | Terms & Conditions | Copy the body text into Settings → Policies |
| `shipping-policy.md` | Shipping Policy | Copy the body text into Settings → Policies (fill in one placeholder first) |
| `return-refund-policy.md` | Return & Refund Policy | Copy the body text into Settings → Policies |
| `contact-page.md` | Contact page copy | Already live in the theme — reference only |
| `faq-page.md` | FAQ page copy | Already live in the theme — reference only |

## Step 1 — Paste the four legal policies

1. Go to **Settings → Policies** in Shopify Admin.
2. For each policy field, click into the box, select all existing placeholder text, delete it, and paste in the body from the matching file above (everything below the `---` divider, skip the "Paste into..." instruction line at the top of the file).
3. Do this for all four: **Privacy policy**, **Terms of service**, **Shipping policy**, **Refund policy**.
4. Before pasting the Shipping policy, replace `[Estimated Shipping Times by Region]` in that text with your real regional delivery estimates (I don't have your carrier data, so I left this one placeholder deliberately).
5. Click **Save** at the top of the page.

Once saved, these will automatically appear as links in your site footer — the theme's footer already has "Show policy links" turned on, so no further theme changes are needed for this part.

## Step 2 — Create the Contact and FAQ pages

The content for these two pages is already built into the theme (`templates/page.contact.json` and `templates/page.faq.json`), but Shopify still needs an actual **Page** record in Admin pointing at each template.

**Contact page:**
1. Go to **Online Store → Pages → Add page**.
2. Title: `Contact`. Leave the content box empty (the theme template supplies the content, not this box).
3. In the right sidebar, under **Theme template**, choose `page.contact`.
4. Save.
5. Check the page's handle (shown under the title field) is `contact` — if Shopify picked something else, edit it to `contact` so `/pages/contact` works.

**FAQ page:**
1. Go to **Online Store → Pages → Add page**.
2. Title: `FAQ`. Leave the content box empty.
3. In the right sidebar, under **Theme template**, choose `page.faq`.
4. Save.
5. Confirm the handle is `faq`.

If a "Contact Us" page already exists from Shopify's default setup, edit that existing page instead of creating a new one — just change its **Theme template** to `page.contact` (or `page.faq`) rather than creating a duplicate.

## Step 3 — Build the "Help Center" menu

The footer has a "Help Center" quick-links block wired up (added via the theme), pointing at a **dedicated** navigation menu with the handle `help-center` — deliberately separate from Shopify's general-purpose "Footer menu" (which often carries default clutter like a "Search" link) so this one stays clean with exactly the 4 links below, in this order:

1. Go to **Online Store → Navigation**.
2. Click **Add menu**.
   - Title: `Help Center` — Shopify will auto-assign the handle `help-center`. If it picks a different handle, open the menu and check/edit the handle field so it's exactly `help-center` (this must match what the theme block looks for).
3. Add exactly these 4 menu items, in this order:
   1. Name: `FAQ` → Link: select the FAQ page (created in Step 2).
   2. Name: `Contact Us` → Link: select the Contact page (created in Step 2).
   3. Name: `Shipping Policy` → Link: under "Policies" in the link picker, select Shipping policy.
   4. Name: `Return & Refund Policy` → Link: under "Policies", select Refund policy.
4. Do **not** add Privacy Policy or Terms of Service here — those already appear automatically in the footer's separate policy-links row (from Step 1), so adding them here would just duplicate them.
5. Save.

Once saved, these 4 links will appear under the "Help Center" heading in your footer. If you'd previously added Contact/FAQ links to the default "Footer menu" per an earlier version of these instructions, you can leave that menu alone (it's no longer referenced by this block) or clean it up — your call.

## Where things already stand

- ✅ Theme templates for Contact and FAQ — done, in git, no action needed beyond Step 2 above.
- ✅ Footer wired to show policy links and a Help Center quick-links block — done, in git.
- ⬜ Pasting the 4 policy texts into Admin — your action (Step 1).
- ⬜ Creating/assigning the Contact and FAQ Page records — your action (Step 2).
- ⬜ Building the dedicated Help Center navigation menu — your action (Step 3).
