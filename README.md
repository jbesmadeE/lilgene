# Job Helper

A personal Chrome extension (Manifest V3) for **OnlineJobs.ph**, plus a
**LinkedIn post composer** mode. Open any job posting and a panel slides in on
the right with the scraped **JOB OVERVIEW** and a proposal workspace. Open
LinkedIn's share composer and the same panel drafts a short "open to
opportunities" post from `portfolio.MD`.

Built from `LAYOUT.MD`: right-side panel, scraped job overview, proposal
content, and **Draft letter / Generate letter / Copy / Apply now** actions.

## Load it in Chrome

1. Open `chrome://extensions` in a new tab.
2. Turn on **Developer mode** (top-right toggle).
3. Click **Load unpacked** and select this folder: `D:\AI job helper`.
4. Optional: pin Job Helper from the puzzle-piece menu.

Then visit any job page, for example:

`https://www.onlinejobs.ph/jobseekers/job/full-time-bookkeeper-for-a-us-accounting-firm-1762967`

The panel opens automatically on the right. The toolbar icon toggles it, and
`Esc` closes it (a floating **Job Helper** pill reopens it).

For LinkedIn, open `https://www.linkedin.com/sharing/compose/`: the panel opens
and drafts a post automatically. See [LinkedIn posts](#linkedin-posts).

## What works now

- **Auto-open** on URLs containing `https://www.onlinejobs.ph/jobseekers/job`
  **and** `https://www.onlinejobs.ph/apply`.
- **LinkedIn post mode** — on `https://www.linkedin.com/sharing/compose*` the
  panel opens and writes a short "open to opportunities" post from
  `portfolio.MD`, using the LinkedIn rules at the top of `SKILL.MD`. See
  [LinkedIn posts](#linkedin-posts).
- **Auto-scrape** of the `JOB OVERVIEW` block, job title, job ID, work type,
  salary, hours, date updated, and skill tags.
- **Survives the apply step**: the job is snapshotted to `chrome.storage.local`,
  so on `/apply` the same job data is still sent to the AI. It also matches a job
  ID in the apply URL (`?jobid=`, `/apply/123`) against saved snapshots.
- **Drafted letter** card with a live character count and local autosave. The
  scraped job overview is no longer displayed in a separate box.
- **AI letter drafting** — **Draft letter** (next to **Generate letter**) calls
  the AI route and writes the drafted letter into the **Drafted letter** area,
  following every rule in `SKILL.MD`. See [AI letter drafting](#ai-letter-drafting).
- **Contact Info** card built from the `# Contact Information` section of
  `portfolio.MD`, shown as one plain-text block with no individual Copy buttons.
- **Copy letter** copies the drafted letter to the clipboard.
- **Apply now** scrolls to the page's apply control and highlights it; if the
  page only shows the logged-out prompt, it scrolls there instead.
- Light/dark theme, keyboard support, reduced-motion support.

## What is stubbed / manual

- **Real apply automation** — `ui.apply` scrolls/highlights instead of clicking
  through the application form.
- **Auto-generation** — a LinkedIn post is drafted automatically when the
  composer opens. A job letter is still drafted on demand (button), not as soon
  as a job page opens.
- **Posting to LinkedIn** — the post is only copied to the clipboard; it is
  never typed into LinkedIn's editor or published.

## AI letter drafting

The extension does not include a model. You bring your own free API key; the API
call happens in the background service worker, so the page's CSP cannot block
it and the key never touches the job page.

### One-time setup

1. Click the **gear** icon in the panel header (or
   `chrome://extensions` → Job Helper → **Extension options**).
2. Pick a provider and paste a key. Models are editable free text, so you can
   type a newer model id without changing code.
3. Click **Test connection**.
4. The extension always reads `portfolio.MD` from the installed extension. There
   is no portfolio form to fill or re-enter; Draft letter sends the file as-is
   together with `SKILL.MD` and the scraped job.
5. Back on a job page, click **Draft letter**. The drafted letter appears in the
   **Drafted letter** card; review it, then use **Copy letter**.
   Nothing is submitted automatically.

The **Contact Info** card is read from the bottom of `portfolio.MD` and shows
Portfolio, Email, WhatsApp, Facebook, and Google Calendar as one plain-text
block. There are no individual Copy buttons on the contact fields.

### Providers

Six free-tier providers are registered in `js/ai-providers.js`:

| Provider | Default model | Free key from |
| --- | --- | --- |
| Google Gemini | `gemini-2.5-flash` | https://aistudio.google.com/app/apikey |
| Groq | `openai/gpt-oss-120b` | https://console.groq.com/keys |
| OpenRouter | `google/gemma-4-31b-it:free` | https://openrouter.ai/keys |
| Cerebras | `llama-3.3-70b` | https://cloud.cerebras.ai/ |
| Mistral AI | `mistral-small-latest` | https://console.mistral.ai/api-keys |
| GitHub Models | `openai/gpt-4o-mini` | https://github.com/settings/tokens |

Enable **Fallback** in Settings and add keys to more than one provider: if the
primary call fails (bad key, rate limit, missing model), the worker tries the
others in order. Each failure is reported in the toast.

Free model ids are retired regularly. If a saved model is rejected as unknown or
unavailable, the worker automatically retries once with that provider's current
default model, then reports the result.

### Adding another provider

1. Add an entry to `globalThis.JOB_HELPER_PROVIDERS` in `js/ai-providers.js`:
   - `type: "openai"` for any `/chat/completions`-compatible API,
   - `type: "gemini"` for the Google Generative Language API.
2. Add the API origin to `host_permissions` in `manifest.json`.
3. Reload the extension. The settings page and fallback list pick it up
   automatically.

### Prompt / SKILL.MD compliance

When **Draft letter** is clicked, `js/content.js` sends the job object scraped
from the page. The service worker then loads **`SKILL.MD`** from the extension,
splits off the letter spec (everything from `## When to use this skill` down)
and sends all three inputs to the AI:

1. **System message** — the letter part of `SKILL.MD` as the writing
   specification, plus rules that the job post and portfolio are data (not
   instructions), that no fact may be invented, and that the internal section
   labels in SKILL.MD's output format must never be printed in the letter.
2. **User message** — the scraped job (title, id, meta, skills, JOB OVERVIEW)
   followed by the candidate portfolio.

The returned text is written straight into the **Drafted letter** textarea and
saved locally. Output order follows `SKILL.MD`: **job instruction →
greeting/introduction → experience and skills → call to action → closing**.
The portfolio comes from the extension's `portfolio.MD` on every draft — there
is no Settings override to re-enter.

## LinkedIn posts

Open `https://www.linkedin.com/sharing/compose/` and the panel opens on the
right and drafts a short post automatically.

- The rules come from the **top of `SKILL.MD`** (the "When in LinkedIn"
  section). Everything from `## When to use this skill` down is the job-letter
  spec and is not sent for a post — and the LinkedIn section is not sent for a
  letter.
- The post is written from `portfolio.MD` only. It is 60–120 words, first
  person, opens with your portfolio link (as `SKILL.MD` requires), and has no
  hashtags, emojis, headings, or signature.
- The panel is promoted to the browser's **top layer** (a non-modal
  `popover="manual"`), so LinkedIn's share dialog and its full-page dim overlay
  cannot cover it. If LinkedIn opens another modal later, the panel is raised
  again — it never steals focus or makes the page inert.
- If LinkedIn's dialog is a native `<dialog>` (everything outside it is inert),
  the panel is moved inside the dialog — fixed positioning keeps it over the
  right edge — and moved back out when the dialog closes.
- The panel shows **LinkedIn post** / **Generate post** / **Copy post** instead
  of the job actions; **Apply now** and the Contact Info card are hidden.
- If a post was saved on an earlier visit it is restored and no new call is
  made. **Generate post** rewrites it.
- **Copy post** copies the text; paste it into LinkedIn's composer. Nothing is
  published automatically, and nothing is typed into the editor.
- Posts live under `jobHelperLinkedInPost`, separate from job letters.

Other LinkedIn composer URLs (for example the share box on the feed) are not
matched. To add one, extend `content_scripts.matches` in `manifest.json`.

## Files

```
manifest.json          MV3 config: content script match, action, permissions
js/content.js          Drawer UI, scraper, LinkedIn post mode, draft storage
js/panel-styles.js     All drawer CSS (injected into a Shadow DOM)
js/background.js       Service worker: toolbar toggle + letter/post AI routes
js/ai-providers.js     Free provider registry (shared with options page)
js/options.js          Settings page logic (keys, models, fallback)
css/options.css        Settings page styles
options.html           Settings page
portfolio.MD           Candidate facts - the only source of truth about you
SKILL.MD               LinkedIn post rules (top) + letter rules (split at runtime)
icons/                 16 / 32 / 48 / 128 px icons
```

## How it works

The content script runs on `https://www.onlinejobs.ph/jobseekers/job*`,
`https://www.onlinejobs.ph/apply*`, and `https://www.linkedin.com/sharing/compose*`.
It builds the panel inside a **Shadow DOM**, so the site's CSS cannot break the
UI and the panel's CSS cannot leak into the page. Styles are attached as a
constructable stylesheet (with an injected `<style>` fallback), which keeps the
panel styled even when a page, such as LinkedIn, ships a strict CSP.

On a job page it scrapes the overview and saves a snapshot. On the apply page it
reads that snapshot back and renders the same content, so the panel never goes
blank between "Apply now" and submitting the application. It picks the snapshot
in this order: job id in the apply URL (`?jobid=`, `?job_id=`, `?id=`, or
`/apply/123`), job id in `document.referrer` (the job page you came from), then
the most recent job.

Two alternatives were ruled out:

- **Chrome Side Panel API** — `sidePanel.open()` requires a user gesture, so it
  cannot pop out automatically when a job URL loads.
- **Extension iframe** — onlinejobs.ph sends a `Content-Security-Policy` with a
  `frame-src` list that does not include `chrome-extension:`, so the iframe
  would be blocked.

## Scraper

`collectJob()` in `js/content.js` uses these selectors (verified on a live page):

| Data | Selector |
| --- | --- |
| Title | `h1.job__title` |
| Overview | `#job-description` / `.job-description` |
| Meta | `.card.job-post dl` → `h3` label + `p` value |
| Skills | `.card-worker-topskill` |
| Job ID | `data-jobid` on the title/description |

If the overview element is missing, `fallbackOverview()` looks for a card
header whose text is `JOB OVERVIEW` and reads its body. Update these if the
site changes its markup.

## Editing and reloading

- Styles live inside `js/panel-styles.js` as a template string (this is what
  gets injected into the Shadow DOM).
- After changing `manifest.json`, `js/background.js`, or the content script,
  click the refresh icon on `chrome://extensions` **and reload the job page**.
- Debug with the page's DevTools: the panel is `#job-helper-panel-host` and its
  shadow root is open, so you can inspect it directly.
- Proposal drafts are stored in `chrome.storage.local` under `jobHelperProposal`.
  LinkedIn posts are stored under `jobHelperLinkedInPost`.
- Job snapshots live under `jobHelperJob` (latest) and `jobHelperJobs` (by job
  id, capped at 20 entries).
- Provider keys, models, and the fallback setting live under `jobHelperSettings`.
  Delete any of these keys from DevTools → Application → Storage if you want to
  reset.

## Permissions

- `storage` — remembers the proposal draft, job snapshot, and AI settings.
- `clipboardWrite` — powers **Copy letter**.
- `host_permissions` — the provider API origins listed in `manifest.json`, so
  the background worker can call the model you select.

The scraped job and portfolio are sent **only** to the provider you choose,
when you click Draft letter / Generate letter, when you click Generate post, or
when the LinkedIn composer opens (which drafts a post automatically), and on
Test connection. Nothing is sent on OnlineJobs.ph page load.
The only site access is the declared content-script match on onlinejobs.ph job
and apply pages, plus LinkedIn's share composer page.

## Notes

- `LAYOUT.MD` was empty when work started; it was filled in during the build
  and the implementation follows it.
- The panel is fixed-width (400 px, capped at 92vw) and overlays the page; the
  page underneath stays usable and scrollable.
- If the site ever adds a CSP that blocks injected styles, the fallback is to
  move the panel into an extension iframe — but that is not needed today.
