# Gamification API — Complete Reference

> **Audience:** Frontend / Mobile developers integrating with the Hafar gamification backend.
> **Last updated:** 2026-04-09

---

## Table of Contents

| # | Section | File |
|---|---------|------|
| 1 | [Quick-Start Cheatsheet](#quick-start-cheatsheet) | — |
| 2 | [Platform Ranks & Tiers](#platform-ranks--tiers) | [ranks_and_thresholds.md](./ranks_and_thresholds.md) |
| 3 | [Streaks & Daily Rewards](#streaks--daily-rewards) | [streak_and_rewards.md](./streak_and_rewards.md) |
| 4 | [Missions](#missions) | [missions.md](./missions.md) |
| 5 | [Badges](#badges) | [badges.md](./badges.md) |
| 6 | [Leaderboards](#leaderboards) | [leaderboard.md](./leaderboard.md) |
| 7 | [Referrals](#referrals) | [referrals.md](./referrals.md) |
| 8 | [WebSocket Reference](#websocket-reference) | — |

---

## Base Configuration

### Base URLs

| Environment | URL |
|-------------|-----|
| **Production** | `https://api.joinhafar.com` |
| **Development** | `http://localhost:8000` |

### URL Prefixes

| Module | REST Prefix | Notes |
|--------|-------------|-------|
| Gamification (streaks, missions, badges, leaderboards) | `/gamification/` | All require auth |
| Referrals | `/referral/` | All require auth |
| Ranks & Config | `/ranks/` | **Public** — no auth needed |

### Authentication

All endpoints (except `/ranks/`) require a JWT Access Token:

```http
Authorization: Bearer <your_access_token>
```

---

## Quick-Start Cheatsheet

### All REST Endpoints at a Glance

| Method | Endpoint | Auth | Description |
|--------|----------|------|-------------|
| `GET` | `/ranks/` | ✗ | All subscription tiers, badge tiers, leaderboard types |
| `GET` | `/gamification/streak/` | ✓ | Current streak status + 7-day calendar |
| `POST` | `/gamification/streak/check-in/` | ✓ | Claim daily check-in reward |
| `GET` | `/gamification/streak/history/` | ✓ | Check-in history (paginated) |
| `GET` | `/gamification/missions/` | ✓ | All missions grouped by type |
| `POST` | `/gamification/missions/{id}/claim/` | ✓ | Claim completed mission reward |
| `GET` | `/gamification/badges/` | ✓ | Current user's earned badges |
| `GET` | `/gamification/badges/all/` | ✓ | Full badge library with earned status |
| `GET` | `/gamification/badges/progress/` | ✓ | Unearned badges with % progress |
| `GET` | `/gamification/badges/user/{user_id}/` | ✓ | Another user's public badges |
| `GET` | `/gamification/leaderboard/` | ✓ | Main leaderboard (with query filters) |
| `GET` | `/gamification/leaderboard/active/` | ✓ | Most Active leaderboard |
| `GET` | `/gamification/leaderboard/streaks/` | ✓ | Streak Masters leaderboard |
| `GET` | `/gamification/leaderboard/referrers/` | ✓ | Top Ambassadors leaderboard |
| `GET` | `/gamification/leaderboard/my-ranks/` | ✓ | Current user's rank on all boards |
| `GET` | `/referral/summary/` | ✓ | Referral stats, milestones, link |
| `GET` | `/referral/list/` | ✓ | List of referred users (paginated) |

### All WebSocket Events at a Glance

**Connection URL:** `ws://<host>/ws/realtime/`
**Auth:** JWT token via query param or header

#### Events You Send (Client → Server)

| `type` value | Payload | Description |
|--------------|---------|-------------|
| `ping` | `{}` | Keep-alive heartbeat |
| `get_streak_status` | `{}` | Request streak status |
| `claim_daily_reward` | `{}` | Claim today's check-in reward |
| `get_missions` | `{}` | Request missions list |
| `claim_mission` | `{ "mission_id": "uuid" }` | Claim a completed mission |
| `get_badges` | `{}` | Request user's badges |

#### Events You Receive (Server → Client)

| `type` value | When | Popup? |
|--------------|------|--------|
| `initial_data` | On connection | No — hydrate UI state |
| `pong` | After `ping` | No |
| `streak_status` | After `get_streak_status` | No |
| `daily_reward_claimed` | After successful claim | ✅ Show reward animation |
| `daily_reward_error` | Claim failed (already claimed) | No |
| `daily_reward_available` | Push reminder (offline users) | ✅ Notification |
| `missions_list` | After `get_missions` | No |
| `mission_complete` | Auto — mission target reached | ✅ Show "Claim" button |
| `mission_claimed` | After successful claim | ✅ Show coins earned |
| `mission_claim_error` | Claim failed | No |
| `badges_list` | After `get_badges` | No |
| `badge_earned` | Auto — badge threshold crossed | ✅ **Congratulations modal** |
| `streak_milestone` | Auto — streak milestone hit | ✅ Celebration |
| `leaderboard_update` | Auto — rank improved | ✅ Rank-up notification |
| `level_up` | Auto — level changed | ✅ Level-up celebration |

---

## WebSocket `initial_data` Payload

When a user connects to `ws://<host>/ws/realtime/`, the server immediately sends this payload. **Use it to hydrate all gamification UI on app launch.**

```json
{
  "type": "initial_data",
  "data": {
    "counts": {
      "unread_notifications_count": 5,
      "chat_requests_count": 2,
      "sent_requests_count": 1,
      "likes_count": 12,
      "matches_count": 3,
      "visitors_count": 8,
      "unread_support_count": 0
    },
    "gamification": {
      "streak": {
        "current_streak": 21,
        "longest_streak": 35,
        "total_check_ins": 42,
        "day_in_cycle": 2,
        "can_claim": true,
        "status": "active",
        "next_reward": { "day": 2, "coins": 15, "bonus": 0 },
        "rewards_calendar": [
          { "day": 1, "coins": 10, "bonus": 0, "status": "claimed" },
          { "day": 2, "coins": 15, "bonus": 0, "status": "current" },
          { "day": 3, "coins": 20, "bonus": 0, "status": "locked" },
          { "day": 4, "coins": 30, "bonus": 0, "status": "locked" },
          { "day": 5, "coins": 40, "bonus": 0, "status": "locked" },
          { "day": 6, "coins": 50, "bonus": 0, "status": "locked" },
          { "day": 7, "coins": 100, "bonus": 50, "status": "locked" }
        ],
        "last_check_in": "2026-04-08"
      },
      "show_daily_reward_popup": true,
      "badge_count": 8,
      "missions_completed": 3
    },
    "features": {
      "wallet": 2,
      "general": 1
    }
  }
}
```

> **Key frontend logic:** If `show_daily_reward_popup` is `true`, immediately display the 7-day reward calendar modal so the user can claim.

### Integrated Profile Statistics

When fetching the current user profile via `GET /users/profiles/me/`, the response already includes:

```json
{
  "badge_count": 8,
  "current_streak": 21,
  "missions_completed": 3,
  "daily_mission_progress": 72
}
```

No separate API call needed for profile display.

---

## Platform Ranks & Tiers

Full details → [ranks_and_thresholds.md](./ranks_and_thresholds.md)

### Subscription Tiers (4 tiers)

| Order | Key | Label | Description |
|-------|-----|-------|-------------|
| 0 | `free` | Free | Default standard account |
| 1 | `plus` | HafarPlus | Mid-tier premium |
| 2 | `premium` | Premium | Full platform access |
| 3 | `vip` | VIP | Highest exclusive tier |

### Badge Tiers (5 tiers)

| Order | Tier | Color | Visual Style |
|-------|------|-------|-------------|
| 1 | `BRONZE` | `#CD7F32` | Orange / warm tones |
| 2 | `SILVER` | `#C0C0C0` | Grey / silver hues |
| 3 | `GOLD` | `#FFD700` | Gold / yellow hues |
| 4 | `PLATINUM` | Cyan | Diamond / crystal styling |
| 5 | `SPECIAL` | Custom | Exclusive / unique styling |

### Leaderboard Types (5 types)

| Key | Label | Metric | Update Frequency |
|-----|-------|--------|-----------------|
| `most_active` | Most Active | Posts, comments, check-ins | Real-time |
| `top_streaks` | Streak Masters | Longest active streak | Daily |
| `top_referrers` | Top Ambassadors | Completed referrals | Real-time |
| `top_gifters` | Top Gifters | Gifts sent | Real-time |
| `weekly_active` | Weekly Active | Weekly engagement score | Weekly |

### Dynamic Ranks API

Use this **public** endpoint to fetch all ranks dynamically instead of hardcoding:

```
GET /ranks/
```
No authentication required.

```json
{
  "success": true,
  "subscription_ranks": [
    { "key": "free", "label": "Free", "display_order": 0 },
    { "key": "plus", "label": "HafarPlus", "display_order": 1 },
    { "key": "premium", "label": "Premium", "display_order": 2 },
    { "key": "vip", "label": "VIP", "display_order": 3 }
  ],
  "badge_tiers": ["BRONZE", "SILVER", "GOLD", "PLATINUM", "SPECIAL"],
  "leaderboard_types": [
    { "key": "most_active", "label": "Most Active" },
    { "key": "top_streaks", "label": "Streak Masters" },
    { "key": "top_referrers", "label": "Top Ambassadors" },
    { "key": "top_gifters", "label": "Top Gifters" },
    { "key": "weekly_active", "label": "Weekly Active" }
  ]
}
```

---

For detailed per-module docs, see the individual files linked above.
