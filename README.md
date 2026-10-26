# Q4 2026 Engagement Event deck

A 6-slide deck built from `Q4_Engagement_Event_Deck.docx`, themed as a friendly game night: bowling-alley teal with cute pins on one side, olive camo with crosshairs on the other. **Voting is built into the slides**: there is no QR code and no separate voting page. Share one link; people open it, read the options, and vote on slides 4 and 5.

| Slide | Content |
| --- | --- |
| 1 | Title |
| 2 | Option 1: Bowling Games |
| 3 | Option 2: War Games |
| 4 | Poll: event option (type your name, click an option) |
| 5 | Poll: time option |
| 6 | Polling Result: a pie chart per question, plus every voter and their choices |

## Voting deadline

Voting closes automatically on **30 October 2026 at 23:59 (UTC+7)**, which is 16:59 UTC.

- Before then, every poll slide shows when voting closes and a live countdown. When the time passes, the cards lock by themselves, even on screens that were left open, and the results slide says these are the final results.
- The deadline is also enforced by the database rules below, so it cannot be bypassed with an old browser tab or a modified page.
- The organizer can still close voting early with **Close voting**. After the deadline **Reopen voting** is disabled.
- Times are shown in UTC+7 to everyone, whatever their own time zone.
- The deadline is exactly 23:59:00. Anyone voting at 23:59:30 is too late.

**To change the deadline**, edit two places:
1. In `index.html`, `CONFIG.votingCloses` (an ISO date with offset, for example `'2026-11-02T18:00:00+07:00'`).
2. In the database rules, the number after `now <` on the `ballots` line. Get it by pasting the same date into your browser console: `Date.parse('2026-11-02T18:00:00+07:00')`.

Set `votingCloses` to `''` for no deadline (and remove the `now <` condition from the rules).

## How voting works

1. A person opens the deck link on their own phone or laptop.
2. On slide 4 they type their **name** and click an option. On slide 5 they click a time option. Each click saves their vote immediately.
3. They can change any vote until voting is closed. The name field is shared between slides 4 and 5.
4. Slide 6 shows live pie charts and the list of voters with their choices. It updates every couple of seconds.

On phones and narrow windows the deck becomes one scrolling page (no slide-by-slide navigation), so the polls are easy to use on a small screen. On laptops and projectors it is a normal 16:9 deck (`←` `→` / `Space`, `F` full screen, `T` light/dark).

## The motion

- **Title slide:** the pins drop in one by one and their eyes follow your cursor. Click a pin and it tumbles.
- **Slide transitions:** slides glide in from the direction you are moving; lists and stickers pop in.
- **Voting:** each card reacts when you pick it. Bowling rolls a ball into falling pins, War Games locks a crosshair on and splats, dates burst into confetti. The status line cheers to match ("Strike!", "Bullseye!", "Locked in!").
- **Results:** the pies sweep in, the vote counter bumps when a number changes, and each new voter pops into the list.
- All of it switches off automatically for people who have "reduce motion" turned on in their device settings.

## Links

| URL | For | What it does |
| --- | --- | --- |
| `…/index.html` | Everyone | The deck, with voting and live results. This is the link you share. |
| `…/index.html?mode=present` | You (organizer) | Same deck plus admin controls at the bottom (move the mouse to show them): close or reopen voting (`V`), reset all votes, download a CSV (on slide 6). |

## Try it locally (no setup)

Open `index.html`. Without a database it runs in demo mode: votes stay in your browser. Open `index.html?mode=present#6` and use **Add sample votes** to preview the results slide.

## Deploy to GitHub Pages

1. Create a GitHub repository and upload `index.html` (and this README).
2. **Settings → Pages → Build and deployment → Deploy from a branch**, choose `main` and `/ (root)`, then **Save**.
3. Your deck is live at `https://<your-user>.github.io/<repo>/` after a minute. That address is the link to send to everyone.

## Turn on live voting (Firebase Realtime Database, free tier)

GitHub Pages only serves static files, so votes need a place to live. The deck talks to Firebase over plain REST calls: no SDK and no keys in your code.

1. In the [Firebase console](https://console.firebase.google.com), create a project, then **Build → Realtime Database → Create database** (start in locked mode).
2. Open the **Rules** tab, paste the rules below, and **Publish**.
3. On the **Data** tab, copy the database URL (like `https://<project>-default-rtdb.firebaseio.com` or `https://<project>-default-rtdb.<region>.firebasedatabase.app`).
4. In `index.html`, find `CONFIG`, paste the URL into `firebaseUrl`, and commit.

```json
{
  "rules": {
    ".read": false,
    ".write": false,
    "decks": {
      "$deck": {
        ".read": true,
        "session": {
          ".write": true,
          ".validate": "newData.hasChildren(['open'])",
          "open": { ".validate": "newData.isBoolean()" },
          "$other": { ".validate": false }
        },
        "ballots": {
          ".write": "now < 1793379540000",
          "$voter": {
            ".validate": "newData.hasChildren(['name', 'answers'])",
            "name": { ".validate": "newData.isString() && newData.val().length > 0 && newData.val().length <= 40" },
            "answers": {
              "$q": { ".validate": "newData.isNumber() && newData.val() >= 0 && newData.val() <= 3" }
            },
            "ts": { ".validate": "newData.isNumber()" },
            "$other": { ".validate": false }
          }
        }
      }
    }
  }
}
```

## Customize

- **Slides 1 to 3:** edit the text inside the `<section>` blocks.
- **Questions and options:** edit `QUESTIONS` near the bottom of `index.html` (up to 4 options each). `short` is the small label on the voter chips on slide 6. To add a question, add it to `QUESTIONS` and copy a poll slide with `data-q="<its id>"`; the results slide picks it up automatically.
- **Per-option look:** in `QUESTIONS`, each option has `color` (`teal`, `orange`, `sun`, `lilac`), `icon` (`pin`, `target`, `cal`, `splat`), `fx` (the vote effect: `bowl`, `blast`, `confetti`) and `cheer` (the message after voting). Colors carry through to the cards, pies, legend and voter chips.
- **Palette and fonts:** the tokens are at the top of the `<style>` block. Fonts (Bowlby One, Rubik, Saira Stencil One, all open source under the SIL Open Font License) are embedded in the file.

## Good to know

- **Names are public.** They appear on slide 6 for everyone who has the link. The poll slides say so.
- **Everyone who has the link can see the results**, including while voting is still open. If you want a reveal moment, close voting before sharing the link widely, or share the link only when you are ready.
- **One vote per device is an honor system.** Each browser gets a random ID. Someone who clears their browser data (or uses a second device) can vote again or use a different name. For strict control, add sign-in (e.g. Firebase Auth) and tighten the rules.
- **`?mode=present` is not a password**, and the rules above let anyone write ballots and open or close voting. That is fine for an internal event; for anything sensitive, add authentication.
- **A reset also clears people's selections.** After you reset, their screens notice within a few seconds and let them vote again.
- **After the deadline the rules block all writes to ballots**, including **Reset all votes**. If you need to clear the votes later, delete the `ballots` node in the Firebase console.
- **The countdown uses each device's own clock**, while the database rules use the database's clock. If someone's device clock is wrong, the page may look open or closed a little off, but the database decides: a late vote is refused and the page tells them.
- **Free-tier limits.** Firebase's free plan limits simultaneous connections and monthly downloads. Check Firebase's current pricing page for large audiences.
- **Network:** the page makes no external requests except to your Firebase database. Fonts, illustrations and animations are all inside the single HTML file (about 170 KB).
- **Emoji** (🎳 💥 📍) are drawn by each device's own emoji font, so they look slightly different on different phones.
