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
- Well done pop-up restyled after a Grok mockup Val loves: no dark overlay, cream card with a glowing gold border, and two layers of slowly turning light rays behind it (`.cheerbg::before/::after`, conic gradients) plus a breathing glow. Still mode stops the motion.
- Double-tap zoom is turned off everywhere (`touch-action:manipulation` on every element) because fast rep tapping kept zooming the page. Pinch zoom is left on for people who need larger text. Buttons are not text-selectable, so a long press does not pop up the copy menu.
- Tablets and computers: on screens wider than 4:5 the tall background picture is shown whole and centred (`object-fit:contain`) with a blurred copy (`#bgfill`) filling the sides. Phones are unchanged (picture fills the screen).
- Nov 16 evening: (1) First-time tour: six pop-up cards (`TOUR`, `viewTour`, actions `tourstart/tournext/tourback/tourskip`), each lighting up one bottom-menu item (`.tourhl`) and showing that page behind it. It starts right after the Welcome screen, and once for existing users (`S.tour` is set to 'done' afterwards). The last card's button says "Add my first passage" and opens the setup steps when the user has no passage yet. Settings > About has "Show the tour again". (2) Bottom menu order is now Today, Personal, Calendar, Group, Settings. (3) App Store link: `APP_STORE_URL` in app.js is empty for now. When the app is live, paste its link there (https://apps.apple.com/app/id...); group invites and the new Settings > About > "Share the app" button then include it next to the web link.
- Each unfinished verse card on Today has its own "Snooze reminder" button (iPhone app only, since the web version has no reminders). It opens the same dials; the snoozed notification then names that verse ("Romans 8:28 is waiting for you."). The button in Settings > Reminder times stays as the general one.
- Nov 16, 9 PM (App Store prep): (1) Paste without verse numbers now works: `alignSplit(text, refs, n)` compares the pasted text with the built-in KJV/RV1909 verses (fetched from bible-api) and cuts at punctuation where each verse best matches; without network it makes an even best-guess split. `parsePassage` also drops URL lines and the outer quotation marks that Bible apps add. (2) Settings > About > "Delete my group data" (`wipe` action) deletes member docs, owned groups and the anonymous Firebase user; needed for Apple. (3) `privacy.html` rewritten and `support.html` added (live on GitHub Pages). Val has not given a public support email yet; the support page points to the repo's Issues. (4) `APP-STORE.md` was replaced by `APP_STORE_LISTING.md`, and `HANDOFF.md` (submission steps and open questions) plus `screenshots/` were added; these three live in `~/Documents/Treasured`, not on GitHub. (5) iOS project: added `PrivacyInfo.xcprivacy` and `INFOPLIST_KEY_ITSAppUsesNonExemptEncryption = NO`. Release archive and upload are still to do and need Val's paid Apple Developer membership.
- Nov 16, 9:30 PM: (1) Setup step 1: Continue is no longer greyed out. With nothing in the list yet it adds whatever is filled in (lookup or paste) and then moves on by itself (`wnext` calls `wfetch`/`wpaste` with auto=true). Val and her mother both got stuck there because they did not tap "Add these verses" first. (2) "Paste my own" uses the same Book / Chapter / From / To pickers as the King James option instead of a typed reference; `wpaste` builds `W.ref` from them. (3) Group tab no longer offers "Or use a passage you already have"; a group passage is always chosen from scratch (the `gcreate` action is left in the code, unused).
- Today page: when a verse's reps are finished its button turns gold (`.btn.gold`) for about 2 seconds, then the card leaves Today for the rest of the day. A small line "Finished today: N verses" with Show/Hide (`UI.showDone`) brings finished cards back, so Undo last tap is still reachable.
- Comfort print (Nov 16, 9:50 PM): Val's mother (50s) could not read the small text, and her grandmother will use the app too. A block at the end of the stylesheet enlarges and darkens all small text (labels, dates, notes, menu labels, small buttons, calendar weekday names and legend). Typefaces and the glass level are unchanged. This replaces the earlier "keep sizes as they were" note: do not shrink text again.
- The Show/Hide button beside "Finished today" is gold (`btn gold small`), as a small reward.
- Buttons: Val tried strong "puffy" 3D buttons and found them childish and distracting from the artwork. Final style: the original shapes and outlines with a soft top-to-bottom shade and a light shadow only (block at the end of the stylesheet). Do not make them more 3D. The Show button stays gold. The text-only Undo link uses `.btn.flat`.
- Invites (Nov 16, 10:15 PM): each group card has a wide "Invite friends" button with the code under it. The invite message now carries a link to `join.html?code=XXXXXX` (new page on GitHub Pages): it shows the code, a "Join in your browser" button (opens the web app with `?join=CODE`, which lands on Group with the code filled in), and a "Get the iPhone app" button that appears once `APP_STORE_URL` is filled in at the top of join.html. The tour's Group card explains inviting. NOT built yet (Val asked; deferred so tonight's build is not put at risk): the link opening the iPhone app directly and joining. That needs a native change (a custom URL scheme or Universal Links via Associated Domains, plus `.onOpenURL` in TreasuredApp.swift passing the code to the page) and a real-device test; do it as the first update after approval.
- Personal page decluttered (Nov 16, 10:40 PM): the Grace mode explanation no longer sits on the page; tapping a passage's Grace mode button opens a small pop-up with that explanation and Cancel / "Pick up where I left off" (`graceask`, `graceno`). The "N of M verses started" text was removed from passage cards. Val wants fewer words on screen.
- Nov 16, 10:50 PM: bundle ID changed to `com.valcortez.swordoftheword` before the first App Store upload (permanent from then on). The Xcode project, target and folder are still named "Treasured" on purpose; that name is only visible inside Xcode. Renaming them is optional cosmetic work for a calm day, with a build test afterwards.
- Nov 16, 11:25 PM: the project folder on Val's Mac was renamed from the old name to `~/Documents/Treasured`. `ios/Treasured.xcodeproj`, its `Treasured` source folder, the GitHub repo name and the github.io address keep the old name on purpose (renaming them risks the build and would break the links inside the app). Val follows instructions best from pictures of her own screen with red circles, arrows and numbered labels; see `guide-pictures/` and the top of HANDOFF.md. Membership: she paid for the Apple Developer Program at 11:20 PM; waiting for Apple to activate it before the archive and upload (see "Where things stand" in HANDOFF.md).
- Nov 16, 11:30 PM: the folder rename was undone at Val's request. The project folder is `~/Documents/Treasured` again; ignore the 11:25 PM note about a renamed folder. The picture-guidance note and `guide-pictures/` stay.
- Nov 18 (Opus): American Standard Version (1901, public domain, from eBible.org eng-asv via the BibleNLP/ebible GitHub mirror) added as the default English Bible. Data: `assets/asv.js` (web) and `ios/Treasured/asv.js` (iPhone; `build.py` copies it and rewrites the path), `window.ASV_DATA[BOOK_ID][chapter-1][verse-1]`, loaded lazily by `loadASV()`; lookups via `asvRange()`, tag ' · ASV'. Wizard source chips: ASV (default, English only), KJV/RV1909 (bible-api), Paste. Spanish users still default to RV1909. Pasted text without verse numbers is aligned against the ASV in English (offline). Settings > About starts with the credit line "Scripture: American Standard Version (1901), public domain." Tests: /tmp/asvt.js, /tmp/asves.js.
- Nov 18: the translation buttons in setup step 1 became one "Translation" picker (`<select id="w_src">`, handled in the document change listener). English: American Standard Version (ASV, default), King James Version (KJV), Paste from my own Bible. Spanish: Reina-Valera 1909 (default), Paste. Val plans to add more versions: add an option to that list, plus a lookup branch in `wfetch` (and the credit line in Settings > About).
- The Translation picker (`#w_src`) is styled as a red pill with a white arrow, like the old red version button; Val likes the colour it adds.
- Setup step 1 "Your list" shows every chosen verse in full (reference and translation on one line, the whole verse below in the verse font), all in one card, so Val can see how verses read together (e.g. Romans 5:8 then 2 Corinthians 5:21) before choosing.
- Nov 18: the two fonts (Cormorant Garamond, DM Sans; SIL Open Font License, Latin subset) are now bundled: `assets/cormorant.ttf`, `assets/dmsans.ttf` on the web and copied to `ios/Treasured/` by build.py. The Google Fonts link was removed, so the lettering is identical on every phone and works offline (and the app no longer contacts Google Fonts).
- Nov 18, offline review (Opus): the King James (eBible eng-kjv2006, paragraph marks removed) and the Reina-Valera 1909 (eBible spaRV1909) are now built in too: `assets/kjv.js`, `assets/rv1909.js` (copied to ios/Treasured by build.py). `loadBible(code)` / `bibleRange(code,...)` serve all three; bible-api.com is no longer used anywhere. Offline behaviour, tested with the iPhone copy in an offline browser: fonts, pictures, all three Bibles, paste splitting (ASV in English, RV1909 in Spanish), practice and reminders all work. Groups need internet: starting or joining offline is stopped up front with a clear message; the Group page explains that checkmarks will be shared once back online (`online` event and app reopen trigger `queueReport`); "Delete my group data" no longer reports success when offline and keeps the data until it can really delete it. The website itself still needs internet to open (it is not an offline web app); the iPhone app is the offline one.
- Nov 18, verse-number fix (Opus): the BibleNLP corpus stores every Bible in Hebrew-style (org) verse numbers, which shifted most titled Psalms by 1-2 verses and moved Malachi 4, Joel 2:28-3:21, Numbers 16:36-50 and others. ASV and KJV are now rebuilt to English numbers with eBible's `metadata/eng.vrs` mapping (script /tmp/remap.py), plus two repairs: Acts 19:41 split back out of 19:40, and 2 Corinthians 13:14 added (missing in the corpus; wording checked against bible.com). Check: every ASV verse was compared with its KJV verse; the only low matches (9) are genuine translation differences (e.g. ASV swaps Philippians 1:16/17, omits the 1 John 5:7 Trinity clause). Verses the ASV leaves out on purpose (Matt 17:21, Acts 8:37, etc.) are simply missing. RV1909: kept only the 1,141 chapters that line up exactly with English numbers; the other 48 are `null` in rv1909.js and fetched from bible-api.com when online (privacy page says so). Psalm titles merged into the Spanish verse 1 were removed (125 psalms). 2 Corinthians 5:21 in the ASV really reads "Him who knew no sin he made to be sin on our behalf..."; nothing is cut off.
- Nov 18: the 48 Spanish chapters that are not built in are saved on the phone (localStorage key `treasured.ch.rv1909.<BOOK>.<chapter>`) after the first online lookup, so they work offline afterwards. Val asked for a built-in "memory file" for them instead; that was not done because the eBible Spanish file is damaged in exactly those chapters (verses missing or joined), the conversion table cannot repair that, and bible-api.com is not reachable from Claude's tools. To build them in later: from a machine that can reach bible-api.com, fetch /data/rv1909/<BOOK>/<chapter> for the chapters stored as null in rv1909.js and write the verses in.
- Nov 18: tapping a reminder now opens Today's practice. Swift: `Reminders.pendingToday` is set for `UNNotificationDefaultActionIdentifier` in AppDelegate's didReceive, and `deliverSnooze()` calls `window.openToday()` once the page is ready (works when the app was closed too). Page: `window.openToday` closes setup/snooze/pop-ups and shows Today (skipped while the first-time tour is open). Not compiled by Claude (no Swift compiler in its workspace); first Xcode build after Nov 18 verified it.

- Nov 22: The setup list ("Your list") never shows the same verse twice. addUnique() in app.js skips any verse whose reference is already on the list (fetch and paste both), and the message names the skipped ones.
