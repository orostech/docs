# Gamification API Documentation

## Overview
This directory contains documentation for the Hafar Gamification module. The system handles user engagement through streaks, daily rewards, missions, badges, referrals, and leaderboards. It heavily utilizes both REST APIs and real-time WebSockets to deliver instant feedback ("Congratulations" modals, dynamic badge updates, etc.) to the user interfaces.

## Modules

| Module | Description | File |
| :--- | :--- | :--- |
| **Streaks & Rewards** | Daily check-in tracking, 7-day cyclical rewards, and streak tracking. | [streak_and_rewards.md](./streak_and_rewards.md) |
| **Missions** | One-time, daily, and weekly tasks that reward coins and badges. | [missions.md](./missions.md) |
| **Badges** | Achievement badges awarded for milestone completions (e.g., 50 posts). | [badges.md](./badges.md) |
| **Leaderboards** | Global and categorized user rankings with percentiles. | [leaderboard.md](./leaderboard.md) |
| **Referrals** | Ambassador program tracking, milestones, and invite generation. | [referrals.md](./referrals.md) |

## Base Configuration

### Base URL
`https://api.joinhafar.com` (Production)
`http://localhost:8000` (Development)

Note: Most gamification REST endpoints are mounted under the `/gamification/` or `/referral/` prefix.
- Gamification: `/gamification/`
- Referrals: `/referral/`

### Authentication
All endpoints require authentication.
Include the JWT Access Token in the request header:

```http
Authorization: Bearer <your_access_token>
```

### WebSocket Integration
Many gamification features operate over the main WebSocket connection (`ws://<host>/ws/main/`).

When a user connects, they receive an `initial_data` event. This payload contains critical aggregates needed to render the gamified profile and popup Modals immediately:

**WebSocket `initial_data` Event**
```json
{
  "type": "initial_data",
  "data": {
    "gamification": {
      "show_daily_reward_popup": true,
      "badge_count": 8,
      "missions_completed": 3,
      "streak": {
        "current_streak": 21,
        "can_claim": true,
        "rewards_calendar": [
          {"day": 1, "coins": 10, "status": "claimed"},
          {"day": 2, "coins": 15, "status": "current"},
          {"day": 3, "coins": 20, "status": "locked"}
        ]
      }
    }
  }
}
```

### Integrated Profile Statistics
When fetching the current user profile overview (`GET /users/profiles/me/`), the payload natively includes gamification aggregates so the frontend doesn't need to make separate requests:

```json
{
  "badge_count": 8,
  "current_streak": 21,
  "missions_completed": 3,
  "daily_mission_progress": 72
}
```
