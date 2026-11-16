# Sword of the Word — notes for Claude

The app was first called Treasured. Val renamed it **Sword of the Word** (Ephesians 6:17 theme:
the Word of God as the believer's sword). The folder, the Xcode project and the GitHub repository
still use the old name "Treasured"; only what users see was renamed. Spanish name: Espada de la Palabra.

Read this first. It explains what this project is, how it is put together, and how Val likes to work.

## What it is

Sword of the Word is a Scripture memory app made by Val. It is free, with no accounts and no server.
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
- `assets/flow.jpg` — the background for look A. `assets/ribbon-today.jpg`, `ribbon-calendar.jpg`,
  `ribbon-passages.jpg`, `ribbon-settings.jpg` — one art piece per page for look B.
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

1. Replace every `'assets/….jpg'` path in the script with that image as a
   `data:image/jpeg;base64,...` string.
2. Add `, viewport-fit=cover` to the viewport meta tag.
3. Before `</style>` add safe-area padding:
   `main{padding-top:calc(28px + env(safe-area-inset-top))}`
   `.theme-b main{padding-top:calc(200px + env(safe-area-inset-top))}`
   `#tabs{bottom:calc(16px + env(safe-area-inset-bottom))}`

## Name, reminders and Grace mode

- The welcome screen asks for a first name or nickname (`S.name`, editable in Settings). It stays on
  the device.
- Reminders: `buildReminders()` makes the list for the next 21 days: one at the user's time for
  each day that has verses due, plus a nudge 45 minutes later for the next three days. Messages
  rotate from the `MSG` table (English and Spanish, sword and armour theme, `{n}` is the name);
  days three and beyond use the "come back" messages, because they only fire if the app has not
  been opened. The page sends the list with
  `window.webkit.messageHandlers.treasured.postMessage({lang, items})` on every save and whenever
  the app comes to the front; `TreasuredApp.swift` replaces all scheduled reminders with it, using
  the soft chime `chime.wav` and one Snooze button. Snooze opens the app, the app calls
  `window.showSnooze()` in the page, and the page shows hour and minute dials (0-12 h, 5-minute
  steps); the choice goes back as `{snooze: minutes, title, body}`. `S.snz` holds back today's
  other reminders until the snooze time. Today's reminders are left
  out once today's verses are done. `isNative()` tells whether the page is inside the iPhone app.
- Grace mode (`graceStart`, `applyGrace`): when someone falls behind, one button turns the missed
  days into rest days so today repeats the first unfinished day. Never make users "catch up".

## Pasting verses, hiding words, sounds

- Pasting (`parsePassage`, `readRef`): the user pastes a whole passage from any Bible app. A
  reference line pasted with the text (for example "Romans 8:28 LSB") is used as the reference and
  not treated as a verse. Verses are split by the verse numbers inside the text, or one per line,
  and each is named properly ("Romans 8:28"). A single verse may wrap over many lines. Never label
  verses "My verse": if there is no reference, ask for one.
- Do not build "connectors" that pull text out of YouVersion, Literal Word, Blue Letter Bible or
  similar. That does not get around copyright. Pasting for personal use is the route.
- Hide words uses an "invisible ink" look (`.ink`): the verse's own letters turn into shimmering,
  blurred gold and crimson dust (no box around it), and the verse name shows large and bold on top.
- Sounds (`sfx`): Val found button sounds annoying, so buttons, the bottom menu and the dials
  make no sound, only a light tap you can feel. The one sound is a soft three-note chime (the same
  notes as the reminder sound `chime.wav`) when a verse's repetitions for the day are finished,
  with a fourth higher note when everything for the day is done. One-repetition reviews do not
  chime individually. Settings has an off switch (`S.sound`, "Celebration chime").
- The motto shows on the welcome screen and at the top of Settings ("Our mission").

## Groups (built 15 November 2026, not yet tested against the real server)

People memorize a passage together and see who has finished each day.

- Server: Val's free Firebase project `sword-of-the-word` (Firestore in nam5, Anonymous sign-in on,
  free Spark plan). The app uses plain web requests, no Firebase library: `fbAuth()` signs each
  phone in anonymously once (invisible ID, kept in `S.fb`), `fbCall()` reads and writes Firestore.
  `enc`/`dec`/`encDoc`/`decDoc` convert values to Firestore's shape. The web config (`FB`) is public
  by design; the security rules are what protect the data.
