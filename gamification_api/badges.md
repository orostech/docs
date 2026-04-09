# Badges API — Complete Reference

Base Endpoint: `/gamification/badges/`
Authentication: **Required** (all endpoints)

---

## Badge System Overview

Badges are achievement milestones awarded **automatically** when users hit specific thresholds (e.g., 50 posts, 7-day streak). They cannot be purchased — they must be earned. When earned, the server pushes a real-time `badge_earned` event for a celebratory popup.

---

## Complete Badge Library

| # | Category | Tier | Slug | Name | Threshold | Coin Bonus | Auto-Trigger |
|:-:|:---------|:-----|:-----|:-----|:---------:|:----------:|:-------------|
| 1 | Streak | BRONZE | `streak_7` | 7-Day Streak | 7 consecutive days | — | Daily check-in |
| 2 | Streak | SILVER | `streak_30` | 30-Day Streak | 30 consecutive days | — | Daily check-in |
| 3 | Streak | GOLD | `streak_100` | 100-Day Streak | 100 consecutive days | — | Daily check-in |
| 4 | Engagement | BRONZE | `posts_10` | Voice Starter | 10 posts total | — | Any post (Vent + Community) |
| 5 | Engagement | SILVER | `posts_50` | Content Creator | 50 posts total | — | Any post |
| 6 | Engagement | GOLD | `posts_100` | Community Voice | 100 posts total | — | Any post |
| 7 | Ambassador | BRONZE | `referrals_5` | Bronze Ambassador | 5 completed referrals | 500 | Referral status → COMPLETED |
| 8 | Ambassador | SILVER | `referrals_10` | Silver Ambassador | 10 completed referrals | 1,000 | Referral status → COMPLETED |
| 9 | Ambassador | GOLD | `referrals_25` | Gold Ambassador | 25 completed referrals | 2,500 | Referral status → COMPLETED |
| 10 | Action | BRONZE | `profile_complete` | Profile Pro | 80% profile completeness | — | Profile save |
| 11 | Action | BRONZE | `voice_bio` | Voice Identity | 1 voice bio uploaded | — | Voice bio field set |
| 12 | Action | BRONZE | `first_gift` | Generous Heart | 1 gift sent | — | First gift created |
| 13 | Premium | PLATINUM | `premium` | Premium Member | 1 active subscription | — | Subscription activated |
| 14 | Premium | SPECIAL | `verified` | Verified User | Admin verified | — | Manual |

### Badge Tier Color Map

Use these for card borders, glow effects, and badge icons:

| Tier | Color Hex | RGB | Suggested CSS |
|:-----|:----------|:----|:--------------|
| BRONZE | `#CD7F32` | `rgb(205, 127, 50)` | `border-color: #CD7F32; box-shadow: 0 0 8px rgba(205,127,50,0.4)` |
| SILVER | `#C0C0C0` | `rgb(192, 192, 192)` | `border-color: #C0C0C0; box-shadow: 0 0 8px rgba(192,192,192,0.4)` |
| GOLD | `#FFD700` | `rgb(255, 215, 0)` | `border-color: #FFD700; box-shadow: 0 0 12px rgba(255,215,0,0.5)` |
| PLATINUM | `#00CED1` | `rgb(0, 206, 209)` | `border-color: #00CED1; box-shadow: 0 0 16px rgba(0,206,209,0.5)` |
| SPECIAL | Custom | varies | Use animated gradient border |

---

## REST Endpoints

### 1. My Earned Badges

Get all badges the current user has earned.

- **URL:** `GET /gamification/badges/`
- **Auth:** Required

**Response** — Array of earned badges:
```json
[
  {
    "id": "uuid",
    "slug": "streak_30",
    "name": "30-Day Streak",
    "description": "Checked in for 30 consecutive days!",
    "tier": "SILVER",
    "icon_url": "https://...",
    "icon_name": "fire-silver",
    "color": "#C0C0C0",
    "earned_at": "2026-03-15T10:30:00Z",
    "earned_value": 35
  }
]
```

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique badge ID |
| `slug` | string | Machine-readable identifier |
| `name` | string | Display name |
| `description` | string | Short description |
| `tier` | string | `BRONZE` / `SILVER` / `GOLD` / `PLATINUM` / `SPECIAL` |
| `icon_url` | string? | Custom icon URL (may be null) |
| `icon_name` | string | Icon name for local icon lookup |
| `color` | string | Hex color code |
| `earned_at` | ISO datetime | When the badge was earned |
| `earned_value` | int | Value at time of earning (e.g., streak count was 35) |

---

