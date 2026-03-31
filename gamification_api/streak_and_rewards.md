# Streaks & Rewards API

Base Endpoint: `/gamification/streak/`

The Streaks and Daily Rewards system encourages user engagement through escalating cyclical daily coin bonuses. Users check-in once per day for 7 consecutive days to double their rewards and receive bonus items. Daily Check-in triggers can be managed entirely over WebSocket.

## Endpoints

### 1. Daily Check-in Status
View the current user's streak and full 7-day calendar cycle.

- **URL**: `/gamification/streak/`
- **Method**: `GET`
- **Auth**: Required

**Response**
```json
{
  "current_streak": 21,
  "longest_streak": 35,
  "total_check_ins": 42,
  "day_in_cycle": 2, // integer 1-7
  "can_claim": true, // Whether they can claim today
  "status": "active", // active, claimed, broken
  "next_reward": {
    "day": 2,
    "coins": 15,
    "bonus": 0
  },
  "rewards_calendar": [ // Easily render your UI 7-Day calendar using this
    {"day": 1, "coins": 10, "bonus": 0, "status": "claimed"},
    {"day": 2, "coins": 15, "bonus": 0, "status": "current"},
    {"day": 3, "coins": 20, "bonus": 0, "status": "locked"},
    {"day": 4, "coins": 30, "bonus": 0, "status": "locked"},
    {"day": 5, "coins": 40, "bonus": 0, "status": "locked"},
    {"day": 6, "coins": 50, "bonus": 0, "status": "locked"},
    {"day": 7, "coins": 100, "bonus": 50, "status": "locked"}
  ],
  "last_check_in": "2026-03-31" // ISO date
}
```

### 2. Claim Daily Reward (REST)
Claim current check-in. Usually better handled via WebSockets natively!

- **URL**: `/gamification/streak/check-in/`
- **Method**: `POST`
- **Auth**: Required

**Response**
```json
{
    "success": true,
    "day_number": 2,
    "coins_earned": 15,
    "bonus_earned": 0,
    "total_coins": 15,
    "is_day_7": false,
    "next_day_preview": 20
}
```

### 3. Check-in History
Returns a list of check-in entries tracking coin gains per day.

- **URL**: `/gamification/streak/history/`
- **Method**: `GET`
- **Auth**: Required
- **Query Params**: `limit` (default: 30)

---

## WebSocket Listeners

You don't need REST to claim rewards; the backend handles real-time payloads.

### Claim Reward over WebSocket

**Send this payload:**
```json
{
    "type": "claim_daily_reward",
    "data": {}
}
```

**Listen for response:** `daily_reward_claimed`
```json
{
    "type": "daily_reward_claimed",
    "data": {
        "success": true,
        "day_number": 2,
        "coins_earned": 15,
        "rewards_calendar": [
            // Returns the entire active 7-Day Calendar refreshing the UI statuses!
        ]
    }
}
```
