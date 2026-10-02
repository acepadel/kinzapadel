# Kinza Padel Live (Firebase)

Live scores, schedule and group standings for the Kinza Padel Championship at Ace Padel Club, Damascus.
Firebase project: `padeltournament-ed489`.

## Files

| File | What it is |
|---|---|
| `index.html` | The public page. Anyone can watch; scorekeepers sign in to enter points. |
| `import.html` | One-time page that loads `data.json` into Firestore. Delete it after use. |
| `data.json` | Days 1–4 schedule (111 matches) and all 20 groups. |
| `firestore.rules` | Security rules: everyone reads; only people in the `keepers` collection write. |

## Setup (about 10 minutes)

1. **Firestore:** done. (Database created in production mode.)
2. **Turn on sign-in.** Build → Authentication → Sign-in method → Email/Password → Enable.
3. **Create the accounts.** Authentication → Users → Add user, once per person who may edit (yourself first).
4. **Approve them as scorekeepers.** Firestore Database → Data → Start collection → ID `keepers`.
   Add one document per person: **Document ID = their email in lowercase** (for example `name@gmail.com`).
   The document needs one field; anything works, for example `name` (string) = their name.
5. **Publish the rules.** Firestore Database → Rules → paste all of `firestore.rules` → Publish.
6. **Allow your domain.** Authentication → Settings → Authorized domains → add the site's domain (for example `taweltysy.com`).
7. **Put the files online.** Push this folder to a GitHub repo with Pages on (a repo named `padel` appears as `taweltysy.com/padel`).
8. **Import the data.** Open `import.html` on the live site, sign in, press Import. It skips matches that already exist unless you tick Overwrite, so it never wipes live scores.
9. **Delete `import.html`** from the repo.

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

- **Add:** create the user in Authentication, then add a document with their lowercase email as the ID in `keepers`.
- **Remove:** delete their document from `keepers`. They lose edit rights right away, even if they are still signed in.

## Data model

- `matches/{id}`: `day`, `cat` (B/C/D), `court`, `time` ("17:00"; "24:00" means midnight), `t1`, `t2`, `group` (optional override),
  `bo`, `games`, `tbAt`, `deuce` (`adv1` = one advantage then golden point, `golden`, `adv`), `status` (scheduled/live/final),
  `s` (sets as `[{a,b}]`), `g` (current game points `{a,b}`), `winner` (0/1/null), `hist` (undo stack), `t` (last update, ms).
- `groups/{cat-name}`: `cat`, `name`, `teams` (array of "Player + Player").
- `keepers/{email}`: one document per approved scorekeeper (lowercase email as ID). Not readable from the page.
