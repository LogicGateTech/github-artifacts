# The Farm — Full-Stack Test Coverage Challenge

:pig2: :turkey:

> This is a **live** exercise — you're seeing this for the first time on the call, with about
> 35–40 minutes of coding time after setup. We're not expecting exhaustive coverage of everything
> below in that window; we're far more interested in your prioritization and process than in how
> much you finish. Talk out loud as you go.

### Overview

This is one full-stack application: an Angular UI (`angular-app/`) talking to a small
Express/TypeScript mock server (`mock-server/`). There are a number of rules about how animals
should be organized within the farm's array of colored barns, documented in detail in the
comments at the top of `mock-server/src/farmService.ts`, including two fully worked examples —
that comment is the specification the whole app (mock server and UI) is built against.

**Everything here is already fully implemented and working.** You are not being asked to build
anything. You're being asked to test it — as a real full-stack system, not two disconnected
pieces.

### Business Rules

What the app lets you do:

- Add an animal, giving it a name and a favorite color (chosen from a fixed list of colors).
- See every animal currently on the farm, organized into "barns." Each barn holds animals of
  exactly one color.
- Remove an animal from the farm.

The rule for how many barns exist, and who's in them:

- Each barn can hold up to 20 animals.
- All animals of the same favorite color share that color's barn(s). An animal is never placed in
  a barn of a different color.
- When every existing barn of a color is full, a new barn of that color is created.
- Animals of the same color are kept spread out as evenly as possible across however many barns
  of that color exist — not filling one barn before starting the next. Two barns of the same
  color should generally hold close to the same number of animals, not one nearly full and one
  nearly empty.
- When an animal is removed, the farm checks whether the remaining animals of that color could
  now fit into fewer barns without exceeding capacity. If so, the extra barn is removed entirely,
  and everyone left is redistributed evenly across the smaller number of barns.

Two worked examples:

**Growth.** Start with one YELLOW barn holding 20 animals (completely full).
- Add one more YELLOW animal: a second YELLOW barn is created, and the 21 animals are split
  evenly across the two barns (10 and 11).
- Add another YELLOW animal: now 22 total, still split evenly across 2 barns (11 and 11).

**Shrinking.** Start with 4 RED barns holding 15, 15, 16, and 16 animals (62 total).
- Remove one RED animal: 61 remain. That's still too many for 3 barns (3 × 20 = 60), so it
  stays at 4 barns, now holding 15, 15, 15, 16.
- Remove one more RED animal: 60 remain. That now fits exactly into 3 barns, so one barn is
  destroyed and the remaining 60 animals are split evenly into 3 barns of 20 each.

Worth thinking about: simple cases (add one animal, see it appear) are easy to verify and
probably not where the interesting risk lives. The growth and shrinking rules above have real
"boundary" moments — exactly where a barn gets created or destroyed. The app also has two things
that both need to agree: what these rules say should happen, and what you can actually see on
the screen once it happens. Those aren't automatically the same thing.

This same spec is also documented in the code, as a comment at the top of
`mock-server/src/farmService.ts` — the two worked examples above are copied verbatim from there.

### Running it

From this directory:

```
npm run setup
npm run dev
```

This is an npm workspaces project, so `setup` installs everything in one shot — the mock server,
the Angular app, and Playwright (including its browser binary) all come down with a single
`npm install` under the hood — so there's nothing else to set up later. `dev` then starts the
mock server on port 3001 and the Angular dev server on port 4200 together (which proxies `/api`
requests to the mock server, so the browser only ever talks to one origin).

Then open **http://localhost:4200** in a browser. You should see a working UI: a form to add an
animal by name and favorite color, and a live view of every barn and which animals are in it.
Adding and removing animals through the page calls the real API underneath — there's no mock
data baked into the frontend itself.

Leave it running for the rest of this exercise — the Playwright suite below talks to this same
running app.

### Troubleshooting

