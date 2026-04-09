# Ranks & Badge Thresholds

This document outlines the ranks, subscription tiers, leaderboard types, and badge thresholds available in the Gamification system.

## Rankings & Tiers

The user ecosystem is organized around several rankings and tiers. Below are the tables outlining the system's structural ranks.

### Leaderboard Rankings
These are the different global categories by which users are ranked across the platform.

| Key | Label | Description | Updating Frequency |
| :--- | :--- | :--- | :--- |
| `most_active` | Most Active | Overall activity (posts, comments, check-ins) | Real-time |
| `top_streaks` | Streak Masters | Sorted by the longest active daily check-in streak | Daily |
| `top_referrers` | Top Ambassadors | Sorted by the highest number of active referred users | Real-time |
| `top_gifters` | Top Gifters | Users who have sent the most gifts on the platform | Real-time |
| `weekly_active` | Weekly Active | A shorter window for weekly engagement rankings | Weekly |

### Subscription Tiers
These are the account-level subscription ranks dictating feature access.

| Display Order | Key | Label | Description |
| :--- | :--- | :--- | :--- |
| `0` | `free` | Free | The default standard user account |
| `1` | `plus` | HafarPlus | Middle-tier premium features |
| `2` | `premium` | Premium | Access to all platform features |
| `3` | `vip` | VIP | The highest exclusive tier |

### Badge Ranking Tiers
Every badge earned corresponds to a specific rarity tier, ordered below by prestige.

| Prestige Order | Tier Name | Color Scheme |
| :--- | :--- | :--- |
| `1` | BRONZE | `#CD7F32` / Orange / Basic colors |
| `2` | SILVER | `#C0C0C0` / Grey / Silver hues |
| `3` | GOLD | `#FFD700` / Gold / Yellow hues |
| `4` | PLATINUM | Cyan / Diamond visual styling |
| `5` | SPECIAL | Exclusive / Custom styling |

---

## Static Badge Thresholds

Badges are awarded automatically when users hit specific milestones (thresholds). Below is the comprehensive table of active triggers and their exact required thresholds (including the highest caps).

| Badge Class | Tier | Badge Name | Threshold Required | Highest Cap? |
| :--- | :--- | :--- | :--- | :--- |
| **Streak** | BRONZE | 7-Day Streak | `7` days | No |
| **Streak** | SILVER | 30-Day Streak | `30` days | No |
| **Streak** | GOLD | 100-Day Streak | `100` days | **Yes** |
| **Engagement** | BRONZE | Voice Starter | `10` posts | No |
| **Engagement** | SILVER | Content Creator | `50` posts | No |
| **Engagement** | GOLD | Community Voice | `100` posts | **Yes** |
| **Ambassador** | BRONZE | Bronze Ambassador | `5` referrals | No |
| **Ambassador** | SILVER | Silver Ambassador | `10` referrals | No |
| **Ambassador** | GOLD | Gold Ambassador | `25` referrals | **Yes** |
| **Action** | BRONZE | Profile Pro | `80`% profile complete | **Yes** |
| **Action** | BRONZE | Voice Identity | `1` voice bio added | **Yes** |
| **Action** | BRONZE | Generous Heart | `1` first gift sent | **Yes** |
| **Premium** | PLATINUM | Premium Member | `1` premium sub | **Yes** |

---

## Dynamic Ranks API

The frontend can dynamically fetch all available ranks, badge tiers, and leaderboard categories directly using a dedicated configuration endpoint instead of hard-coding them.

- **URL**: `/configs/ranks/`
- **Method**: `GET`
- **Auth**: None Required (Public)

**Successful Response**
```json
{
  "success": true,
  "subscription_ranks": [
    { "key": "free", "label": "Free", "display_order": 0 },
    { "key": "plus", "label": "HafarPlus", "display_order": 1 },
    { "key": "premium", "label": "Premium", "display_order": 2 },
    { "key": "vip", "label": "VIP", "display_order": 3 }
  ],
  "badge_tiers": [
    "BRONZE", "SILVER", "GOLD", "PLATINUM", "SPECIAL"
  ],
  "leaderboard_types": [
    { "key": "most_active", "label": "Most Active" },
    { "key": "top_streaks", "label": "Streak Masters" },
    { "key": "top_referrers", "label": "Top Ambassadors" },
    { "key": "top_gifters", "label": "Top Gifters" },
    { "key": "weekly_active", "label": "Weekly Active" }
  ]
}
```
