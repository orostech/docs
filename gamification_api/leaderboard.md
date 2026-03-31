# Leaderboards API

Base Endpoint: `/gamification/leaderboard/`

Leaderboards rank users across metrics system-wide based on recent timeframe intervals. It aggregates data into dynamic rankings calculating real-time percentiles.

## Endpoints

### 1. View Leaderboards List
Retrieves full user listings.

- **URL**: `/gamification/leaderboard/`
- **Method**: `GET`
- **Auth**: Required

**Query Parameters**
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `period` | string | No | `daily`, `weekly`, `monthly`, `all-time` (default: `weekly`) |
| `type` | string | No | `active`, `streaks`, `referrers` (default: `global`) |
| `limit` | int | No | Returns top N results (default: `50`) |

**Response**
```json
{
  "leaderboard_type": "MOST_ACTIVE",
  "entries": [
    {
      "rank": 1,
      "score": 55,
      "display_name": "User Alpha",
      "profile_photo_url": "https://...",
      "user_id": "uuid",
      "is_current_user": false
    }
  ],
  "user_rank": { "rank": 40, "score": 8, "user_id": "..." },
  "total_participants": 150,
  "user_percentile": 27,
  "user_percentile_message": "You are among the top 27% of our MOST_ACTIVE!"
}
```

### 2. Top Ambassadors View
Shorthand for `/?type=referrers`.

- **URL**: `/gamification/leaderboard/referrers/`
- **Method**: `GET`

### 3. Streak Masters View
Shorthand for `/?type=streaks`.

- **URL**: `/gamification/leaderboard/streaks/`
- **Method**: `GET`
