# Prompt Heist – Delivery Plan

DevFest Pune 2026 · 20 working days · 67 stories · Node.js, Flutter Web, Firestore, Gemini on Vertex AI

**PR reviewer and QE:** Suraj

| Engineer | Role | Stories | Days | Finishes |
|---|---|---|---|---|
| [Shravani](team/Shravani.md) | Senior Backend | 10 | 18.0 | D18 |
| [Asmita](team/Asmita.md) | Backend | 14 | 18.0 | D18 |
| [Shreya](team/Shreya.md) | Infra, DevOps and Backend | 16 | 19.25 | D20 |
| [Mrudula](team/Mrudula.md) | Senior Frontend | 14 | 19.0 | D19 |
| [Harsimar](team/Harsimar.md) | Frontend | 13 | 18.5 | D20 |

Backend load is 55.25 of 60 engineer-days (92%). Frontend is 37.5 of 40 (94%). `FE-04` (hosting, Shreya) and `FE-31` (emoji faces, Asmita) count as backend load, because frontend has no spare capacity.

Each engineer's file has their stories with acceptance criteria, API contract, socket events, design links and a QE check.

## 1. Decisions

| # | Topic | Decision |
|---|---|---|
| D1 | Hosting | **Cloud Run**, one instance, always on. See below. |
| D2 | Ending a session | **Controller-driven, no timers.** `POST /sessions/:id/next-group` and `POST /sessions/:id/close`. See below. |
| D3 | Encryption | **No application-level encryption** of names or IDs. This overrides PRD 11 and the backend overview. Firestore still encrypts all data at rest by default. |
| D4 | After deletion | One record with no personal data, `system/eventStatus`, survives closure so the Controller can see the outcome. |
| D5 | Terms | **Event** is the full day. **Session** is one game with one round for one group. When it finishes, it is game over, and a fresh session starts with new players. |
| D6 | Prompt limit | **200 characters**, with a counter on the phone, enforced by the server. |
| D7 | Treasure image | **Mandatory.** A treasure without an image cannot be approved. |
| D8 | Emoji | **Each audience row has its own copy of one emoji**, made from an open-source 3D model and **pre-rendered** as short animations, so there is no live 3D. Its face reacts to each prompt. The Hindi-English text stays beside it. Shapes are only colour markers. |
| D9 | Apps | **One app, three routes:** `/controller`, `/audience`, `/player`. The domain is config and can change. |
| D10 | Unfilled groups | **The Controller abandons the session.** No replacing players or lowering capacity. |
| D11 | Colours | **Google logo palette.** |
| D12 | Entrance ID | **Any letters, numbers or special characters.** No format validation. |
| D13 | Controller login | **One account**, username and password set directly in the database. No second factor. This overrides PRD 13.2's strong authentication. |
| D14 | Duration | **Steps of 20 seconds, no maximum.** |
| D15 | Privacy note | **Removed from the screen.** This overrides PRD 13.3. |
| D16 | Names | **The Controller reviews every name before the round.** On reject, the Controller types the new name, which replaces it everywhere, including the CSV. Audience and phones see all names at once, only after all are approved. |
| D17 | Entrance IDs on screen | **Controller only.** Never on the big screen, because an ID alone is enough to rejoin as that player. |
| D18 | Copying prompts | **Allowed.** Identical text may get a different score, as it is judged by AI. |
| D19 | Guard scoring | Full prompt and method are in **BE-17**, based on `game.txt`. The model gives 40/30/30 marks and code adds them up. Smart, tailored tricks earn the top of the Quality marks. 100 needs full marks and all 3 hidden criteria met. |
| D20 | Missing screens | **Developers design them.** Marked 🎨 in stories. This includes where the treasure image goes. |
| D21 | Image outage | **Accepted.** If image generation fails, the round cannot start. |
| D22 | Provisional winner | **Shown on the big screen**, as the PRD says, even though a discard may follow. |
| D23 | Game style | **`game.txt` is the model, but not the only style.** Treasures can vary. Each has 3 hidden criteria, at least one beyond the visible objective, so repeating the objective cannot win. PRD unlock conditions and disallowed approaches are dropped. |
| D24 | Domain | The bare domain opens the player route. Final domain is needed before go-live, and is config only. |
| D25 | Score colours | 0–39 red, 40–79 yellow, 80–99 blue, 100 green, in the Google logo palette. |
| D26 | Entrance ID matching | `abc-123` and `ABC-123` are the same person. |
| D27 | Language | **No language restriction**, including Hindi and Hinglish, as long as it is not offensive. This overrides the PRD's English-only rule. |
| D28 | Starting a round | All players must be **connected**, and **all five agents must be up**. |
| D29 | Controller login | A new login ends the previous one. |
| D30 | After the game | The final result stays on phones until the Controller starts the next session, then phones clear. |
| D31 | Abandon | Needs a reason. Those people can join a later session. |
| D32 | Treasure review | One reject action for the whole treasure. No "reviewed" or "auto-filtered" counters. |
| D33 | Registration | Players enter their own details. |
| D34 | Discard | **The session ends with no winner.** The same people can join a new session with new codes. There is no replay inside a session. This overrides PRD 7.7. |
| D35 | Round number | Counts confirmed sessions only. A discarded session does not use up a number. |
| D39 | Reaction length | 100 characters at most. |
| D40 | No score | The player sees "Your score is zero". Such players rank last, in alphabetical order. |
| D41 | Slow scoring | Results wait up to 60 seconds. No warning to the Controller. |
| D42 | Fallback reactions | **None.** This overrides PRD 8.4. |
| D43 | Story time chip | Removed. |
| D44 | Close event day | Available at any time, with a confirmation dialog. Closing during play stops that round, and any unconfirmed result is lost. **A CSV must always be exported first**, covering every winner. With no winners, an empty CSV is still required. |
| D45 | Next group | Reuses the last settings. An **End session** button is added for the last group. |
| D46 | Prompts on the big screen | Shown in full, never cut off. |
| D47 | Final ranking | All players shown on the big screen. |
| D48 | Mockups | Not final. Developers can change them as needed. |
| D49 | Failed reactions | No reaction text and a neutral face. |
| D50 | Blocked prompts | The Controller sees **one blocked total**. No filter categories. This overrides PRD 8.2. |
| D51 | Choosing the winner | **Plain code**, never AI: highest best score, then earliest time, then lowest sequence number. The AI prompt in `game.txt` section 5 is not used. |

**The PRD should be updated** for D3, D5, D8, D9, D13, D15, D16, D23, D27, D34, D42 and D50.

### D1: Cloud Run, not Cloud Functions or EC2

| | Cloud Run | Cloud Functions | EC2 |
|---|---|---|---|
| Live sockets | Yes, reconnect every 60 minutes | Poor fit: built for short requests | Yes |
| In-process timer and queue | Yes, with instance-based billing | No | Yes |
| Access to Firestore and Vertex AI | Native, no key files | Native | Cross-cloud, needs stored keys |
| Operations | Container deploy, instant rollback | Simple, but wrong model | You patch the OS and manage TLS, deploys and restarts |

The game needs one long-running process that holds sockets, runs the round timer and scores prompts after replying. Cloud Functions is built for the opposite. EC2 would work, but it adds an AWS account, cross-cloud credentials and server upkeep, with no benefit for 5 players. If a plain VM were ever needed, Compute Engine would be the better choice for the same reason.

Settings: `min-instances=1`, `max-instances=1`, instance-based billing, session affinity, 3600-second timeout.

Two limits to design for, both covered in stories:

- Cloud Run closes each socket at 60 minutes. Clients must reconnect and rehydrate (FE-02).
- An instance can still restart. Timers and unfinished prompts recover from Firestore on start-up (BE-20, BE-22).

### D2: Next group

```
RESULTS ── Next group ──► session CLOSED + new session JOINING (one transaction)
RESULTS ── End session ─► session CLOSED, nothing open
JOINING / TREASURE_REVIEW / READY ── Abandon + reason ─► session CLOSED
RESULT_PENDING ── Discard + reason ─► session DISCARDED, no winner
```

- The session stays in `RESULTS` until the Controller acts. Nothing is timed.
- The design already has a **Next group** button, so this matches the screens.
- `next-group` needs an `Idempotency-Key`, so a double click cannot open two sessions.

## 2. Open questions

