# Missions API — Complete Reference

Base Endpoint: `/gamification/missions/`
Authentication: **Required** (all endpoints)

---

## How Missions Work

1. Missions are tasks that reward users with coins (and optionally badges)
2. Progress increments **automatically** via Django signals when the user performs the tracked action
3. When a mission reaches 100% progress, its status changes to `CLAIMABLE`
4. The user must **explicitly claim** the reward (via REST or WebSocket)
5. After claiming, coins are deposited into their wallet immediately

---

## Mission Types

| Type | Key | Reset Cycle | Behavior |
|:-----|:----|:-----------|:---------|
| Newcomer | `ONETIME` | Never | One-time tasks for new users (e.g., "Complete your profile") |
| Daily | `DAILY` | Every 24h | Resets at midnight; new instance created each day |
| Weekly | `WEEKLY` | Every 7 days | Resets weekly; new instance created each week |

---

## Mission Statuses

| Status | Key | Can Claim? | Frontend Action |
|:-------|:----|:----------:|:---------------|
| In Progress | `PENDING` | ✗ | Show progress bar + "Go" button (deep link) |
| Ready to Claim | `CLAIMABLE` | ✓ | Show glowing animated "Claim" CTA |
| Claimed | `CLAIMED` | ✗ | Show ✅ checkmark, greyed out |
| Expired | `EXPIRED` | ✗ | Greyed out with "Expired" label (daily/weekly) |

---

## Mission Triggers (What Auto-Increments Progress)

| Trigger Key | Display Label | When It Fires | Example Mission |
|:------------|:-------------|:-------------|:----------------|
| `profile_complete` | Complete Profile | Profile completeness ≥ 80% | "Complete your profile" |
| `upload_photo` | Upload Photo | New photo added to gallery | "Add 3 photos" |
| `voice_bio` | Record Voice Bio | Voice bio field populated | "Add a voice bio" |
| `vent_post` | Post in Vent Room | New VentPost created | "Post in the Vent Room" |
| `community_post` | Post in Community | New community Post created | "Share in the community" |
| `comment` | Leave a Comment | New VentComment created | "Leave 5 comments" |
| `receive_like` | Receive a Like | Someone likes user's content | "Get 10 likes" |
| `first_message` | Send First Message | User's 1st chat message ever | "Send your first message" |
| `send_gift` | Send a Gift | Gift created | "Send a gift" |
| `daily_login` | Daily Login | User logs in for the day | "Log in today" |
| `referral` | Refer a Friend | Referral status → COMPLETED | "Refer a friend" |
| `manual` | Manual Trigger | Admin-initiated | Special event missions |

---

## REST Endpoints

### 1. List All Missions

Get all active missions for the current user, grouped by mission type.

- **URL:** `GET /gamification/missions/`
- **Auth:** Required

**Response:**
```json
{
  "newcomer": [
    {
      "id": "550e8400-e29b-41d4-a716-446655440000",
      "slug": "add_voice_bio",
      "title": "Add a Voice Bio",
      "description": "Record a short voice introduction so matches can hear your voice",
      "coin_reward": 50,
      "target_count": 1,
      "icon_name": "mic-outline",
      "action_route": "/profile/edit",
      "progress": 0,
      "progress_percent": 0,
      "status": "PENDING",
      "can_claim": false
    },
    {
      "id": "uuid",
      "slug": "complete_profile",
      "title": "Complete Your Profile",
      "description": "Fill out your profile to at least 80% to unlock all features",
      "coin_reward": 100,
      "target_count": 1,
      "icon_name": "person-outline",
      "action_route": "/profile/edit",
      "progress": 1,
      "progress_percent": 100,
      "status": "CLAIMABLE",
      "can_claim": true
    }
  ],
  "daily": [
    {
      "id": "uuid",
      "slug": "daily_vent_post",
      "title": "Daily Vent Post",
      "description": "Share your thoughts in the Vent Room today",
      "coin_reward": 25,
      "target_count": 1,
      "icon_name": "chatbox-outline",
      "action_route": "/vent-room",
      "progress": 0,
      "progress_percent": 0,
      "status": "PENDING",
      "can_claim": false
    }
  ],
  "weekly": [
    {
      "id": "uuid",
      "slug": "weekly_community_posts",
      "title": "Community Contributor",
      "description": "Create 3 community posts this week",
      "coin_reward": 75,
      "target_count": 3,
      "icon_name": "people-outline",
      "action_route": "/community",
      "progress": 1,
      "progress_percent": 33,
      "status": "PENDING",
      "can_claim": false
    }
  ]
}
```

