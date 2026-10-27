# Firestore / Firebase change notes

These are the backend changes to make before treating the current client as production-ready. This note does not modify Firestore or deploy rules.

## Player profile documents

The client stores player profiles in \`players/{uid}\`. Keep these fields:

- \`username\`: lowercase, unique, 3–16 characters (\`a-z\`, \`0-9\`, \`_\`, \`-\`).
- \`avatarColor\`: six-digit hex color such as \`#4fa3d1\`.
- \`name\`: in-game callsign.
- \`level\`, \`xp\`, \`roundsWon\`, \`winStreak\`, \`bestWinStreak\`.
- \`totalKills\`, \`headshots\`, \`botLevel\`, \`matchFormat\`.
- \`bombs\`, \`owned\`, \`equipped\`, \`ownedSkins\`, \`skins\`, \`ownedWraps\`, \`wrap\`.
- Existing event fields: \`eventKey\`, \`eventCasesEarned\`, \`eventCasesOpened\`, \`eventTasks\`, \`eventStats\`.
- \`updatedAt\`: client save timestamp.

## Username uniqueness

The current Firebase Auth implementation uses \`username@players.blastrivals.app\` as the login email, so Auth already rejects duplicate usernames. For a stronger server-side profile check, add a private lookup collection:

\`usernames/{username}\` → \`{ uid, createdAt }\`

Username changes should be performed as one protected server operation: reserve the new username, update the Auth email, update \`players/{uid}.username\`, then release the old username. Do not let clients write arbitrary username lookup documents.

## Multiplayer data

Team and lobby presence currently belongs in Realtime Database, not Firestore. The player presence payload includes \`team\`, \`format\`, \`map\`, \`playing\`, \`avatarColor\`, \`name\`, and position fields. Keep this separate from the persistent Firestore profile.

## Security rules / indexes

- Keep profile reads and writes limited to the authenticated owner.
- Add server validation for username format, avatar color format, numeric progression values, and allowed weapon/cosmetic IDs.
- No Firestore composite index is currently required.
- Deploy updated rules after adding any username lookup collection.
