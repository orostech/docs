# Leaderboards API — Complete Reference

Base Endpoint: `/gamification/leaderboard/`
Authentication: **Required** (all endpoints)

---

## Leaderboard Types

The platform has **5 distinct leaderboard categories**:

| Key | Display Label | What It Measures | Score Source | Refresh |
|:----|:-------------|:-----------------|:------------|:--------|
| `most_active` | Most Active | Overall platform activity | Posts + comments + check-ins + logins | Real-time |
| `top_streaks` | Streak Masters | Longest currently active streak | `current_streak` from UserStreak model | Daily |
| `top_referrers` | Top Ambassadors | Most successful referrals | Completed referral count | Real-time |
| `top_gifters` | Top Gifters | Most generous users | Total gifts sent | Real-time |
| `weekly_active` | Weekly Active | Short-window weekly engagement | Weekly activity points | Resets weekly |

---

## REST Endpoints

### 1. Main Leaderboard (Filterable)

Get any leaderboard with optional period and type filters.

- **URL:** `GET /gamification/leaderboard/`
- **Auth:** Required

**Query Parameters:**

| Parameter | Type | Default | Options | Description |
|-----------|------|---------|---------|-------------|
| `period` | string | `weekly` | `daily`, `weekly`, `monthly`, `all-time` | Time window filter |
| `type` | string | `global` | `global`, `active`, `streaks`, `referrers` | Leaderboard category |
| `limit` | int | `50` | Any positive int | Number of entries returned |

**Type-to-Backend Mapping:**

| Query `type` value | Backend `LeaderboardType` |
|:-------------------|:------------------------|
| `global` | `most_active` |
| `active` | `most_active` |
| `streaks` | `top_streaks` |
| `referrers` | `top_referrers` |

**Response:**
```json
{
  "leaderboard_type": "most_active",
  "period": "weekly",
  "entries": [
    {
      "rank": 1,
      "score": 8500,
      "display_name": "Adaobi Chukwuemeka",
      "profile_photo_url": "https://example.com/photo.jpg",
      "user_id": "550e8400-e29b-41d4-a716-446655440000",
      "is_current_user": false
    },
    {
      "rank": 2,
      "score": 7200,
      "display_name": "Chinedu Okoro",
      "profile_photo_url": "https://example.com/photo2.jpg",
      "user_id": "uuid-2",
      "is_current_user": true
    }
  ],
  "user_rank": {
    "rank": 2,
    "score": 7200
  },
  "total_participants": 1500,
  "user_percentile": 99,
  "user_percentile_message": "You are among the top 1% of our most_active!"
}
```

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `leaderboard_type` | string | Which board this is |
| `period` | string | Time period filter applied |
| `entries` | array | Ranked list of users |
| `entries[].rank` | int | Position (1 = first place) |
| `entries[].score` | int | Numeric score |
| `entries[].display_name` | string | User's display name |
| `entries[].profile_photo_url` | string? | Avatar URL (may be null) |
| `entries[].user_id` | UUID | User ID |
| `entries[].is_current_user` | bool | `true` if this entry is the requesting user |
| `user_rank` | object? | Current user's position (null if not ranked) |
| `user_rank.rank` | int | User's rank number |
| `user_rank.score` | int | User's score |
| `total_participants` | int | Total users on this board |
| `user_percentile` | int | User's percentile (0-100) |
| `user_percentile_message` | string | Human-readable percentile message |

> **UI tip:** Highlight the entry where `is_current_user` is `true` with a distinct background color. If the user isn't in the top results, show their rank separately below the list using `user_rank`.

---

### 2. Most Active Users (Shortcut)

- **URL:** `GET /gamification/leaderboard/active/`
- **Auth:** Required
- **Query Params:** `limit` (default: 50)

Same response format as endpoint #1 with `leaderboard_type: "most_active"`.

---

### 3. Streak Masters (Shortcut)

- **URL:** `GET /gamification/leaderboard/streaks/`
- **Auth:** Required
- **Query Params:** `limit` (default: 50)

Same response format with `leaderboard_type: "top_streaks"`.

---

### 4. Top Ambassadors (Shortcut)

- **URL:** `GET /gamification/leaderboard/referrers/`
- **Auth:** Required
- **Query Params:** `limit` (default: 50)

Same response format with `leaderboard_type: "top_referrers"`.

---

### 5. My Rankings Across All Boards

Get the current user's rank across **all** leaderboard types in a single request.

- **URL:** `GET /gamification/leaderboard/my-ranks/`
- **Auth:** Required

**Response:**
```json
{
  "active": {
    "rank": 3,
    "score": 8500
  },
  "streaks": {
    "rank": 2,
    "score": 150
  },
  "referrers": {
    "rank": 5,
    "score": 10
  }
}
```

> **Note:** A value of `null` for any key means the user has no entry on that board yet.

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `active` | object? | Rank on Most Active board |
| `streaks` | object? | Rank on Streak Masters board |
| `referrers` | object? | Rank on Top Ambassadors board |
| `*.rank` | int | Numeric rank position |
| `*.score` | int | User's score on that board |

---

## WebSocket Events

### `leaderboard_update` — Server Push (Auto)

When a user's rank **improves** on any leaderboard, the server pushes:

```json
{
  "type": "leaderboard_update",
  "data": {
    "leaderboard_type": "most_active",
    "new_rank": 5,
    "previous_rank": 8
  }
}
```

> **Only sent on rank improvement.** If rank stays the same or drops, no event is sent.

**Recommended UX:** Show a toast notification: "📈 You moved up to **#5** on Most Active!"

---

## Leaderboard Entry Schema

Each leaderboard entry stored in the database has these fields:

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Entry primary key |
| `leaderboard_type` | string | One of the 5 board types |
| `user` | FK → User | The ranked user |
| `rank` | int | Current position |
| `score` | BigInt | Numeric score (supports large values) |
| `display_name` | string | Denormalized for fast display |
| `profile_photo_url` | string? | Denormalized avatar URL |
| `period_start` | date? | Start of ranking period (weekly/monthly) |
| `period_end` | date? | End of ranking period |

> Entries are **pre-computed snapshots** refreshed by a Celery background task, not computed on-the-fly. This ensures fast reads even with 100k+ users.
