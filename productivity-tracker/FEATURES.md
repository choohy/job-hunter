# Productivity Tracker: feature map

A static page (`index.html`). There is no build step and no server. Data is saved in the browser's `localStorage`.

## Which features need AI

| Step / area | Feature | Needs AI? |
|---|---|---|
| Header | Import / Export (file download, file upload, clipboard) | No |
| Header | My CV and LinkedIn sections (stored text) | No |
| Header | Stats tiles (tasks, habits, apps, energy) | No |
| All steps | Step checklists, progress bar, celebration | No |
| All steps | "Tips and AI prompts" tips | No |
| All steps | "Tips and AI prompts" prompt buttons | **Yes** |
| 1 Reflect | Achievements (STAR/CAR library, rich text) | No |
| 1 Reflect | Strengths: VIA / Clifton / CV / Other lists | No |
| 1 Reflect | Strengths: reflection questions, feedback table, My thoughts | No |
| 1 Reflect | Strengths: "Analyse my CV for strengths" | **Yes** |
| 1 Reflect | Strengths: Section 4 "Suggest roles for me" | **Yes** |
| 1 Reflect | Missions | No |
| 1 Reflect | AI: About Me (profile summary) | **Yes** |
| 2 Plan | Today: check-in, Pomodoro, auto-sorted schedule, dependencies | No |
| 2 Plan | Tasks, archive | No |
| 2 Plan | Habits, streaks, 7-day dots, sync to Today | No |
| 2 Plan / 1 / 6 | Weekly: goal, target, wins, reflections | No |
| 3 Network | Contacts log, next steps, stale-contact flag | No |
| 3 / 4 | Resources and suggested links | No |
| 4 Apply | Job application pipeline, JD notes, notes | No |
| 4 Apply | Evaluate role / Generate resume / Generate cover letter | **Yes** |
| 5 Interview | AI: Interview (mock interview coach, saved sessions) | **Yes** |
| 5 Interview | AI: LinkedIn (section review chat) | **Yes** |
| 6 Thrive | AI: Salary (salary band estimate) | **Yes** |
| 6 Thrive | AI Coach (all buttons) | **Yes** |

Summary: 9 AI features out of 25 listed. The other 16 are plain data entry and logic that need no AI and no API key.

## How AI works on GitHub Pages

The original was a Claude.ai artifact. Inside Claude.ai, `fetch("https://api.anthropic.com/v1/messages")` works with no key because Claude.ai handles auth. On GitHub Pages it will not work: there is no key and no proxy.

This version:

- Adds **Settings**, where you paste your own Anthropic API key. The page calls the API straight from the browser using the `anthropic-dangerous-direct-browser-access` header.
- Stores the key in `localStorage` under `pt_settings`. It is never included in Export.
- Turns AI off when there is no key. AI tabs show a lock and a notice, and the rest of the app works as normal.

Risks to accept before you add a key:

1. **Shared origin.** Every GitHub Pages site on `<user>.github.io` shares one origin, so any page in any of your Pages repos can read this `localStorage`, including the key and your CV. Only host pages you trust there.
2. **Key exposure.** Anyone with access to the browser profile can read the key. Use a dedicated key with a low monthly spend limit.
3. **Public URL.** The page is public, but your data is not. Each visitor only sees their own browser's data.

If those risks are too high, the alternative is a small proxy (for example a Cloudflare Worker) that holds the key server-side. That is no longer "just GitHub Pages".

## Local cache limits

- About 5 MB per origin. Large CVs plus many saved interview sessions can fill it, and the app shows a warning if a save fails.
- Data is per browser and per device. It is lost if site data is cleared. Download a backup regularly.
- Safari can clear storage for sites you have not visited for 7 days. Back up more often on iPhone and Mac.

## Bugs fixed from the original code

1. **Wrong "today" in Melbourne.** `toISOString().slice(0,10)` is UTC. Before 10 or 11am AEDT/AEST, habits, streaks, check-ins and wins were stamped with yesterday's date. Now uses local date.
2. **Interview "Save and exit" crashed.** `saveSession`, `clearActiveSession`, `deleteSession` and `continueSession` were defined inside `LinkedInReviewChat`, not `InterviewPracticeChat`, so they threw a ReferenceError. Continuing a session also created duplicates.
3. **AI text never appeared in application fields.** `RichTextarea` only set its content on mount, so Evaluate, Resume and Cover letter results were invisible until you left the page and came back.
4. **Step "AI prompts" looked broken.** Responses only showed inside the AI Coach tab. Now they show under the step card.
5. **AI: LinkedIn tab could not switch sections.** It was passed no-op setters.
6. **Chat API calls could be rejected.** Chats sent history starting with an assistant message. The API needs the first message to be from the user.
7. **Stale state after async AI calls.** Edits made while waiting for AI could be overwritten. Saves now use functional updates.
8. **"Click a line to remove it"** removed every line containing that text, not just the one clicked.
9. **Task dependencies** needed you to type internal IDs. Now chosen from a list.
10. **Imported HTML ran unsanitised** via `dangerouslySetInnerHTML`. Now sanitised with DOMPurify.
11. Missing pieces: no delete for applications, `weeklyGoal` input started blank, Pomodoro drifted in background tabs, modals were unreadable in dark mode.

## Deploying

1. Repo **Settings → Pages → Build and deployment → Deploy from a branch**, choose `main` and `/ (root)`.
2. After merge, open `https://<user>.github.io/job-hunter/productivity-tracker/`.
3. GitHub Pages on a **private** repo needs a paid GitHub plan.

To move data from the Claude artifact: use Import / Export → copy JSON in the artifact, then paste it into Import here. The format is the same.
