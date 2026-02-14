# Gamification API Documentation

## Overview

The Gamification system engages users through streaks, missions, badges, and leaderboards.
All real-time updates (like earning a badge or completing a mission) are delivered via the unified WebSocket connection.

---

## Table of Contents

1. [HTTP Endpoints](#http-endpoints)
   - [Streaks](#1-streaks)
   - [Missions](#2-missions)
   - [Badges](#3-badges)
   - [Leaderboards](#4-leaderboards)
2. [WebSocket Events](#websocket-events)
   - [Connection](#connection)
   - [Incoming Commands](#incoming-commands)
   - [Outgoing Events](#outgoing-events)

---

## HTTP Endpoints

### 1. Streaks

#### Get Streak Status

Retrieve current streak count, last check-in, and freeze status.

- **URL**: `GET /gamification/streak/`
- **Auth**: Required
- **Response**:

```json
{
  "current_streak": 5,
  "longest_streak": 12,
  "last_check_in": "2023-10-27T10:00:00Z",
  "freeze_active": false,
  "next_milestone": 7,
  "next_reward": 100
}
```

#### Daily Check-In

Perform daily check-in to maintain streak and claim reward.

- **URL**: `POST /gamification/streak/check-in/`
- **Auth**: Required
- **Response**:

```json
{
  "success": true,
  "coins_earned": 50,
  "new_streak": 6,
  "message": "Check-in successful!"
}
```

### 2. Missions

#### List Missions

Get all available and completed missions for the user.

- **URL**: `GET /gamification/missions/`
- **Auth**: Required
- **Response**:

```json
[
  {
    "id": "mission-uuid",
    "title": "Social Butterfly",
    "description": "Comment on 5 posts",
    "progress": 3,
    "target": 5,
    "reward_coins": 200,
    "status": "IN_PROGRESS", // IN_PROGRESS, COMPLETED, CLAIMED
    "expires_at": "2023-10-30T23:59:59Z"
  }
]
```

#### Claim Mission Reward

Claim coins for a completed mission.

- **URL**: `POST /gamification/missions/{id}/claim/`
- **Auth**: Required
- **Response**:

```json
{
  "success": true,
  "coins_added": 200,
  "new_balance": 1500
}
```

### 3. Badges

#### List My Badges

Get badges earned by the current user.

- **URL**: `GET /gamification/badges/`
- **Auth**: Required
- **Response**:

```json
[
  {
    "slug": "early_adopter",
    "name": "Early Adopter",
    "description": "Joined in the first month",
    "icon_url": "https://...",
    "earned_at": "2023-09-01T10:00:00Z"
  }
]
```

### 4. Leaderboards

#### Get Leaderboard

Retrieve rankings for a specific category.

- **URL**: `GET /gamification/leaderboard/{category}/`
- **Categories**: `active` (most active), `streaks` (longest streaks), `referrers` (top inviters)
- **Response**:

```json
{
  "category": "streaks",
  "results": [
    {
      "rank": 1,
      "user": {
        "id": "uuid",
        "display_name": "Champion User",
        "avatar": "..."
      },
      "score": 45
    },
    ...
  ],
  "my_rank": {
    "rank": 15,
    "score": 12
  }
}
```

---

## WebSocket Events

> **Endpoint**: `wss://api.yourdomain.com/ws/realtime/?token={jwt_token}`

All gamification events are sent through the main realtime WebSocket.

### Connection

Connect using your JWT token. You will automatically be subscribed to your personal events channel.

### Incoming Commands (Client -> Server)

Send these JSON messages to the WebSocket to interact with gamification features continuously.

#### 1. Get Streak Status

```json
{
  "type": "get_streak_status"
}
```

#### 2. Claim Daily Reward

```json
{
  "type": "claim_daily_reward"
}
```

#### 3. Get Missions

```json
{
  "type": "get_missions"
}
```

#### 4. Claim Mission

```json
{
  "type": "claim_mission",
  "data": {
    "mission_id": "mission-uuid"
  }
}
```

#### 5. Get Badges

```json
{
  "type": "get_badges"
}
```

### Outgoing Events (Server -> Client)

#### Responses to Commands

The server sends these structured responses to your commands:

- `streak_status`: Response to `get_streak_status`
- `daily_reward_claimed`: Response to `claim_daily_reward` (contains `success`, `coins`)
- `daily_reward_error`: Error claiming reward
- `missions_list`: List of missions
- `mission_claimed`: Success response for mission claim
- `mission_claim_error`: Failure response
- `badges_list`: List of user badges

#### Real-time Notifications

These are pushed automatically when events happen (e.g., while browsing the app):

**1. Badge Earned**
Received when you unlock a new badge.

```json
{
  "type": "badge_earned",
  "data": {
    "badge_name": "Social Butterfly",
    "badge_slug": "social_butterfly",
    "coin_bonus": 50
  }
}
```

**2. Mission Complete**
Received when a mission goal is met.

```json
{
  "type": "mission_complete",
  "data": {
    "mission_id": "mission-uuid",
    "mission_title": "Post 5 comments",
    "coin_reward": 100
  }
}
```

**3. Streak Milestone**
Received when hitting a milestone (e.g., 7 days).

```json
{
  "type": "streak_milestone",
  "data": {
    "streak_count": 7,
    "reward_coins": 100
  }
}
```

**4. Daily Reward Available**
Reminder that check-in is available.

```json
{
  "type": "daily_reward_available",
  "data": {
    "day_number": 3,
    "coins_available": 20
  }
}
```

**5. Leaderboard Update**
Received when your rank improves.

```json
{
  "type": "leaderboard_update",
  "data": {
    "leaderboard_type": "streaks",
    "new_rank": 5,
    "previous_rank": 10
  }
}
```