- Data: `groups/{CODE}` holds the group name and a copy of the passage (title, verses, start, pace,
  offsets, owner). `groups/{CODE}/members/{uid}` holds one member: name, `behind`, and `days`
  (for each of the last 14 days: verses finished `d`, verses due `t`, repetitions `r`).
- Flow: one person picks one of their passages and a group name (`gcreate`) and gets a 6-letter
  code. Others enter the code (`gjoin`); the passage is added to their own calendar as a normal
  plan marked `plan.group`. `reportGroups()` sends checkmarks after every save (1.5 s delay) and
  when the app comes to the front. The Group tab (`viewGroup`) lists members: a check mark and the
  repetition count appear only when all of that day's repetitions are done. Someone whose rest
  days or Grace mode put them behind the group calendar shows "catching up".
- No phone numbers, emails or accounts. Invites are a text message with the code (`gshare`).
- Security rules to paste in Firebase (Firestore > Rules):

      rules_version = '2';
      service cloud.firestore {
        match /databases/{database}/documents {
          match /groups/{code} {
            allow get: if request.auth != null;
            allow list: if false;
            allow create: if request.auth != null && request.resource.data.owner == request.auth.uid;
            allow update, delete: if request.auth != null && resource.data.owner == request.auth.uid;
            match /members/{uid} {
              allow read: if request.auth != null;
              allow write: if request.auth != null && request.auth.uid == uid;
            }
          }
        }
      }

- Still to do for groups: test with two real phones; a privacy policy before any App Store
  release; the group passage text is copied to the server, so prefer public-domain text (KJV,
  RV1909) for groups or get permission; the web preview on claude.ai cannot reach the server
  (it blocks outside connections), so test on GitHub Pages or the iPhone.

## Design

- Two looks the user chooses between: A "Flowing gold" (red, white and gold liquid with frosted
  glass cards) and B "Silk ribbon" (crimson ribbon on white, solid white cards, no glitter).
- Colours: ink `#161413`, grey text `#5E5955`, crimson `#A4161A`, deep red `#7D0F13`,
  gold `#B8892B`. Gold is for lines and accents only; gold text on white is too faint to read.
- Fonts: Cormorant Garamond for verses and headings, DM Sans for everything else.
- Backgrounds drift slowly, and look A has gold glitter drawn on a canvas (`#glitter`, the
  `glitter…` functions). Look B is still: each page shows its own ribbon art piece (`bgFor()`
  picks it; welcome uses the Today piece and setup uses the Passages piece). Val found a moving
  ribbon looked artificial, so keep look B still. "Still mode" in Settings turns all motion off.
- The motto is Val's wording; keep it: "The Word of God is your Sword. Keep it sharp in your
  heart for life's daily battles. We're here to help you stay armed and faithful." The words
  "Word of God", "Sword", "armed" and "faithful" show in bold crimson (marked with *asterisks*
  in the text and turned into `<b class="hl">`). Val prefers colour over extra capital letters.
- All background images were generated for this project with Python (ribbons drawn along curves
  with brush texture, folds and shadows, 1170 x 2532), not taken from anywhere else. Val's taste:
  vivid, three-dimensional, like an art piece; crimson with small gold and black accents.

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

- A small knight mascot who peeks in from the side with encouragement and holiday greetings;
  occasional, with an off switch. Needs proper character art.
- Replace look B's ribbon art with glossy 3D crimson-liquid-and-sword artwork (Val has a Grok
  mockup she loves; she needs a clean version without the phone and buttons), and restyle the
  interface to show more background: title straight on the background, bolder lettering, smaller
  buttons, glassier cards. Apply the same to look A.
- Optional email sign-up for updates, as a link outside the app (no accounts or phone numbers in
  the app for now).
- "Time-sensitive" reminders need a capability added in Xcode; not turned on yet.
- Voice recording of a memorised verse, with one-tap share to a group chat (keep only the latest
  take per verse so phone storage stays small).
- Bundle the full King James text inside the app so it works offline.
- App icon; App Store listing; changing the pace of a plan after it has started.

## Where we left off (Nov 15, 2026, late evening)

