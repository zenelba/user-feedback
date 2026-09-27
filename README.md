# @zenel/user-feedback

Shared **Report error / idea** stack for Vercel apps: header icon button with tooltip, page screenshot, pin-and-describe modal, client submit, and Neon + Blob + Resend API handlers.

Users click the screenshot to pin where the issue is. The pin is baked into the uploaded PNG and stored as `pin_x` / `pin_y`.

## Install

```json
"@zenel/user-feedback": "github:zenelba/user-feedback#v0.2.2"
```

`dist/` is committed, so git installs need no build step (works on Vercel).

## Client

Drop the launcher into the header. It renders a round bug icon; the label shows as a tooltip on hover and keyboard focus and is the accessible name.

```tsx
import { FeedbackLauncher } from "@zenel/user-feedback/client";
import "@zenel/user-feedback/styles.css";

<FeedbackLauncher
  toolId="map"
  toolLabel="Map"
  label={t("reportError")} // optional, default "Report error / idea"
  captureSelector=".app"   // optional, element to screenshot
  getJournal={() => journalSnapshot()}
  getJournalMarkdown={() => journalMarkdown()}
  idbName="myapp-feedback"
  onOpen={() => logEvent("feedback_open")}
/>
```

Styling uses host CSS variables when present: `--surface`, `--border`, `--text`, `--radius`.

Lower-level pieces are exported too: `FeedbackButton` (icon only) and `FeedbackModal` (modal only).

## Server (Vercel)

```ts
import { createFeedbackSaveHandler } from "@zenel/user-feedback/server";

export default createFeedbackSaveHandler({
  appName: "MyApp",
  projectId: "myapp",
  authorize: (req) => hasValidAccessCookie(req.headers?.cookie),
  ensureEnv: ensureProjectEnv,
});
```

Same pattern for `createFeedbackListHandler` and `createFeedbackResolveHandler`.

Env (on each host app — never commit these):

| Variable | Required | Purpose |
| --- | --- | --- |
| `POSTGRES_URL` | for DB | shared Neon store (same as Bluring); rows separated by `projectId` |
| `BLOB_READ_WRITE_TOKEN` | for screenshots | Vercel Blob upload |
| `RESEND_API_KEY` | for email | optional; without it, submit still saves to DB/Blob |
| `FEEDBACK_TO_EMAIL` | for email | recipient; **required** if you want Resend mail (no package default) |
| `FEEDBACK_FROM_EMAIL` | optional | defaults to `AppName Feedback <onboarding@resend.dev>` |

Screenshot blobs are stored with public URLs, e.g.  
`https://<store>.public.blob.vercel-storage.com/feedback/2026-09-27T18-01-00_map_error.png`  
Anyone who knows that URL can open the PNG.

## Updating apps

1. Change the package, bump `version` in `package.json`, run `npm run build`, commit (including `dist/`), tag `vX.Y.Z`, push the tag.
2. In each app: change the tag in its `package.json` dependency, `npm install`, commit the lockfile, deploy.
