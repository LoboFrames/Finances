# Safe-to-Spend — private deployment

A private, authenticated URL for the budget app. Free at this scale.

Deployed as a **Cloudflare Worker with static assets**, protected by **Cloudflare Access**.

## Why Workers and not Pages

Cloudflare Pages *can* use Access, but by default it only protects **preview**
deployments — the production `*.pages.dev` URL stays open unless you apply a
documented workaround. A Worker is cleaner: an Access policy attached to the
Worker covers every hostname it serves (workers.dev, previews, and any custom
domain you add later) in one step, with no domain purchase required.

## Cost

| Piece | Free tier | This app's usage |
|---|---|---|
| Workers | 100,000 requests/day | a few dozen |
| Zero Trust / Access | 50 users | 1 |
| Domain | not required | uses `*.workers.dev` |

Total: **$0**. Access blocks user 51 rather than silently billing you, so there
is no path to a surprise charge.

## One-time setup

```bash
cd deploy
npx wrangler login      # opens a browser, authorises your Cloudflare account
npx wrangler deploy
```

Wrangler prints the live URL: `https://safe-to-spend.<your-subdomain>.workers.dev`

**At this point the URL is public.** Do the Access step before putting real data
on it.

## Lock it down with Access

1. Cloudflare dashboard → **Zero Trust**. First visit asks you to pick a team
   name and a plan — choose **Free**. A payment method may be requested for
   verification; the free plan does not charge.
2. **Workers & Pages** → `safe-to-spend` → **Settings** → enable Access
   protection. Choose the option covering **all hostnames**, not previews only.
3. Set the policy to allow **your email address only**. Pick *One-time PIN* as
   the login method — it emails you a code, no extra account needed. Google
   login also works.
4. Open the URL in a private window to confirm you get the login screen.

## Redeploying after changes

```bash
cp ../safe-to-spend.html public/index.html
npx wrangler deploy
```

## GitHub

A **private** repo for this directory is fine and worth having for version
history.

**Do not enable GitHub Pages on it.** On Free and Pro plans a Pages site is
publicly accessible even when published from a private repository —
access-controlled Pages is GitHub Enterprise Cloud only. Cloudflare Access is
what makes this private; GitHub Pages would route around it.

## What's actually in this file

`public/index.html` is not just code. Embedded in it:

- 554 transactions with real merchant names and amounts
- Family names from Zelle memos
- Card last-4s (`...0952`, `...9537`)
- `PAYROLL_YTD_GROSS` / `PAYROLL_YTD_NET`
- A July timesheet with daily clock-in/out times
- 4 receipt photos as base64 JPEGs

Treat the repo as sensitive. Keep it private, and don't paste the file into
anything public.

## Storage behaviour

The app persists to IndexedDB (falling back to localStorage, then to memory
with a warning banner). Served over HTTPS it will always get IndexedDB.

Storage is **per browser, per device**. Your Mac and your phone each keep their
own copy — they do not sync. Data baked into the file (the `ensure*Import()`
functions) seeds both identically; anything you categorise or edit afterwards
stays on that device.

To move state between devices, from the browser console:

```js
await exportBudgetData()          // downloads a JSON snapshot
await importBudgetData(jsonText)  // restores it, then reloads
```

Real cross-device sync needs a backend. That's the same piece of work as live
bank data via SimpleFIN or Teller — worth doing once, together.