Done and on Val's Mac (`~/Documents/Treasured`): everything described above, including Groups (Firebase) and a placeholder app icon (red background, upright sword; made by `icon.py`, lives in `ios/Treasured/Assets.xcassets/AppIcon.appiconset/icon-1024.png` and `UPLOAD-TO-GITHUB/assets/apple-touch-icon.png` + `icon-512.png`).

Not yet confirmed by Val:
- Groups against the real Firebase project. If the app says "Could not reach the group server", first re-check the API key in `app.js` (`FB.key`) character by character against Firebase console > Project settings > Your apps (it was typed from a photo), then the Firestore rules.
- Bible lookup (bible-api.com) on the live site.
- The app icon showing on the iPhone (needs `ASSETCATALOG_COMPILER_APPICON_NAME = AppIcon` in the project build settings; added Nov 15).

Next steps, in order:
1. GitHub is behind. Upload everything inside `UPLOAD-TO-GITHUB` to the repo top level (drag the `assets` folder itself so the pictures land in `assets/`). Delete the loose `flow.jpg` / `ribbon.jpg` at the repo top level. Then the tablet can use https://val620-cyber.github.io/Treasured/ and join a group by code.
2. Look B redesign: Val wants the ribbon art replaced by glossy 3D crimson-liquid + sword art (like a Grok mockup she has), plus a restyle for BOTH looks: title straight on the background, bolder lettering, smaller buttons, glassier cards. Waiting on a clean image from her (no phone frame, no text, tall portrait). Do it in one pass. The final app icon should match that art.
3. App Store (only when Val decides to pay Apple's $99/year): enroll in the Apple Developer Program, switch the Xcode team to the paid one, write a short privacy policy page (host it in this repo; say that names and group progress are stored in Firebase and nothing is sold), take screenshots, Product > Archive > upload, use TestFlight for friends first. After that every iPhone change needs a new upload and Apple review; the web version still updates instantly.
4. Ideas still parked: knight mascot, optional email sign-up link, time-sensitive notifications.

### Added late Nov 15
- Bottom menu: "Passages" is now "Personal" (tab id is still `passages`). Group plans show there with a "Group passage" label.
- Group tab > Start a group: type a group name, then "Choose the passage and schedule" opens the same wizard in group mode (`UI.wiz.group` = group name; 4 steps, the reminder step is skipped; last button is "Create group", which calls `createGroup(name, plan)`). "Or use a passage you already have" keeps the old path. Reminder times are always each person's own (`S.reminders`); the group only shares passage, start date and days.
- `APP-STORE.md` (all App Store text, privacy answers, reviewer notes) and `privacy.html` (privacy policy; fill in `[DATE]` and `[YOUR CONTACT EMAIL]`) are in this folder, ready to paste.
- Val worries the sword + red might look violent to Apple. Keep the art reading as crimson silk/liquid light: no drips, splatter or wounds, and never call it blood in store text.
- Group cards show "Created <date>" (or "Joined <date>" for groups made before this was stored). "Delete group" (action `gdel`) deletes the group document for everyone; Firestore only lets the owner do it, so a 403 shows "Only the person who started this group can delete it" and hides the button (`g.owner=false`). Member documents of other people are left behind in Firestore (harmless; clean up in the Firebase console if ever needed). Other members' apps still list a deleted group until they leave it.
- GitHub pushes from Claude are still refused (403) until Val installs the Claude GitHub App on `val620-cyber/Treasured` (https://github.com/apps/claude/installations/select_target). Once she has: clone the repo to a fresh folder, copy `UPLOAD-TO-GITHUB/*` in, commit, push to `main`. Until then she uploads by hand.

### Nov 16 morning
- Look B is now "Sword and shield": three images Val made in Grok (`assets/armor-sword.jpg`, `armor-shield.jpg`, `armor-helmet.jpg`, 784x1168). Today + Settings = sword, Calendar + Group = shield, Personal + setup = helmet. The old ribbon art, `art.py` output and `ribbon-*.jpg` are retired; ignore earlier mentions of them. Look B cards and the bottom menu are now frosted glass so the art shows through.
- The app icon is now a square crop of the sword image (replaces the placeholder drawn by `icon.py`).
- Still to do from Val's wish list: the fuller restyle for both looks (title straight on the background, bolder lettering, smaller buttons).
- Later Nov 16: added `assets/armor-dagger.jpg` (Val: the biblical "sword" is closer to a dagger, so this is the main image). Look B pages now: Today + Settings = dagger, Group = sword, Calendar = shield, Personal + setup = helmet. Look B restyle done per her Grok mockup: no empty band at the top (the old 200px top padding is gone, also removed from `build.py`), page titles float on the background in bold sans (`.card.hero`, look B only), smaller buttons, more transparent glass cards with bolder text. Look A (Flowing gold) is unchanged; give it the same treatment only if she asks. Preview tool: `/tmp/shot.py`-style Playwright script with Chrome at /usr/bin/google-chrome works in the sandbox for screenshots.
- Nov 16, second design pass (look B only): cards are now very light glass (white .26, 5px blur); verse practice cards have no box at all (`.card.task`): the label and reference are small glass pills, each line of verse text has its own snug glass strip (`span.vl` with box-decoration-break:clone), and the counter row is a glass strip (`.prow`). Grace mode notes float with no box (`.card.bare`). Page titles use the Cinzel font (loaded from Google Fonts). "Switch background" and "Snooze reminder" were removed from Today; background lives in Settings > Background and Snooze is a button under Settings > Reminder times (iPhone only). `main` now has a full screen of bottom padding so the user can scroll everything up and look at the art (both looks).
- Val tried the Cinzel title font and did not like it. Titles are back to bold DM Sans (800, 36px). Do not change fonts again unless she asks.
- Val wants ALL lettering kept as it was on the morning of Nov 16: Cormorant Garamond for verses and card headings, DM Sans for everything else, same sizes and weights. Only boxes, glass and layout may change.
- App icon: Val tried a dagger image and went back to the square crop of the sword image. Keep the sword crop.
- Nov 16, 7:20 AM changes: (1) the gold shimmer now floats on both looks (deeper gold on Sword and shield so it shows on the bright art; Still mode turns it off). (2) All in-app sound is gone: no chime for finishing a verse or the day, and the "Celebration chime" setting was removed. Finishing a verse gives a vibration only; finishing the whole day shows a small "Well done, <name>!" pop-up (`showCheer`/`viewCheer`, three random encouragements, closes itself after 4 seconds or on tap). The reminder notification sound (chime.wav) stays; Val likes it. Ignore older notes above about the celebration chime and `bell()`. (3) On Today the verse cards and the Grace mode card are boxed glass cards again (Val: without the box "things fell all over the place"); the per-line strips are gone. Keep the current light glass level (white .26, 5px blur). (4) Notifications use the app icon automatically; a blank icon on a notification means the phone had an older build or a cached icon.
- Fonts, final word from Val (Nov 16, 7:20 AM): every big title, including the floating page titles and the Well done pop-up, uses Cormorant Garamond 600 (the app's normal h1). The bold DM Sans titles are retired. Sandbox screenshots cannot load Google Fonts and show Times instead; warn her before sending one.
- Gold dust on Sword and shield: plain gold specks with no halo, and none in the top 30% of the screen (it fades in between 30% and 42% down) so the title area stays clean. Flowing gold keeps its original glitter with halo.
- Nov 16, 7:40 AM: the Flowing gold look is REMOVED at Val's request (distracting, hard to explain, not worth credits). The app has one look, Sword and shield: `body` is always `theme-b`, `flow.jpg`, `IMG_A`, `themePicker` and the `theme` action are gone, Welcome no longer asks for a look, and Settings > Background only has Still mode (turns the gold dust off). Ignore every earlier mention of look A / two looks; the leftover `.theme-a` CSS lines are dead and can be deleted. The Español button is gone from Today: language is picked from the phone's language on first open and can be changed in Settings > Language (and on the Welcome screen).
- Nov 16, 8:00 AM: Val installed the Claude GitHub App on this repo, so Claude can push now (earlier notes saying pushes are refused are out of date). The repo was brought fully up to date; pictures live in `assets/`. To publish a change: clone the repo to a fresh folder, copy everything from `UPLOAD-TO-GITHUB` in, commit, push to `main`. Claude's sandbox cannot open the github.io site, so ask Val to confirm the live page after a push.
