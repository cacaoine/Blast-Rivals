# Blast-Rivals

A 3D browser deathmatch shooter inspired by Roblox RIVALS. Built with Three.js in a single `index.html` — no build step.

## Play

Open `index.html` in a browser (or serve the folder with any static server). First to 10 eliminations wins the round.

- **Bombs 💣** are the currency: earn 30 per elimination, 45 per headshot, +150 for winning a round.
- **Armory**: 27 weapons from the free Blaster up to the Railgun — higher damage costs more bombs. Hold to fire. Every weapon has a **Test Fire** button for a free practice session (no purchases, no rewards).
- **Cosmetics**: 35 per-gun skins and 26 universal wraps (Gold Plate, Galaxy, Hologram, Emberglass…).
- **Maps**: Arena, Docks, Splash (ride the slides!), Construction, Neon District, Glacier, Temple.
- **Modes**: Solo vs Bots, or **online Multiplayer** — create a room, share its 4-letter code (or Quick match) and play live against people on any computer, anywhere.
- **Unlimited levels**: eliminations and round wins award XP; your level raises bot difficulty and adds rivals over time.
- **Monthly Golden Case Day**: one day a month the lobby runs 25 hard tasks; every five completed tasks earns a case (max 5) with all contents and exact odds shown up front.

Controls: WASD / arrow keys move, Space jump, Shift sprint, mouse aim/fire, R reload, 1–7 swap weapons, Esc pause.

## Accounts — save progress and log in from any computer

The game picks a save mode automatically:

| Where the game runs | How progress is saved |
|---|---|
| Any site with Firebase configured (recommended) | **Username + password login**, progress in a Firestore database, live-synced across devices |
| The published Claude artifact | Account code (`BR-XXXX-XXXX`) in the artifact's built-in database |
| Plain `index.html` with no Firebase config | Local account (username + SHA-256-hashed password) on this device only, plus Download/Load JSON backup |

### Set up Firebase (about 5 minutes, free)

1. Go to <https://console.firebase.google.com>, **Add project**, name it (e.g. `blast-rivals-game`). Google Analytics can be off.
2. **Build → Authentication → Get started → Sign-in method → Email/Password → Enable → Save.**
   (Players type a username; the game turns it into `username@players.blastrivals.app` behind the scenes.)
3. **Build → Firestore Database → Create database → Start in production mode** → pick a region → Enable.
4. In Firestore open the **Rules** tab, replace everything with the contents of [`firestore.rules`](firestore.rules), and **Publish**. This makes every player's save private to them.
5. **Project settings (gear) → Your apps → Web (`</>`) → register the app** → copy the `firebaseConfig` object.
6. Paste it into [`firebase-config.js`](firebase-config.js) as `window.BLAST_FIREBASE_CONFIG = { ... };` and commit.
7. **Authentication → Settings → Authorized domains → Add domain**: add where the game is hosted, e.g. `caroline6741.github.io`. (`localhost` is already allowed for testing.)

Reload the game — the panel under the wallet now shows **Log in / Sign up**. Sign up once, then log in from any laptop and your bombs, guns, skins, XP and rounds-won carry over. Changes sync live between open devices.

### Online multiplayer (Firebase Realtime Database)

Rooms live in Firebase **Realtime Database** (a separate product from Firestore, built for fast tiny updates). Each player in a room writes their position ~12×/sec to `rooms/<code>/players/<session>`; hits, deaths and round wins are small records under `rooms/<code>/events`. A player's node is removed automatically when their tab closes (`onDisconnect`), and the last player out deletes the room.

Setup is already done for `caroline-s-game`. For a new project: **Build → Realtime Database → Create database**, then deploy [`database.rules.json`](database.rules.json) (`firebase deploy --only database`) and add the `databaseURL` to [`firebase-config.js`](firebase-config.js). Players must be logged in to use rooms.

There is no authoritative game server, so a determined cheater could edit their own browser — fine for friends and classrooms. The upgrade path, if it ever matters, is a dedicated WebSocket server; the room/lobby design stays the same.

### Optional: host on Firebase instead of GitHub Pages

With the Firebase CLI installed (`npm i -g firebase-tools`):

```bash
firebase login
firebase use --add        # pick your project
firebase deploy           # publishes the site and the Firestore rules
```

Your game will be live at `https://<project-id>.web.app`, and that domain is authorized for login automatically.