**Response fields per mission:**

| Field | Type | Description |
|-------|------|-------------|
| `id` | UUID | Unique mission instance ID (use for claiming) |
| `slug` | string | Machine-readable identifier |
| `title` | string | Display title |
| `description` | string | Explanatory description |
| `coin_reward` | int | Coins awarded on claim |
| `target_count` | int | How many times action must be performed |
| `icon_name` | string | Icon name for local icon rendering |
| `action_route` | string | **Deep link route** — navigate here when user taps "Go" |
| `progress` | int | Current progress count |
| `progress_percent` | int (0-100) | Pre-computed — **use directly for progress bars** |
| `status` | string | `PENDING` / `CLAIMABLE` / `CLAIMED` / `EXPIRED` |
| `can_claim` | bool | `true` when status is `CLAIMABLE` |

> **Deep link logic:** When `action_route` is `/profile/edit`, tapping "Go" should navigate the user to their profile edit screen. When it's `/vent-room`, navigate to the Vent Room, etc.

---

### 2. Claim Mission Reward

Claim the reward for a completed mission.

- **URL:** `POST /gamification/missions/{id}/claim/`
- **Auth:** Required
- **Body:** None required

**Success Response (200):**
```json
{
  "success": true,
  "coins_earned": 50,
  "badge_earned": null,
  "message": "You earned 50 coins!"
}
```

**Error Response (400):**
```json
{
  "success": false,
  "message": "Mission is not claimable"
}
```

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `success` | bool | Whether claim succeeded |
| `coins_earned` | int | Coins deposited to wallet |
| `badge_earned` | string? | Badge name if mission grants a badge, else `null` |
| `message` | string | Human-readable confirmation |

---

## WebSocket Events

### Get Missions via WebSocket

**Send:**
```json
{
  "type": "get_missions",
  "data": {}
}
```

**Receive:** `missions_list`
```json
{
  "type": "missions_list",
  "data": {
    "newcomer": [...],
    "daily": [...],
    "weekly": [...]
  }
}
```

Same format as REST endpoint #1.

### Claim Mission via WebSocket

**Send:**
```json
{
  "type": "claim_mission",
  "data": {
    "mission_id": "550e8400-e29b-41d4-a716-446655440000"
  }
}
```

**Receive on success:** `mission_claimed`
```json
{
  "type": "mission_claimed",
  "data": {
    "success": true,
    "coins_earned": 50,
    "badge_earned": null,
    "message": "You earned 50 coins!"
  }
}
```

**Receive on error:** `mission_claim_error`
```json
{
  "type": "mission_claim_error",
  "data": {
    "success": false,
    "message": "Mission is not claimable"
  }
}
```

### `mission_complete` — Server Push (Auto)

When a user's action causes a mission to reach 100% progress, the server pushes this event automatically. **Show a toast or modal prompting the user to claim.**

```json
{
  "type": "mission_complete",
  "data": {
    "mission_id": "550e8400-e29b-41d4-a716-446655440000",
    "mission_title": "Daily Vent Post",
    "coin_reward": 25
  }
}
```

> **Recommended UX:** Show a slide-up toast: "🎯 **Daily Vent Post** complete! Tap to claim 25 coins."

---

## Mission Lifecycle

```
Mission Created (PENDING, progress = 0)
        │
        ▼ User performs trigger action (e.g., creates a vent post)
        │
        ▼ Django signal fires → MissionService.update_progress()
        │
        ▼ progress increments → progress_percent recalculated
        │
        ▼ target_count reached?
        │
   ┌────┴────┐
   No        Yes
   │          │
   ▼          ▼
 Stay at    Status → CLAIMABLE
 PENDING    WS push: "mission_complete"
              │
              ▼ User taps "Claim" (REST or WebSocket)
              │
              ▼ MissionService.claim_mission()
              │
              ▼ Coins added to wallet
              │
              ▼ Status → CLAIMED
              │
              ▼ WS push: "mission_claimed"
```

### Important: New User Mission Initialization

When a new user account is created, all `ONETIME` missions are automatically initialized for them via a `post_save` signal. Daily and weekly missions are created on-the-fly when queried.
