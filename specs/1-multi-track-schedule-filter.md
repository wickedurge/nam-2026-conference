# Let attendees filter the schedule by several tracks at once

## What this changes
On `/schedule` the track pills become toggles: an attendee can select several, and the list shows sessions from any selected track. The URL carries them as `?track=a,b`; each pill exposes `aria-pressed`. "All" clears every selected track, and deselecting the last one also shows all tracks. A single-track link such as `?track=a` behaves as before. Any selected track still forces the list view and shows the grid-blocked notice. The Speakers Track filter is untouched.

## Where
- `server/routes/sessions.js` — the `trackSlug` line (~38): split on commas, trim, drop empties, and push `t.slug IN (?, ?)`; one value gives the same query as today. Update the Filters doc comment (~29). Test in `tests/api/endpoints.test.js` (API layer, `harness.js`): two slugs return the union of each slug's sessions.
- `src/pages/SchedulePage.jsx` — `filters.track` stays the raw comma string (so `qs()`, `activeKeys`, `BLOCKERS` and `clearBlocker` keep working). Derive `selectedTracks = filters.track === ALL ? [] : filters.track.split(',')`. Pill `onClick` (~174) toggles a slug in that list and calls `set('track', list.join(',') || ALL)`; `active = selectedTracks.includes(t.slug)`; add `aria-pressed={active}` to each pill (and to "All", pressed when none selected).
- `src/lib/api.js` — no change expected: `getSessions` passes `trackSlug` through `qs()`.
- `tests/schedule.spec.js` — new spec using `visit`, `waitForResults`, `laneFor` and the API's `/sessions?trackSlug=` counts (helpers in `tests/helpers.js`); read-only, no bookings. Select track A, then B, then deselect A, then "All", checking `result-count` against the API's count each time; pin the clock with `at`.
- `context/engineering-context/schedule.md` — `GET /api/sessions` and `SchedulePage` sections: `track`/`trackSlug` accepts several comma-separated values.

## How it will be proved
| Done when | Check | Layer |
| --- | --- | --- |
| Selecting two tracks shows both; deselecting one narrows | Spec: after each click `result-count` equals API count for the selected slugs, and `aria-pressed` matches | Playwright spec |
| "All" clears; deselecting last shows all | Same spec: after deselecting the last pill the URL has no `track` and count equals the all-tracks count; "All" click clears two | Playwright spec |
| Single-track link unchanged; endpoints test passes | `tests/api/endpoints.test.js` unchanged cases stay green; new API case compares `trackSlug=a,b` to the union of `a` and `b` (false on main: matches nothing) | API |
| Any track selected forces list and shows the notice | Spec: with two tracks in the URL on a wide viewport, `grid-blocked-notice` is visible | Playwright spec |
| Speakers Track filter as today | Existing `tests/speakers.spec.js` and `speakers.js` untouched | Playwright spec (existing) |
| Add then remove test compares to API | The spec above | Playwright spec |
| Doc describes several values | `node scripts/context.mjs` doc check via `tests/unit/context.test.js` stays green; doc edited in the same PR | Unit (existing) |

## Decisions
None left open; the ticket specifies the convention.