- **`Local:` line never appears / browser says "connection refused" at localhost:4200.** The
  Angular CLI requires **Node 18.13 or newer**. Check with `node -v`. If you're on an older
  version, switch (e.g. `nvm use --lts`, or `nvm install --lts` if you don't have it yet) and
  re-run `npm run dev`. If you use `nvm`, note that switching versions is per-terminal-session
  unless you set a default (`nvm alias default lts/*`) — if you open a new terminal partway
  through, it can silently revert to an older Node and cause this exact symptom.
- **`EADDRINUSE` on port 3001 or 4200.** Something from an earlier run is still listening. Find
  and stop it with `lsof -t -i:3001` (or `:4200`) and `kill <PID>`, then re-run `npm run dev`.
- **Playwright test fails with `Executable doesn't exist at ... chrome-headless-shell`.** The
  browser binary didn't finish downloading. Run `cd playwright && npx playwright install` and
  retry. (On Linux, if a test still fails to launch the browser after that, missing OS libraries
  are the likely cause — `npx playwright install --with-deps` additionally installs those via
  `apt-get`, but that flag is Linux-only and should not be used on macOS or Windows.)

### What's here

- `mock-server/src/farmService.ts` — the core business logic, and the spec comment describing
  the barn rules.
- `mock-server/src/server.ts` — the HTTP API (`GET`/`POST /api/v1/animals`,
  `DELETE /api/v1/animals/:id`) that the UI calls.
- `angular-app/src/app/` — the UI itself: standalone components (`animal-form`, `barn-list`),
  a service (`animal.service.ts`) wrapping the API calls, and the root `app.component`.
- `playwright/` — **your starting point**: true end-to-end tests that drive the real page in a
  real browser against the real running app. Two parallel starter suites are provided —
  `playwright/tests-js/` and `playwright/tests-ts/` — with the same example test and TODOs in
  both. Pick whichever language you're more comfortable with; you don't need to touch both.

### Task

Build the test coverage you'd want in place before you'd trust this app to ship. You're welcome
to add focused unit or component-level tests around the Angular or mock-server code if you think
they're worth the time, but the primary deliverable is the Playwright E2E suite — since that's
the layer that can actually tell you whether the UI and the API agree with each other.

Two things worth calling out:

1. Read the Business Rules section above (and the matching spec comment in `farmService.ts`)
   closely, including both worked examples. A thorough suite should be able to reproduce both
   worked examples on its own, at whichever layer makes sense to you.
2. The UI and the API are not independent of each other. Something that's true of the API in
   isolation isn't automatically true of what a user actually sees on the page, and vice versa.
   Where you land on that is part of what we want to talk through live.

You are **not** required to modify any production code (the Angular app or the mock server). If
your tests turn up a case where something doesn't do what it's documented or expected to do,
that finding — and how you'd describe/report it — is part of the deliverable.

### Rules

_You must:_
- use Playwright for anything that drives the real page

_You may:_
- use any assertion style, test data builder, or fixture pattern you like
- add unit or component tests for the Angular app or mock server if you think they add value
- restructure or replace any of the starter test files if you think that makes the suite better

_You may not:_
- modify the mock server (`mock-server/`) or the shared data models
- add a UI framework or third-party runtime dependency to the Angular app itself

### Running the tests

Playwright and its browser were already installed by `setup` above, so there's nothing extra to
set up — just run the suite:

```
cd playwright

npm run test:js                 # Playwright, JavaScript suite (needs the app running via `npm run dev` first)
# or
npm run test:ts                 # Playwright, TypeScript suite (same tests, same TODOs)

npm run test:js:ui               # either suite also has a --ui variant, e.g.:
npm run test:ts:ui
```

### Resources

- [Playwright docs](https://playwright.dev/docs/intro)
- [Angular docs](https://angular.dev/)
- [Angular testid / locator best practices](https://playwright.dev/docs/locators#locate-by-test-id)

---

:goat:
