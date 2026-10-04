# Kinza Padel Live (Firebase) 

Live scores, schedule and group standings for the Kinza Padel Championship at Ace Padel Club, Damascus.
Firebase project: `padeltournament-ed489`.

## Files

| File | What it is |
|---|---|
| `index.html` | The public page. Anyone can watch; scorekeepers sign in to enter points. |
| `import.html` | One-time page that loads `data.json` into Firestore. Delete it after use. |
| `data.json` | Days 1–4 schedule (111 matches) and all 20 groups. |
| `manifest.json`, `sw.js`, `icons/` | Installable app: home-screen icon, full-screen launch, faster repeat loads. Bump `VERSION` in `sw.js` after every upload. |
| `firestore.rules` | Security rules: everyone reads; only usernames in the `keepers` collection write. |

## Setup (about 10 minutes)

1. **Firestore:** done. (Database created in production mode.)
2. **Turn on sign-in.** Build → Authentication → Sign-in method → Email/Password → Enable.
3. **Block self sign-up.** Authentication → Settings → User actions → untick **Enable create (sign-up)** → Save.
   Only you can create accounts, from the console.
4. **Create the scorekeeper logins.** Authentication → Users → Add user, once per person (yourself first).
   For username `ahmad`, the email is **`ahmad@kinzapadel.com`** and you choose the password.
   The address never receives mail; it's only a login ID. Scorekeepers type just `ahmad` on the site.
5. **Approve them.** Firestore Database → Data → Start collection → ID `keepers`.
   Add one document per person: **Document ID = the username in lowercase** (`ahmad`), with one field, for example `name` = their full name.
6. **Publish the rules.** Firestore Database → Rules → paste all of `firestore.rules` → Publish.
7. **Allow the site's address.** Authentication → Settings → Authorized domains → add it (for example `kinzapadel.github.io`).
   Not needed if you host on Firebase Hosting (`padeltournament-ed489.web.app`).
8. **Put the files online.** GitHub organization repo named `<org>.github.io`, or Firebase Hosting.
9. **Import the data.** Open `import.html` on the live site, sign in, press Import. It skips matches that already exist unless you tick Overwrite, so it never wipes live scores.
10. **Delete `import.html`** from the site.

## Who can do what

| | Visitors (no login) | Scorekeepers |
|---|---|---|
| See schedule, live scores, standings | Yes | Yes |
| Score points live, undo, mark final | No | Yes |
| Add, edit or delete matches (teams, time, court, format, scores) | No | Yes |
| Add, edit or delete groups | No | Yes |

Scorekeepers press **Scorekeeper sign in**, then:
- **Matches tab:** tap any match to score it or edit its details. "Add a match" is at the bottom of each day.
- **Standings tab:** "Edit group" next to each group, "Add a group" at the bottom.

## Adding or removing a scorekeeper

- **Add:** Authentication → Add user `username@kinzapadel.com` with a password, then add `keepers/username` in Firestore. Give them the username and password.
- **Remove:** delete their `keepers` document. They lose edit rights right away, even if still signed in. Disable or delete the login too.
- **Forgotten password:** reset emails can't be delivered to these addresses. Delete the user in Authentication and add them again with the same username and a new password; their `keepers` document stays as it is.

## Data model

- `matches/{id}`: `day`, `cat` (B/C/D), `court`, `time` ("17:00"; "24:00" means midnight), `t1`, `t2`, `group` (optional override),
  `bo`, `games`, `tbAt`, `deuce` (`adv1` = one advantage then golden point, `golden`, `adv`), `status` (scheduled/live/final),
  `s` (sets as `[{a,b}]`), `g` (current game points `{a,b}`), `winner` (0/1/null), `hist` (undo stack), `t` (last update, ms).
- `groups/{cat-name}`: `cat`, `name`, `teams` (array of "Player + Player").
- `keepers/{username}`: one document per approved scorekeeper (lowercase username as ID). Not readable from the page.
