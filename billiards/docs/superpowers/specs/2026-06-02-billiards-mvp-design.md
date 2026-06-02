# Billiards Club MVP Design

## Goal

Build a web-first MVP for a persistent billiards club room where friends keep creating games, recording scores, watching live matches, making pre-game winner predictions, and reviewing room-specific records and rankings.

## Product Summary

The service is centered on a persistent room, not a one-off game lobby. A group of friends creates a room once, keeps its members there, and creates a new game inside the same room whenever some members play billiards. Finished games accumulate into room-only records, rankings, recent form, head-to-head stats, and streaks.

The MVP should prove these concepts:

- A room remains after games end.
- Multiple games can be created inside one room.
- Game results accumulate into room-specific statistics.
- Non-participants can watch a live game.
- The game screen shows who is currently watching.
- Betting is only available before the game starts.
- Betting records winner predictions only; it does not manage money, rewards, or penalties.

## Chosen Approach

Use a split web app:

- `frontend`: React web app for the MVP user experience.
- `backend`: Node.js, Express, SQLite, and WebSocket server.

The project starts with SQLite for speed, but the schema and query boundaries should avoid SQLite-only assumptions where practical so a later PostgreSQL migration remains straightforward. The backend exposes HTTP APIs for durable actions and WebSocket events for live score and spectator updates.

## MVP User Identity

The MVP uses a lightweight account model instead of full authentication.

Users are created with a nickname and a server-generated user ID. The frontend stores the current user ID locally and sends it with API requests. This gives the MVP a stable identity for room membership, game history, betting, and stats without adding signup, passwords, sessions, or social login.

Authentication and account recovery are outside the MVP.

## Core Screens

### Entry and Room List

Users can create or select their nickname, then see the rooms they belong to. They can create a new room or join an existing room using an invite code.

### Room Detail

The room detail screen shows:

- Room name and invite code.
- Current members.
- Active games in the room.
- Recent completed games.
- Ranking summary.
- Button to create a new game.

This screen must make it clear that the room is persistent and game records are accumulating inside it.

### Game Creation

A room member can create a game by selecting 2 to 4 room members as participants.

For each participant, the creator can enter:

- Starting score.
- Turn order or first-player marker.

The creator can also enter:

- Game type or short memo.

New games begin in `pending` status. Betting is open while the game is pending.

### Betting

Room members who are not game participants can predict the final winner before the game starts. A bet records:

- Game ID.
- Bettor user ID.
- Predicted winner participant user ID.
- Result status after the game ends.

The server must reject betting attempts once the game is `active`, `completed`, or `cancelled`.

The MVP does not store bet amounts, money, rewards, penalties, drinks, settlement status, or payment information.

### Game Play

When the game starts, status changes from `pending` to `active` and betting closes.

During active play, participants or room members can adjust participant scores up or down. Each score change is stored as a score event and applied to the participant's current score.

The game screen shows:

- Participants and current scores.
- Current ranking by score for display only.
- Game status.
- Betting closed/open state.
- Current spectators.
- Score adjustment controls.
- Finish game action.

Because billiards rules vary by group, the MVP does not enforce a target-score or handicap rule. When finishing a game, the user manually confirms the final rank order for all participants.

### Spectator Mode

Room members who are not playing can enter spectator mode for an active or pending game. Spectator mode shows:

- Current scores.
- Current ranking by score.
- Game status.
- Current spectator list.
- Betting state.

Entering spectator mode registers the user as a current spectator. Leaving the screen or disconnecting should remove the user from the live spectator list after a short server-side timeout or disconnect event.

### Stats

Stats are calculated within a room only. The MVP does not provide global user rankings.

The room stats screen shows:

- Games played per member.
- First-place wins.
- Losses, meaning participated and did not finish first.
- Win rate, calculated as first-place wins divided by games played.
- Room ranking by win count and win rate.
- Recent results.
- Head-to-head summary between members.
- Win/loss streaks based on first-place finish.

For multi-player games, only the rank 1 participant counts as the winner. All other participants count as non-winners for win rate and streak purposes.

## Data Model

### users

- `id`: stable user identifier.
- `nickname`: display name.
- `created_at`: creation time.

### rooms

- `id`: stable room identifier.
- `name`: room name.
- `invite_code`: short join code.
- `created_by`: user ID.
- `created_at`: creation time.

### room_members

- `room_id`: room ID.
- `user_id`: user ID.
- `role`: `owner` or `member`.
- `joined_at`: join time.

### games

- `id`: stable game identifier.
- `room_id`: room ID.
- `status`: `pending`, `active`, `completed`, or `cancelled`.
- `memo`: optional game type or note.
- `started_at`: nullable start time.
- `completed_at`: nullable completion time.
- `created_by`: user ID.
- `created_at`: creation time.

### game_participants

- `game_id`: game ID.
- `user_id`: participant user ID.
- `starting_score`: initial score entered during game creation.
- `current_score`: current score after score events.
- `turn_order`: nullable order number.
- `is_first`: whether this participant has first turn.
- `final_rank`: nullable rank assigned when the game is completed.

The schema supports 2 to 4 participants in the MVP. The server validates participant count and membership.

### score_events

- `id`: stable score event identifier.
- `game_id`: game ID.
- `user_id`: participant whose score changed.
- `delta`: positive or negative score delta.
- `score_after`: participant score after the change.
- `created_by`: user ID who recorded the event.
- `created_at`: event time.

