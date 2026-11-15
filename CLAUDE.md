# Treasured — notes for Claude

Read this first. It explains what this project is, how it is put together, and how Val likes to work.

## What it is

Treasured is a Scripture memory app made by Val. It is free, with no accounts and no server.
A person picks a verse or passage, a start date and a pace, and the app builds a schedule on its
own built-in calendar.

The method (do not change it without asking Val):

- A new verse is said 25 times on day 1, 20 on day 2, 15 on day 3 and 10 on day 4.
- Then once a day for 45 days.
- Then once a week for 7 weeks.
- Then on the first of every month.
- New verses start at the chosen pace: every 1, 2, 3 or 7 days, or on chosen weekdays.
- A "rest day" pushes the whole schedule forward by one day (stored as `shifts` on the plan).

The wording of the instructions is our own. Do not copy text from the printed handout the method
came from.

## Working with Val

- Val is not a programmer. Explain in plain words, define any technical term, and give numbered
  steps for anything she has to do herself.
- She prefers Claude to do the building. Keep her steps few and short.
- She is cost-conscious: no paid services without asking first.
- She cares a lot about the look. Show changes rather than describe them.

## Files

On Val's Mac the project is in `Documents/Treasured`, in two folders:
`UPLOAD-TO-GITHUB/` (everything in the GitHub repository, this file included) and `ios/`
(the iPhone project, kept on the Mac only). Paths below starting with `ios/` are in that
second folder; the rest are in the repository.

- `index.html` — the whole app: markup, styles and script in one file, plain JavaScript,
  no build step, no libraries. State is saved in `localStorage` under the key `treasured.v1`.
- `assets/flow.jpg` and `assets/ribbon.jpg` — the two backgrounds.
- `ios/Treasured.xcodeproj` — Xcode project (hand-written; uses a synchronized folder group, so
  any file placed in `ios/Treasured/` is included automatically).
- `ios/Treasured/index.html` — a copy of the web page for the iPhone app (see "Keeping the two
  copies in step").
- `ios/Treasured/TreasuredApp.swift` — shows the page in a web view and schedules reminders.
- `README.md` — plain-English overview and App Store steps.

The web version is published with GitHub Pages from the repository `treasured`
(index.html and the assets folder at the top level of the repository). When you change
anything, give Val one folder to upload from, with no old copies left beside it.

## How the code is organised (`index.html`)

- Between `/*LOGIC*/` and `/*END*/`: pure functions for dates and the schedule
  (`planDay`, `buildOffsets`, `stage`, `tasksFor`, `dateOfPlanDay`, `parsePaste`, `planTitle`).
  These have no page access, so they can be tested with Node.
- `S` is saved state: `theme` ("A" or "B"), `still`, `welcomed`, `plans`, `reminders`
  (seven "HH:mm" strings, Sunday first), `done` (tap counts keyed `planId|verseIndex|date`).
- `UI` is unsaved screen state. `render()` redraws the current screen from `S` and `UI`.
- Buttons carry `data-a="actionName"`; one click listener looks the action up in `ACT`.
- The setup wizard is `UI.wiz` (five steps). `syncWiz()` copies typed values in before a redraw.
- Verses come from two places: `https://bible-api.com` when the user adds them (first the
  address `/data/{translation}/{BOOK}/{chapter}`, then the older `/{reference}?translation=` as a
  backup), or text the user pastes in.

## Keeping the two copies in step

After changing `index.html`, remake `ios/Treasured/index.html` from it with three changes:

1. On the line `var IMG_A='assets/flow.jpg',IMG_B='assets/ribbon.jpg';` replace the two paths
   with the images as `data:image/jpeg;base64,...` strings.
2. Add `, viewport-fit=cover` to the viewport meta tag.
3. Before `</style>` add safe-area padding:
   `main{padding-top:calc(28px + env(safe-area-inset-top))}`
   `.theme-b main{padding-top:calc(150px + env(safe-area-inset-top))}`
   `#tabs{bottom:calc(16px + env(safe-area-inset-bottom))}`

## iPhone reminders

The page calls `window.webkit.messageHandlers.treasured.postMessage({reminders, active})`
whenever state is saved. `TreasuredApp.swift` then schedules one repeating notification per
weekday, with three actions: remind me in 1, 2 or 3 hours. `isNative()` in the page tells
whether it is running inside the iPhone app.

## Design

- Two looks the user chooses between: A "Flowing gold" (red, white and gold liquid with frosted
  glass cards) and B "Silk ribbon" (crimson ribbon on white, solid white cards, no glitter).
- Colours: ink `#161413`, grey text `#5E5955`, crimson `#A4161A`, deep red `#7D0F13`,
  gold `#B8892B`. Gold is for lines and accents only; gold text on white is too faint to read.
- Fonts: Cormorant Garamond for verses and headings, DM Sans for everything else.
- Backgrounds drift slowly, and look A has gold glitter drawn on a canvas (`#glitter`, the
  `glitter…` functions). Look B ripples: the ribbon picture is redrawn in swaying strips on
  a canvas (`#ribbon`, the `ribbon…` functions) with a soft moving highlight (`#sheen`). "Still mode" in Settings turns all motion off.
- The motto is "Bury God's Word deep in your heart" (Val's wording; keep it).
- Both background images were generated for this project (Python scripts), not taken from
  anywhere else.

## Languages

The app is in English and Spanish. The user picks on the welcome screen or in Settings
(`S.lang`, mirrored in the global `LANG`). Every on-screen string goes through `tx('English text')`
or `txf('Text with {a}', value)`; the Spanish wording lives in the `ES` table at the top of the
script, keyed by the English text. When you add or change any wording, add its Spanish line to
`ES` too. Dates use `fmt()`, weekday and month names use `wd()`, `wdl()` and `mo()`.
Bible lookups use the King James Version in English and the Reina-Valera 1909 in Spanish.
The iPhone reminder text is also in both languages (see `Reminders` in `TreasuredApp.swift`).

## Scripture and licensing

- Built-in text: King James Version (English) and Reina-Valera 1909 (Spanish), both public domain.
  Do not use the Reina-Valera 1960; it is copyrighted.
- Do not add the LSB, NASB, ESV or other copyrighted translations without written permission
  from the publisher. Users may paste their own text for personal use.
- Val wants the app free. Donations, if any, go through a link outside the app.

## Status (14 November 2026)

Done: web app with setup, Today, Calendar, Passages and Settings; two backgrounds; rest days;
iPhone project that opens in Xcode 27.

Not yet tested: the first Xcode build; reminders and snooze on a real iPhone; the Bible
lookup (English and Spanish) on the live GitHub page.

Ideas not built yet:

- Voice recording of a memorised verse, with one-tap share to a group chat (keep only the latest
  take per verse so phone storage stays small).
- Bundle the full King James text inside the app so it works offline.
- App icon; App Store listing; changing the pace of a plan after it has started.
