# Referrals API — Complete Reference

Base Endpoint: `/referral/`
Authentication: **Required** (all endpoints)

---

## How the Referral System Works

1. Every user gets a unique **referral link** containing their referral code
2. When a new user signs up using that link, a `Referral` record is created with status `PENDING`
3. Once the referred user completes onboarding (verifies email, etc.), the status changes to `COMPLETED`
4. On `COMPLETED`, the referrer earns **50 coins** per referral
5. Hitting referral milestones (5, 10, 25, 50) awards **Ambassador badges** with bonus coins
6. Referrers are ranked on the **Top Ambassadors** leaderboard

---

## Referral Milestones & Rewards

| Milestone | Referrals Needed | Coin Reward | Badge Earned | Badge Tier |
|:---------:|:----------------:|:-----------:|:-------------|:-----------|
| 1st | 5 completed | 500 | Bronze Ambassador | BRONZE |
| 2nd | 10 completed | 1,000 | Silver Ambassador | SILVER |
| 3rd | 25 completed | 2,500 | Gold Ambassador | GOLD |
| 4th | 50 completed | — | Elite Ambassador | SPECIAL |

> **Per-referral reward:** 50 coins per completed referral (in addition to milestones)

---

## Referral Statuses

| Status | Meaning |
|:-------|:--------|
| `PENDING` | Referred user signed up but hasn't completed onboarding |
| `COMPLETED` | Referred user fully onboarded — coins awarded to referrer |

---

## REST Endpoints

### 1. Referral Summary & Milestones

Get the user's referral stats, their invite link, and progress toward milestone badges.

- **URL:** `GET /referral/summary/`
- **Auth:** Required

**Response:**
```json
{
  "total_referrals": 14,
  "total_earnings": 12426,
  "referral_link": "https://www.joinhafar.com/?ref=U193JD",
  "milestones": [
    {
      "count": 5,
      "reward": "Bronze Ambassador badge + 500 coins",
      "reached": true
    },
    {
      "count": 10,
      "reward": "Silver Ambassador badge + 1000 coins",
      "reached": true
    },
    {
      "count": 25,
      "reward": "Gold Ambassador badge + 2500 coins",
      "reached": false
    },
    {
      "count": 50,
      "reward": "Elite Ambassador badge",
      "reached": false
    }
  ],
  "next_milestone": {
    "count": 25,
    "reward": "Gold Ambassador badge + 2500 coins",
    "progress_percent": 56
  }
}
```

**Response fields:**

| Field | Type | Description |
|-------|------|-------------|
| `total_referrals` | int | Total completed referrals |
| `total_earnings` | int | Total coins earned from all referrals |
| `referral_link` | string | Shareable invite URL |
| `milestones` | array | All milestone thresholds with reached status |
| `milestones[].count` | int | Number of referrals needed |
| `milestones[].reward` | string | Human-readable reward description |
| `milestones[].reached` | bool | Whether user has hit this milestone |
| `next_milestone` | object | Next unreached milestone |
| `next_milestone.count` | int | Referrals needed for next milestone |
| `next_milestone.reward` | string | What user will earn |
| `next_milestone.progress_percent` | int (0-100) | Progress toward next milestone |

> **UI tips:**
> - Show `referral_link` with a "Copy" and "Share" button
> - Render milestones as a progress timeline with checkmarks for `reached: true`
> - Use `next_milestone.progress_percent` for a progress bar to the next badge

---

### 2. Referral List

Get all users referred by the current user.

- **URL:** `GET /referral/list/`
- **Auth:** Required
- **Pagination:** `page`, `page_size`

**Response** — Paginated list:
```json
{
  "count": 14,
  "next": "https://api.joinhafar.com/referral/list/?page=2",
  "previous": null,
  "results": [
    {
      "username": "InvitedUser123",
      "coins_earned": 50,
      "status": "COMPLETED",
      "created_at": "2026-03-31T00:00:00Z"
    },
    {
      "username": "NewUser456",
      "coins_earned": 0,
      "status": "PENDING",
      "created_at": "2026-04-05T14:30:00Z"
    }
  ]
}
```

**Response fields per entry:**

| Field | Type | Description |
|-------|------|-------------|
| `username` | string | Referred user's username |
| `coins_earned` | int | Coins earned from this specific referral |
| `status` | string | `PENDING` or `COMPLETED` |
| `created_at` | ISO datetime | When the referral was created |

---

## Badge Integration

Referral milestones automatically trigger badge awards via Django signals:

| Completed Referrals | Signal Fires | Badge Service Call | Badge Slug |
|:-------------------:|:-------------|:-------------------|:-----------|
| ≥ 5 | `on_referral_completed` | `check_referral_badges()` | `referrals_5` |
| ≥ 10 | `on_referral_completed` | `check_referral_badges()` | `referrals_10` |
| ≥ 25 | `on_referral_completed` | `check_referral_badges()` | `referrals_25` |

When a badge is earned, the user receives a `badge_earned` WebSocket event. See [badges.md](./badges.md).

---

## Leaderboard Integration

Referral counts feed directly into the **Top Ambassadors** leaderboard (`top_referrers`).

- View the leaderboard: `GET /gamification/leaderboard/referrers/`
- See your rank: `GET /gamification/leaderboard/my-ranks/` → check the `referrers` field

See [leaderboard.md](./leaderboard.md) for full details.

---

## Referral Flow Diagram

```
Existing User (Referrer)
        │
        ▼ Shares referral_link
        │
New User clicks link & signs up
        │
        ▼ Referral record created (status: PENDING)
        │
New User completes onboarding
        │
        ▼ Status → COMPLETED
        │
        ├──► 50 coins added to Referrer's wallet
        │
        ├──► MissionService.update_progress(REFERRAL)
        │    └── Mission progress incremented
        │
        └──► BadgeService.check_referral_badges()
             ├── ≥5?  → Award Bronze Ambassador
             ├── ≥10? → Award Silver Ambassador
             └── ≥25? → Award Gold Ambassador
                       └── WS push: "badge_earned"
```