### spectators

- `game_id`: game ID.
- `user_id`: spectator user ID.
- `joined_at`: join time.
- `last_seen_at`: last heartbeat or socket activity time.

Spectators are live presence records, not historical attendance records.

### bets

- `id`: stable bet identifier.
- `game_id`: game ID.
- `bettor_user_id`: non-participant user ID.
- `predicted_winner_user_id`: participant user ID predicted to finish first.
- `result`: `pending`, `won`, or `lost`.
- `created_at`: creation time.

The server validates that the bettor is a room member, is not a game participant, and is betting only while the game is pending.

## HTTP API

The MVP backend exposes APIs for durable actions:

- `POST /api/users`: create lightweight user.
- `GET /api/rooms`: list rooms for current user.
- `POST /api/rooms`: create room.
- `POST /api/rooms/join`: join by invite code.
- `GET /api/rooms/:roomId`: room detail.
- `GET /api/rooms/:roomId/stats`: room stats.
- `POST /api/rooms/:roomId/games`: create pending game.
- `GET /api/games/:gameId`: game detail.
- `POST /api/games/:gameId/start`: start game and close betting.
- `POST /api/games/:gameId/score-events`: record score delta.
- `POST /api/games/:gameId/complete`: save final ranks and resolve bets.
- `POST /api/games/:gameId/bets`: create winner prediction.

The exact API shape can be refined during implementation, but these routes cover the MVP workflows.

## WebSocket Events

The WebSocket layer handles live game updates.

Client to server:

- `join_game`: user starts watching a game channel.
- `leave_game`: user leaves a game channel.
- `spectator_heartbeat`: user confirms they are still watching.

Server to client:

- `game_snapshot`: current game state after joining.
- `score_updated`: participant score changed.
- `spectators_updated`: live spectator list changed.
- `game_status_changed`: game moved between pending, active, completed, or cancelled.
- `betting_closed`: game started and betting is no longer accepted.

HTTP remains the source of truth for writes. WebSocket broadcasts are used to update connected clients after state changes.

## Validation Rules

- A game can only be created inside an existing room.
- Game participants must be room members.
- MVP games must have 2 to 4 participants.
- A user cannot bet on a game they participate in.
- A bet can only predict one of the game participants.
- A user can place at most one bet per game.
- Bets are accepted only while the game is pending.
- Scores can be adjusted only while the game is active.
- Completing a game requires a final rank for every participant.
- Exactly one participant must have final rank 1.
- Completed games cannot receive more score events or bets.

## Statistics Rules

Stats are computed from completed games only.

For each room member:

- `games_played`: count of completed games they participated in.
- `wins`: count of completed games where their final rank is 1.
- `losses`: games played minus wins.
- `win_rate`: wins divided by games played, displayed as 0% when games played is 0.
- `recent_results`: recent completed games as W/L based on final rank 1.
- `streak`: consecutive W or L result from most recent games backward.

Head-to-head stats are computed for members who appeared in the same completed game. In a multi-player game, a participant is considered to have beaten every lower-ranked participant and lost to every higher-ranked participant.

Room ranking sorts by wins first, then win rate, then games played.

## Frontend State and UX

The frontend should keep screens practical and app-like, not marketing-style.

Primary UX principles:

- Dense but readable room dashboard.
- Clear game status labels: pending, active, completed.
- Stable score controls that do not shift layout.
- Spectator list visible directly on the game screen.
- Betting section visibly disabled after the game starts.
- Stats tied clearly to the current room.

The MVP can use seeded empty-state examples or simple prompts when there are no games yet, but it should not hide the main workflows behind a landing page.

## Error Handling

Backend responses should return clear validation errors for invalid actions, such as:

- Betting after the game has started.
- Creating a game with fewer than 2 or more than 4 participants.
- Completing a game with duplicate or missing ranks.
- Recording a score for a non-participant.
- Joining a room with an invalid invite code.

The frontend should display these errors near the action that triggered them.

## Testing Strategy

Backend tests should cover:

- Game creation validation.
- Betting allowed while pending and rejected after start.
- Score event updates current score.
- Game completion stores final ranks.
- Bet resolution after game completion.
- Room stats calculations for multi-player games.
- Head-to-head calculations for multi-player games.

Frontend tests should cover:

- Room detail renders persistent room information.
- Game creation allows 2 to 4 participants.
- Betting UI disables after game start.
- Game screen renders live score and spectator list.
- Stats screen renders room-only rankings.

WebSocket tests should cover:

- Joining a game returns a game snapshot.
- Score changes broadcast `score_updated`.
- Spectator join and leave broadcast `spectators_updated`.

## Out of Scope for MVP

- Native mobile app.
- Push notifications, live notifications, or lock-screen score updates.
- Real money betting, payment, settlement, rewards, or penalties.
- Full authentication, password login, social login, and account recovery.
- Public/global rankings across all rooms.
- Automatic billiards rule enforcement.
- Tournament brackets or season management.

## Future Extensions

- Mobile app using the same backend API.
- App notification or live notification showing current spectator score.
- PostgreSQL migration.
- Authentication and account recovery.
- More detailed billiards rule presets.
- Historical spectator attendance.
- Richer betting settlement notes for drinks or informal penalties without handling money.
