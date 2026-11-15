# Treasured

A Scripture memory app. A new verse is said 25, 20, 15 and 10 times over its first four days,
then once a day for 45 days, once a week for 7 weeks, and on the first of each month after that.

## What is in this folder

- `UPLOAD-TO-GITHUB/` is everything that goes on GitHub: the app itself (`index.html` plus the
  `assets` folder with two background images) and two notes files (`README.md`, `CLAUDE.md`).
  Upload all four items; every file in it is the latest version.
- `ios/` is the iPhone version. It stays on the Mac and is not uploaded to GitHub. Open `ios/Treasured.xcodeproj` in Xcode and press the Play button.
  - `ios/Treasured/index.html` is the same page as `UPLOAD-TO-GITHUB/index.html` with the two images built in.
  - `ios/Treasured/TreasuredApp.swift` shows that page and adds daily reminders with
    snooze buttons (1, 2 or 3 hours).

## Changing the app later

Edit `UPLOAD-TO-GITHUB/index.html`. Then copy it over `ios/Treasured/index.html`, replacing the two image
paths on the line that starts `var IMG_A=` with the images as base64 data, and adding
`viewport-fit=cover` to the viewport line (compare the two files to see the three differences).

## Putting it in the App Store (when ready)

1. Join the Apple Developer Program ($99 a year) at developer.apple.com.
2. In Xcode, click the blue Treasured icon at the top of the left sidebar, then "Signing & Capabilities",
   and choose your team.
3. Add an app icon (1024 x 1024 image) in an asset catalog.
4. Menu: Product > Archive, then "Distribute App" and follow the prompts.
5. In App Store Connect, fill in the description and screenshots, then submit for review.

## Notes

- Scripture text: King James Version (public domain), looked up from bible-api.com when the user
  adds verses, plus anything the user pastes in themselves.
- All passages and progress are stored on the device only. There is no server and no account.
- The reminders and snooze code has not yet been tested on a real iPhone.