None. Every question has been answered and moved to Decisions. If the PRD, a screen and a story disagree during the build, raise it with Suraj rather than guessing.

## 3. Design corrections and engineering defaults

**Where the PRD is explicit, so the screen changes:**

- **Truncated labels:** the display cuts "Meera (Player 5)" to "Meera (Player ...", removing the only thing separating two Meeras (PRD 5.1). FE-21 forbids this.
- **Player state:** the result-pending screen shows one player as "Playing". PRD 10.2 says Result pending.
- **Audit log and winners list:** shown in the Controller sidebar on every screen, so the audit view is required.
- **Prototype text must not ship:** demo code hints, "any email and password work", the flask button, and the blue privacy note.
- **Login label:** the mock says Email, but the account uses a username.
- **Audience joining screen:** the mock shows names one by one. They now appear all at once (D16).
- **Phone placeholder:** the mock says "in English". Any language is now allowed (D27).

**Sample numbers in `game.txt`:** the same two messages score 90 and 40 in the Guard section, but 82 and 25 in the emoji section. This fits D18, so stories use the Guard numbers as the baseline.

**Engineering defaults.** These are values picked to make the contract concrete. They are not product rules, but tell us if any should differ:

Login limit 5 tries per minute · Controller token 12 hours · join token 5 minutes · rejoin limit 10 per IP per minute · prompt throttle 5 per second · `timer:sync` every 3 seconds · filter regeneration 3 tries · image timeout 45 seconds with 2 retries · emoji timeout 2 seconds · name maximum 80 and ID maximum 128 characters · join codes 10 to 99 · signed image URLs 15 minutes and 4 hours · confirmation phrase `CLOSE EVENT` · no new sessions after closure until `event:reset` is run.

## 4. Schedule

| Days | Milestone | Done when |
|---|---|---|
| D1–2 | **M0 Contract** | Mock server and event replay run. Dev environment is up. Gemini text and image models confirmed. |
| D3–11 | **M1 Join flow** | Real backend: login, open session, 5 players join and rejoin, shown on Controller and audience screen. |
| D12–14 | **M2 Game loop** | Real backend and Gemini: treasure with image, approve, start, prompts scored live. |
| D15–18 | **M3 Feature complete** | Result, confirm, discard, next group, name review, CSV and close all work. **Feature freeze: frontend end of D17, backend morning of D18.** |
| D17–20 | **M4 Hardening** | Backend: load run, scoring calibration, harness. D19–20: full rehearsals on 5 real phones and the projector. Go or no-go on D20. |

The schedule is calculated from each person's story order and dependencies. The longest wait on another person is half a day. It assumes full-time work with no leave, so treat it as the plan, not a promise.

## 5. Risks

| # | Risk | Mitigation |
|---|---|---|
| R1 | **Review bottleneck.** About 67 PRs reach one reviewer and QE, roughly 4 a day up to freeze. | Small PRs, same-day review. Consider Shravani and Mrudula doing a first pass on their side. |
| R2 | **Load is very high**, especially frontend, after adding dev-designed screens, name review and the reacting emoji. There is almost no slack for sickness or leave. | Freeze moved from D16 to D17–18, leaving about two days of full rehearsal. Use the cut list. |
| R4 | **Scoring quality.** Unstable or unfair scores ruin the game. | BE-17, BE-34, and play-tests with non-engineers. |
| R5 | **Late real integration.** Scoring reaches the real backend around D14. | Mock server from D2, fakes from D3, harness in CI. |
| R6 | **Rejoin hijack.** An entrance ID alone restores a player (PRD 7.3), and IDs like `DF-2026-0007` could be guessed. | Rate limit (BE-12), IDs kept out of logs and off the display. |
| R3 | **Audience row layout.** Five rows must fit a full 200-character prompt, a 100-character reaction, an animated face and a score, with no scrolling. | Mockups can change (D48). Test real long text on the projector early in FE-21. |
| R7 | **Flutter Web on phones.** First load size, iOS keyboard, emoji rendering. | Measured in FE-04, tested in FE-24, checked in FE-21. |

## 6. Cut list, in order

1. BE-30 throttling and BE-33 dashboards
2. Display polish in FE-21
3. Monitor error history in BE-26 (keep scores and connections)

