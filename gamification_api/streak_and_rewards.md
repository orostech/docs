# Streaks & Daily Rewards API — Complete Reference

Base Endpoint: `/gamification/streak/`
Authentication: **Required** (all endpoints)

---

## How the Streak System Works

1. Users check in **once per day** to earn coins
2. Rewards follow a repeating **7-day cycle** with escalating payouts
3. Day 7 is a **jackpot day** with a 50-coin bonus on top
4. Missing a day **breaks** the streak and resets the cycle to Day 1
5. The streak counter tracks total consecutive days, not just the 7-day cycle

---

## 7-Day Reward Cycle

| Day | Coins | Bonus | Total | UI Suggestion |
|:---:|:-----:|:-----:|:-----:|:-------------|
| 1 | 10 | 0 | **10** | Small coin stack |
| 2 | 15 | 0 | **15** | Small coin stack |
| 3 | 20 | 0 | **20** | Medium coin stack |
| 4 | 30 | 0 | **30** | Medium coin stack |
| 5 | 40 | 0 | **40** | Large coin stack |
| 6 | 50 | 0 | **50** | Large coin stack |
| 7 | 100 | 50 | **150** | 🎉 **Treasure chest animation** |

> **Full cycle total:** 265 coins + 50 bonus = **315 coins per 7 days**
>
> After Day 7, the cycle restarts at Day 1 automatically while the overall streak counter continues.

### Calendar Status Values

Each day in the `rewards_calendar` array has a `status` field:

| Status | Meaning | How to Render |
|:-------|:--------|:-------------|
| `claimed` | Already collected | ✅ Checkmark overlay, greyed background |
| `current` | Today — available to claim | 🟡 Animated glow, pulsing border |
| `locked` | Future day | 🔒 Dimmed with lock icon |

### Streak Statuses

| Status | Meaning |
|:-------|:--------|
| `active` | Streak is ongoing, hasn't missed a day |
| `claimed` | Already checked in today |
| `broken` | Missed yesterday — streak reset |

---

## REST Endpoints

### 1. Get Streak Status

Retrieve the full streak state including the 7-day reward calendar.

- **URL:** `GET /gamification/streak/`
- **Auth:** Required

**Response:**
```json
{
  "current_streak": 21,
  "longest_streak": 35,
  "total_check_ins": 42,
  "day_in_cycle": 2,
  "can_claim": true,
  "status": "active",
  "next_reward": {
    "day": 2,
    "coins": 15,
    "bonus": 0
  },
  "rewards_calendar": [
    { "day": 1, "coins": 10, "bonus": 0, "status": "claimed" },
    { "day": 2, "coins": 15, "bonus": 0, "status": "current" },
    { "day": 3, "coins": 20, "bonus": 0, "status": "locked" },
    { "day": 4, "coins": 30, "bonus": 0, "status": "locked" },
    { "day": 5, "coins": 40, "bonus": 0, "status": "locked" },
    { "day": 6, "coins": 50, "bonus": 0, "status": "locked" },
    { "day": 7, "coins": 100, "bonus": 50, "status": "locked" }
  ],
  "last_check_in": "2026-04-07"
}
```

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `current_streak` | int | Consecutive days checked in |
| `longest_streak` | int | All-time record streak |
| `total_check_ins` | int | Lifetime total check-in count |
| `day_in_cycle` | int (1-7) | Current position in the 7-day cycle |
| `can_claim` | bool | `true` if user hasn't checked in today |
| `status` | string | `active`, `claimed`, or `broken` |
| `next_reward` | object | Preview of today's available reward |
| `rewards_calendar` | array[7] | Complete 7-day cycle with status per day |
| `last_check_in` | ISO date | Date of last check-in |

> **Key frontend logic:**
> - If `can_claim` is `true` → show the reward calendar modal on app open
> - If `status` is `broken` → show a "streak reset" message and encourage user to start again

---

### 2. Claim Daily Reward (REST)

Claim today's check-in reward. Can only be called once per day.

- **URL:** `POST /gamification/streak/check-in/`
- **Auth:** Required
- **Body:** None required

**Success Response (200):**
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

**Error Response (400):**
```json
{
  "success": false,
  "message": "Already checked in today"
}
```

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `day_number` | int (1-7) | Which day in the cycle was claimed |
| `coins_earned` | int | Base coins earned |
| `bonus_earned` | int | Extra bonus (only Day 7) |
| `total_coins` | int | `coins_earned` + `bonus_earned` |
| `is_day_7` | bool | `true` if this was the jackpot day |
| `next_day_preview` | int | How many coins tomorrow's reward is worth |

> **UI tip:** If `is_day_7` is `true`, play a special treasure chest animation!

---

### 3. Check-in History

Get a list of past check-in records with coins earned per day.

- **URL:** `GET /gamification/streak/history/`
- **Auth:** Required
- **Query Params:**

| Parameter | Type | Default | Description |
|-----------|------|---------|-------------|
| `limit` | int | `30` | Number of records to return |

**Response** — Array of check-in entries:
```json
[
  {
    "check_in_date": "2026-04-08",
    "day_number": 1,
    "streak_count": 21,
    "coins_earned": 10,
    "bonus_earned": 0,
    "created_at": "2026-04-08T08:15:00Z"
  }
]
```

---

## WebSocket Events

### Claim Reward via WebSocket (Preferred Method)

**Send:**
```json
{
  "type": "claim_daily_reward",
  "data": {}
}
```

**Receive on success:** `daily_reward_claimed`
```json
{
  "type": "daily_reward_claimed",
  "data": {
    "success": true,
    "day_number": 2,
    "coins_earned": 15,
    "rewards_calendar": [
      { "day": 1, "coins": 10, "bonus": 0, "status": "claimed" },
      { "day": 2, "coins": 15, "bonus": 0, "status": "claimed" },
      { "day": 3, "coins": 20, "bonus": 0, "status": "current" },
      { "day": 4, "coins": 30, "bonus": 0, "status": "locked" },
      { "day": 5, "coins": 40, "bonus": 0, "status": "locked" },
      { "day": 6, "coins": 50, "bonus": 0, "status": "locked" },
      { "day": 7, "coins": 100, "bonus": 50, "status": "locked" }
    ]
  }
}
```

> **Note:** The WebSocket response includes the **full refreshed calendar** so you can update the UI in one step.

**Receive on error:** `daily_reward_error`
```json
{
  "type": "daily_reward_error",
  "data": {
    "success": false,
    "message": "Already checked in today"
  }
}
```

### Get Streak Status via WebSocket

**Send:**
```json
{
  "type": "get_streak_status",
  "data": {}
}
```

**Receive:** `streak_status` — same payload as REST endpoint #1.

### Streak Milestone (Server Push)

When the user hits a streak milestone (7, 30, 100 days), the server automatically pushes:

```json
{
  "type": "streak_milestone",
  "data": {
    "streak_count": 30,
    "reward_coins": 500
  }
}
```

> **Display a celebration modal** with the streak count and coins earned.

---

## Streak Badge Integration

Streaks automatically trigger badge checks:

| Streak Days | Badge Earned |
|:-----------:|:-------------|
| 7 | 7-Day Streak (BRONZE) |
| 30 | 30-Day Streak (SILVER) |
| 100 | 100-Day Streak (GOLD) |

When a streak badge is earned, you'll receive a separate `badge_earned` WebSocket event. See [badges.md](./badges.md) for details.