### 2. All Badges Library

Get every available badge in the system, including whether the current user has earned each one.

- **URL:** `GET /gamification/badges/all/`
- **Auth:** Required

**Response** — Array of all badges:
```json
[
  {
    "id": "uuid",
    "slug": "streak_7",
    "name": "7-Day Streak",
    "description": "Check in 7 consecutive days",
    "tier": "BRONZE",
    "icon_url": null,
    "icon_name": "fire-bronze",
    "color": "#CD7F32",
    "threshold": 7,
    "coin_bonus": 0,
    "earned": true
  },
  {
    "id": "uuid",
    "slug": "posts_100",
    "name": "Community Voice",
    "description": "Create 100 posts across the platform",
    "tier": "GOLD",
    "icon_url": null,
    "icon_name": "trophy-gold",
    "color": "#FFD700",
    "threshold": 100,
    "coin_bonus": 0,
    "earned": false
  }
]
```

> **UI tip:** Use the `earned` boolean to toggle between a full-color badge and a greyed-out locked state.

> **Secret badges:** Badges marked `is_secret` in the admin are automatically filtered out if the user hasn't earned them. They appear only after earning.

---

### 3. Badge Progress (Unearned Badges)

Get the user's current progress percentage toward unlocking **each unearned badge**.

- **URL:** `GET /gamification/badges/progress/`
- **Auth:** Required

**Response** — Array of unearned badges with progress:
```json
[
  {
    "slug": "posts_50",
    "name": "Content Creator",
    "description": "Create 50 posts across the platform",
    "tier": "SILVER",
    "icon_url": null,
    "icon_name": "trophy-silver",
    "color": "#C0C0C0",
    "threshold": 50,
    "current_value": 35,
    "progress_percent": 70,
    "coin_bonus": 0
  }
]
```

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `threshold` | int | Target value needed to earn badge |
| `current_value` | int | User's current progress toward threshold |
| `progress_percent` | int (0-100) | Pre-computed percentage — **use this directly for progress bars** |
| `coin_bonus` | int | Coins awarded when badge is earned |

> **UI tip:** Render a progress bar using `progress_percent` directly. No client-side calculation needed.

---

### 4. User's Public Badges

Get earned badges for another specific user (used on profile view screens).

- **URL:** `GET /gamification/badges/user/{user_id}/`
- **Auth:** Required

Same response format as endpoint #1.

**Error Responses:**

| Status | Body | When |
|--------|------|------|
| `404` | `{ "error": "User not found" }` | Invalid `user_id` |

---

## WebSocket Events

### `badge_earned` — Real-Time Congratulations Modal

Whenever a user's action crosses a badge threshold, the server pushes this event. **You must display a celebratory popup/modal.**

**Event received:**
```json
{
  "type": "badge_earned",
  "data": {
    "badge_name": "Content Creator",
    "badge_slug": "posts_50",
    "badge_tier": "SILVER",
    "badge_icon_name": "trophy-silver",
    "badge_color": "#C0C0C0",
    "badge_description": "You created 50 posts!",
    "coin_bonus": 0
  }
}
```

**Recommended congratulations modal content:**

| Element | Source Field |
|---------|-------------|
| Title | `"🏅 New Badge Earned!"` |
| Badge icon | Use `badge_icon_name` to render local icon, tinted with `badge_color` |
| Badge name | `badge_name` |
| Description | `badge_description` |
| Coin reward line | Show only if `coin_bonus > 0`: `"You earned {coin_bonus} coins!"` |
| Badge tier label | `badge_tier` — style with tier color |

### `get_badges` — Request Badges via WebSocket

**Send:**
```json
{
  "type": "get_badges",
  "data": {}
}
```

**Receive:** `badges_list`
```json
{
  "type": "badges_list",
  "data": [
    {
      "id": "uuid",
      "slug": "streak_7",
      "name": "7-Day Streak",
      "tier": "BRONZE",
      "color": "#CD7F32",
      "earned_at": "2026-01-15T10:30:00Z"
    }
  ]
}
```

---

## Badge Award Flow (How It Works Behind the Scenes)

```
User Action (e.g., creates a post)
        │
        ▼
Django Signal fires (post_save)
        │
        ▼
BadgeService.check_post_badges(user, post_count)
        │
        ▼
Threshold met? ── No ──► Nothing happens
        │
       Yes
        │
        ▼
UserBadge record created + coin_bonus added to wallet
        │
        ▼
GamificationNotificationService.notify_badge_earned()
        │
        ├──► WebSocket push (if user online)
        └──► FCM push notification (if user offline)
```