## 7. Technology notes

- **Gemini SDK:** `@google/genai` with `vertexai: true`. The older `@google-cloud/vertexai` package has been removed.
- **Model IDs:** deliberately not fixed here. Search results listed several candidates, such as `gemini-2.5-flash-image` and Imagen 4, but their status and regional availability could not be confirmed. BE-38 pins them on Day 1 from your GCP project, as env config.
- **Suggested split:** a fast Flash-class model for filter, guard and emoji, because three calls run in sequence for each prompt. A stronger model for treasure text only.

Sources: [Cloud Run WebSockets](https://docs.cloud.google.com/run/docs/triggering/websockets) · [Cloud Run billing settings](https://docs.cloud.google.com/run/docs/configuring/billing-settings) · [Session affinity](https://docs.cloud.google.com/run/docs/configuring/session-affinity) · [Vertex AI SDK migration](https://docs.cloud.google.com/vertex-ai/generative-ai/docs/deprecations/genai-vertexai-sdk) · [Vertex AI image generation](https://docs.cloud.google.com/vertex-ai/docs/generative-ai/image/overview)

## 8. All stories

| Story | Title | Owner | Days | Planned | Needs | Priority |
|---|---|---|---|---|---|---|
| BE-02 | API contract, mock server and event simulator | Shravani | 1.5 | D1–D2 | – | Must |
| BE-03 | Firestore data model and repositories | Shravani | 1.5 | D2–D3 | BE-01 | Must |
| BE-07 | Realtime gateway | Shravani | 2.5 | D4–D6 | BE-01, BE-03, BE-02 | Must |
| BE-17 | guardAgent | Shravani | 2 | D6–D8 | BE-14 | Must |
| BE-20 | Start round and timer job | Shravani | 1.5 | D8–D9 | BE-03, BE-06, BE-07 | Must |
| BE-21 | Prompt intake | Shravani | 2 | D10–D11 | BE-03, BE-06, BE-20 | Must |
| BE-22 | Async scoring pipeline | Shravani | 2.5 | D12–D14 | BE-21, BE-14 | Must |
| BE-24 | Round end and resultCalculator | Shravani | 1.5 | D14–D15 | BE-20, BE-22 | Must |
| BE-32 | Security and privacy review | Shravani | 1 | D16 | BE-13, BE-22, BE-24 | Must |
| BE-34 | Guard calibration and red-team suite | Shravani | 2 | D17–D18 | BE-17, BE-16, BE-22 | Must |
| BE-06 | Session, player and prompt state machines | Asmita | 1.5 | D1–D2 | – | Must |
| BE-05 | Controller auth and RBAC | Asmita | 1.5 | D2–D3 | BE-01 | Must |
| BE-08 | Create session and dashboard state | Asmita | 1.5 | D4–D5 | BE-03, BE-05, BE-06 | Must |
| BE-09 | Change round duration | Asmita | 0.5 | D5 | BE-08 | Must |
| BE-10 | Join code check | Asmita | 1 | D6 | BE-08 | Must |
| BE-11 | Player join | Asmita | 1.5 | D7–D8 | BE-10, BE-07 | Must |
| BE-12 | Rejoin, kick and connection state | Asmita | 1.5 | D8–D9 | BE-11 | Must |
| BE-13 | Player and display state projections | Asmita | 1.5 | D10–D11 | BE-07, BE-11 | Must |
| FE-31 | Emoji faces from an open-source 3D model | Asmita | 1 | D11–D12 | – | Must |
| BE-18 | emojiAgent and expressions | Asmita | 1.5 | D12–D13 | BE-14, BE-15 | Must |
| BE-23 | Prompt status and player result reads | Asmita | 0.5 | D14 | BE-13 | Must |
| BE-25 | Confirm and discard | Asmita | 1 | D14–D15 | BE-06, BE-03 | Must |
| BE-36 | Next group and close session | Asmita | 1 | D15–D16 | BE-25, BE-08 | Must |
| BE-31 | Simulation harness and fairness tests | Asmita | 2.5 | D16–D18 | BE-24, BE-25, BE-36 | Must |
| BE-01 | Service skeleton and CI | Shreya | 0.5 | D1 | – | Must |
| BE-38 | GCP environment and Cloud Run deployment | Shreya | 1.5 | D1–D2 | BE-01 | Must |
| BE-14 | AI gateway and agent health | Shreya | 2 | D3–D4 | BE-38 | Must |
| FE-04 | Frontend hosting | Shreya | 0.5 | D5 | BE-38, FE-01 | Must |
| BE-04 | Log redaction | Shreya | 0.25 | D5 | BE-01 | Must |
| BE-15 | filterAgent | Shreya | 1.5 | D5–D7 | BE-14 | Must |
| BE-16 | treasureAgent and rubric template | Shreya | 2 | D7–D9 | BE-14, BE-15 | Must |
| BE-35 | Treasure image generation and storage | Shreya | 2 | D9–D11 | BE-14, BE-15 | Must |
| BE-19 | Treasure APIs | Shreya | 2 | D11–D13 | BE-16, BE-35, BE-06, BE-08 | Must |
| BE-29 | Close event and verified deletion | Shreya | 2.5 | D13–D15 | BE-03, BE-35 | Must |
| BE-28 | Winners CSV | Shreya | 0.5 | D15–D16 | BE-03 | Must |
| BE-26 | Monitor and audit log APIs | Shreya | 1 | D16–D17 | BE-03, BE-07 | Must |
| BE-39 | Controller name review | Shreya | 1 | D17–D18 | BE-11, BE-20 | Must |
| BE-37 | Load test and event-day runbook | Shreya | 1 | D18–D19 | BE-22, BE-24 | Must |
| BE-30 | Abuse throttling | Shreya | 0.5 | D19 | BE-21 | Should |
| BE-33 | Observability | Shreya | 0.5 | D19–D20 | BE-14 | Should |
| FE-01 | Flutter workspace and app scaffold | Mrudula | 1 | D1 | – | Must |
| FE-02 | API and socket client | Mrudula | 2 | D2–D3 | FE-01 | Must |
| FE-03 | Server-synced countdown | Mrudula | 0.5 | D4 | FE-02 | Must |
| FE-05 | Booth code screen | Mrudula | 1 | D4–D5 | FE-02, FE-26 | Must |
| FE-06 | Name, entrance ID and lobby | Mrudula | 1 | D5–D6 | FE-05 | Must |
| FE-07 | Rejoin and kicked screens | Mrudula | 1 | D6–D7 | FE-06 | Must |
| FE-08 | Battle screen | Mrudula | 2.5 | D7–D9 | FE-03, FE-06 | Must |
| FE-09 | Prompt feedback | Mrudula | 1 | D10 | FE-08 | Must |
| FE-10 | Unlocked, time up, result, discarded, closed | Mrudula | 2 | D11–D12 | FE-09 | Must |
| FE-19 | Audience route shell | Mrudula | 1 | D13 | FE-02, FE-26 | Must |
| FE-20 | Audience idle, joining and get ready | Mrudula | 0.5 | D14 | FE-19 | Must |
| FE-21 | Audience battle view | Mrudula | 3 | D14–D17 | FE-20, FE-03, FE-31 | Must |
| FE-22 | Unlock moment | Mrudula | 0.5 | D17 | FE-21 | Must |
| FE-24 | Player and display integration | Mrudula | 2 | D18–D19 | FE-10, FE-22, BE-24, BE-36 | Must |
| FE-26 | Shared design widgets | Harsimar | 2 | D1–D2 | – | Must |
| FE-11 | Controller login and auth guard | Harsimar | 0.5 | D3 | FE-01, FE-26 | Must |
| FE-12 | Open session and booth codes | Harsimar | 2 | D4–D5 | FE-02, FE-11 | Must |
| FE-13 | Treasure review | Harsimar | 2 | D6–D7 | FE-12 | Must |
| FE-14 | Ready and start | Harsimar | 1 | D8 | FE-13 | Must |
| FE-28 | Name review | Harsimar | 1 | D9 | FE-14 | Must |
| FE-15 | Live monitor | Harsimar | 2 | D10–D11 | FE-14, FE-03 | Must |
| FE-16 | Result review | Harsimar | 1.5 | D12–D13 | FE-15 | Must |
| FE-27 | Winner confirmed and Next group | Harsimar | 0.5 | D13 | FE-16 | Must |
| FE-17 | Event day card: winners, CSV, close | Harsimar | 2 | D14–D15 | FE-12 | Must |
| FE-18 | Audit log panel | Harsimar | 0.5 | D16 | FE-17 | Must |
| FE-23 | Audience result screens | Harsimar | 1.5 | D16–D17 | FE-19, FE-26 | Must |
| FE-25 | Controller integration | Harsimar | 2 | D18–D20 | FE-17, FE-18, FE-27, FE-28, BE-29, BE-36, BE-39 | Must |

## 9. API index

Full request and response details are in the stories. New or changed from `Backend_overview.pdf`: **1** uses a username, **7 and 9** are now asynchronous, and **28, 29 and 30** are new.

| # | Endpoint | Auth | Backend story | Frontend stories |
|---|---|---|---|---|
| 1 | `POST /api/controller/auth/login` | none | BE-05 Asmita | FE-11 |
| 2 | `POST /api/controller/auth/logout` | Controller | BE-05 Asmita | FE-11 |
| 3 | `GET /api/controller/auth/me` | Controller | BE-05 Asmita | FE-11 |
| 4 | `POST /api/controller/sessions` | Controller | BE-08 Asmita | FE-12 |
| 5 | `GET /api/controller/sessions/active` | Controller | BE-08 Asmita | FE-12, FE-17 |
| 6 | `PATCH /api/controller/sessions/:id/duration` | Controller | BE-09 Asmita | FE-12, FE-14 |
| 7 | `POST /api/controller/sessions/:id/treasure` | Controller | BE-19 Shreya | FE-13 |
| 8 | `POST /api/controller/treasure/:id/approve` | Controller | BE-19 Shreya | FE-13 |
| 9 | `POST /api/controller/treasure/:id/reject` | Controller | BE-19 Shreya | FE-13 |
| 10 | `POST /api/controller/sessions/:id/start` | Controller | BE-20 Shravani | FE-14 |
| 11 | `GET /api/controller/rounds/:id/monitor` | Controller | BE-26 Shreya | FE-15 |
| 12 | `GET /api/controller/rounds/:id/result` | Controller | BE-24 Shravani | FE-16 |
| 13 | `POST /api/controller/rounds/:id/confirm` | Controller | BE-25 Asmita | FE-16 |
| 14 | `POST /api/controller/rounds/:id/discard` | Controller | BE-25 Asmita | FE-16 |
| 15 | `GET /api/controller/winners/export` | Controller | BE-28 Shreya | FE-17 |
| 16 | `POST /api/controller/event/close` | Controller | BE-29 Shreya | FE-17 |
| 17 | `GET /api/controller/event/status` | Controller | BE-29 Shreya | FE-17 |
| 18 | `GET /api/controller/health/agents` | Controller | BE-14 Shreya | FE-14 |
| 19 | `GET /api/controller/audit-logs?limit=50&cursor=` | Controller | BE-26 Shreya | FE-18 |
| 20 | `POST /api/player/join/code` | none | BE-10 Asmita | FE-05 |
| 21 | `POST /api/player/join` | joinToken | BE-11 Asmita | FE-06 |
| 22 | `POST /api/player/rejoin` | none | BE-12 Asmita | FE-07 |
| 23 | `GET /api/player/me` | Player | BE-13 Asmita | FE-06, FE-07, FE-08 |
| 24 | `POST /api/player/prompts` | Player | BE-21 Shravani | FE-08 |
| 25 | `GET /api/player/prompts/:id` | Player | BE-23 Asmita | FE-09 |
| 26 | `GET /api/player/result` | Player | BE-23 Asmita | FE-10 |
| 27 | `GET /api/display/state` | none (public) | BE-13 Asmita | FE-19 |
| 28 | `POST /api/controller/sessions/:id/close` | Controller | BE-36 Asmita | FE-12, FE-27 |
| 29 | `POST /api/controller/sessions/:id/next-group` | Controller | BE-36 Asmita | FE-27 |
| 30 | `POST /api/controller/players/:id/name-review` | Controller | BE-39 Shreya | FE-28 |

### Socket events

New compared with `Backend_overview.pdf`: `session:opened`, `session:closed`, `treasure:ready`, `treasure:failed`, `monitor:stats`, `event:status`.

| Event | Sent to |
|---|---|
| `phase:changed` | all |
| `session:opened` | display, controller |
| `session:closed` | all |
| `player:joined` | all |
| `players:revealed` | audience, players |
| `controller:kicked` | old controller socket only |
| `player:connection` | all |
| `player:kicked` | old player socket only |
| `treasure:ready` | controller only |
| `treasure:failed` | controller only |
| `round:started` | all |
| `timer:sync` | all, every 3 seconds while PLAYING |
| `prompt:scored` | that player only |
| `prompt:blocked` | that player only |
| `prompt:failed` | that player only |
| `feed:update` | display, controller |
| `player:unlocked` | all |
| `monitor:stats` | controller only |
| `round:ended` | all |
| `result:provisional` | display, controller |
| `result:confirmed` | all |
| `round:discarded` | all |
| `event:status` | controller only |

## 10. Design reference

| Screen | File ends in |
|---|---|
| [Player: booth code](WhatsApp%20Image%202026-11-05%20at%2019.30.08%20%281%29.jpeg) | `19.30.08 (1)` |
| [Player: name and ID](WhatsApp%20Image%202026-11-05%20at%2019.30.08%20%282%29.jpeg) | `19.30.08 (2)` |
| [Player: lobby](WhatsApp%20Image%202026-11-05%20at%2019.30.09.jpeg) | `19.30.09` |
| [Player: battle](WhatsApp%20Image%202026-11-05%20at%2019.30.08.jpeg) | `19.30.08` |
| [Player: time up](WhatsApp%20Image%202026-11-05%20at%2019.30.09%20%281%29.jpeg) | `19.30.09 (1)` |
| [Player: final result](WhatsApp%20Image%202026-11-05%20at%2019.30.07.jpeg) | `19.30.07` |
| [Display: joining](WhatsApp%20Image%202026-11-05%20at%2019.30.10.jpeg) | `19.30.10` |
| [Display: get ready](WhatsApp%20Image%202026-11-05%20at%2019.30.10%20%281%29.jpeg) | `19.30.10 (1)` |
| [Display: battle](WhatsApp%20Image%202026-11-05%20at%2019.30.10%20%282%29.jpeg) | `19.30.10 (2)` |
| [Display: unlocked](WhatsApp%20Image%202026-11-05%20at%2019.30.11.jpeg) | `19.30.11` |
| [Display: time up](WhatsApp%20Image%202026-11-05%20at%2019.30.09%20%282%29.jpeg) | `19.30.09 (2)` |
| [Display: winner and story](WhatsApp%20Image%202026-11-05%20at%2019.30.11%20%281%29.jpeg) | `19.30.11 (1)` |
| [Controller: login](WhatsApp%20Image%202026-11-05%20at%2019.30.12.jpeg) | `19.30.12` |
| [Controller: setup](WhatsApp%20Image%202026-11-05%20at%2019.30.12%20%281%29.jpeg) | `19.30.12 (1)` |
| [Controller: joining](WhatsApp%20Image%202026-11-05%20at%2019.30.12%20%282%29.jpeg) | `19.30.12 (2)` |
| [Controller: treasure review](WhatsApp%20Image%202026-11-05%20at%2019.30.13.jpeg) | `19.30.13` |
| [Controller: ready](WhatsApp%20Image%202026-11-05%20at%2019.30.11%20%282%29.jpeg) | `19.30.11 (2)` |
| [Controller: live monitor](WhatsApp%20Image%202026-11-05%20at%2019.30.13%20%281%29.jpeg) | `19.30.13 (1)` |
| [Controller: result pending](WhatsApp%20Image%202026-11-05%20at%2019.30.13%20%282%29.jpeg) | `19.30.13 (2)` |
| [Controller: winner confirmed](WhatsApp%20Image%202026-11-05%20at%2019.30.14.jpeg) | `19.30.14` |

## Shared contract (applies to every story)

**Words we use:** an **event** is the full day. A **session** is one game for one group of 4 or 5 players, with one **round**: the timed battle. When it finishes, the game is over, and a fresh session starts with new players. A discard ends the session with no winner.

**One app, three routes.** The domain is config and can change. The bare domain opens the player route.

| Surface | Path |
|---|---|
| Controller | `/controller` |
| Audience (big screen, called "display" in the API) | `/audience` |
| Player | `/player` |

**Conventions**

- JSON, camelCase. Timestamps are ISO 8601 UTC with milliseconds. Durations are whole seconds.
- `Authorization: Bearer <token>`. Token types: `controllerToken` (12 hours), `joinToken` (5 minutes), `playerToken` (until the session closes or the player rejoins elsewhere).
- Every response has `X-Correlation-Id`.
- The client never sends a player ID, score, timestamp or winner. The server works these out.
- All writes go through REST. Sockets only send from server to client.
- After any socket reconnect, call the hydrate endpoint: `GET /api/controller/sessions/active`, `GET /api/player/me` or `GET /api/display/state`.

**Sockets:** namespaces `/controller`, `/player`, `/display` on the API host. Pass the token in `auth: { token }`. `/display` needs no token.

**Error format**

```json
{ "error": { "code": "ROUND_EXPIRED", "message": "Safe text for the user", "correlationId": "..." } }
```

**Enums**

| Name | Values |
|---|---|
| Session `phase` | `SETUP` (no session open), `JOINING`, `TREASURE_REVIEW`, `READY`, `PLAYING`, `RESULT_PENDING`, `RESULTS`, `DISCARDED`, `CLOSED` |
| Player `state` | `LOBBY`, `PLAYING`, `LOCKED_100`, `RESULT_PENDING`, `COMPLETE` |
| `connection` | `CONNECTED`, `DISCONNECTED` |
| `slotState` (Controller) | `UNJOINED`, `LOBBY`, `PLAYING`, `LOCKED_100`, `DISCONNECTED`, `RESULT_PENDING`, `COMPLETE` |
| Prompt `status` | `RECEIVED`, `FILTERING`, `BLOCKED`, `APPROVED`, `SCORING`, `SCORED`, `FAILED`, `DISPLAYED` |
| Event `status` | `OPEN`, `CLOSING`, `CLOSED`, `CLOSING_FAILED` |

**Colour markers, fixed by slot.** These only tell players apart. Separately, each audience row has its own copy of one shared emoji, made from an open-source 3D model and pre-rendered, whose face reacts to each prompt.

| Slot | `shape` | Look |
|---|---|---|
| 1 | `BLUE_CIRCLE` | Blue circle |
| 2 | `RED_SQUARE` | Red rounded square |
| 3 | `YELLOW_DIAMOND` | Yellow diamond |
| 4 | `GREEN_PILL` | Green pill |
| 5 | `GREEN_CIRCLE` | Green circle |

**Colours:** Google logo palette: blue `#4285F4`, red `#EA4335`, yellow `#FBBC05`, green `#34A853`. Scores: 0–39 red, 40–79 yellow, 80–99 blue, 100 green.

**Must never reach a phone or the audience screen:** full names of other people, entrance IDs, the rubric, unapproved treasures, blocked or pending prompts, agent credentials, system prompts, raw model output.

**Working agreement**

- One story, one PR. Suraj reviews every PR and runs the QE check, so keep PRs small and open them as soon as a story is done.
- Your PR description says how to run the QE check.
- A story is done when its acceptance criteria pass, tests are green, the contract is up to date, and it is demoed on dev. Frontend stories are demoed against the mock if the backend story has not landed, and rechecked in integration.
- `game.txt` holds the sample agent prompts. The stories adapt them, so use the story version.
- Backend: agents sit behind interfaces, so use the `Fake*` versions until the real one lands. Use the injected `Clock` in tests rather than sleeping.
- Frontend: build against the mock server. Do not wait for real endpoints.
- **Mockups are not final.** Change them where the story or the screen needs it.
- Frontend: some screens have no mock. Stories marked 🎨 are yours to design, using the existing style. Show the design in your PR.
- **Do not guess business rules.** If a story, the PRD and a screen disagree, or a rule is unclear, raise it with Suraj. Stories marked with an open question stay partly blocked until it is answered.
- A "Needs" entry means the story cannot finish without that work. If you are blocked, say so in stand-up the same day.

