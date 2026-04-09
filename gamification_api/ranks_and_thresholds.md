# Ranks, Tiers & Badge Thresholds — Complete Reference

This document is the **single source of truth** for every rank, tier, and badge threshold in the Hafar platform. Frontend teams should use this to build UI components with the correct colors, labels, and progression logic.

---

## 1. Subscription Tiers

These represent account-level access tiers controlling feature gates.

| Order | Key | Display Label | Description | Duration Options |
|:-----:|:---:|:-------------|:------------|:----------------|
| 0 | `free` | Free | Default standard user account | — |
| 1 | `plus` | HafarPlus | Mid-tier premium features | Weekly, Monthly |
| 2 | `premium` | Premium | Access to all platform features | Weekly, Monthly |
| 3 | `vip` | VIP | Highest exclusive tier | Weekly, Monthly |

### Subscription Types

| Type | Duration | Key |
|------|----------|-----|
| Weekly | 7 days | `WEEKLY` |
| Monthly | 30 days | `MONTHLY` |

### Payment Methods

| Method | Description |
|--------|-------------|
| `FLUTTERWAVE` | Card payment via Flutterwave |
| `REVENUECAT` | In-app purchase via RevenueCat |
| `ADMIN_GRANT` | Manually assigned by admin |

### Subscription Statuses

| Status | Description |
|--------|-------------|
| `ACTIVE` | Currently active subscription |
| `EXPIRED` | Subscription period ended |
| `CANCELLED` | User cancelled subscription |
| `TRIAL` | Free trial period |

---

## 2. Badge Tiers

Every badge has a tier representing its rarity and prestige. Use these for styling badge cards, borders, and glow effects.

| Prestige | Tier Key | Display Name | Recommended Color | CSS / Design Notes |
|:--------:|:---------|:------------|:-------------------|:-------------------|
| 1 | `BRONZE` | Bronze | `#CD7F32` | Warm orange tones, matte finish |
| 2 | `SILVER` | Silver | `#C0C0C0` | Cool grey, subtle metallic sheen |
| 3 | `GOLD` | Gold | `#FFD700` | Rich gold, prominent glow |
| 4 | `PLATINUM` | Platinum | `#00CED1` (Cyan) | Diamond/crystal visual, shimmer animation |
| 5 | `SPECIAL` | Special | Custom per badge | Unique exclusive styling, animated border |

---

## 3. Badge Triggers

These are the trigger keys that the backend uses to auto-award badges. Understanding these helps you anticipate which user action leads to which badge.

| Trigger Key | Display Label | Category |
|:------------|:-------------|:---------|
| `streak_7` | 7-Day Streak | Streak |
| `streak_30` | 30-Day Streak | Streak |
| `streak_100` | 100-Day Streak | Streak |
| `posts_10` | 10 Posts | Engagement |
| `posts_50` | 50 Posts | Engagement |
| `posts_100` | 100 Posts | Engagement |
| `referrals_5` | 5 Referrals | Ambassador |
| `referrals_10` | 10 Referrals | Ambassador |
| `referrals_25` | 25 Referrals | Ambassador |
| `profile_complete` | Profile Completed | Action |
| `voice_bio` | Voice Bio Added | Action |
| `first_gift` | First Gift Sent | Action |
| `premium` | Premium Member | Premium |
| `verified` | Verified User | Premium |
| `manual` | Manual Award | Admin |

---

## 4. Complete Badge Catalog

The table below lists **every badge** defined in the system with its threshold, tier, and coin reward. This is what the frontend should render in the "All Badges" library screen.

| Category | Tier | Badge Slug | Badge Name | Trigger | Threshold | Coin Bonus | How to Earn |
|:---------|:-----|:-----------|:-----------|:--------|:---------:|:----------:|:------------|
| **Streak** | BRONZE | `streak_7` | 7-Day Streak | `streak_7` | 7 days | — | Check in 7 consecutive days |
| **Streak** | SILVER | `streak_30` | 30-Day Streak | `streak_30` | 30 days | — | Check in 30 consecutive days |
| **Streak** | GOLD | `streak_100` | 100-Day Streak | `streak_100` | 100 days | — | Check in 100 consecutive days |
| **Engagement** | BRONZE | `posts_10` | Voice Starter | `posts_10` | 10 posts | — | Create 10 posts (Vent + Community) |
| **Engagement** | SILVER | `posts_50` | Content Creator | `posts_50` | 50 posts | — | Create 50 posts total |
| **Engagement** | GOLD | `posts_100` | Community Voice | `posts_100` | 100 posts | — | Create 100 posts total |
| **Ambassador** | BRONZE | `referrals_5` | Bronze Ambassador | `referrals_5` | 5 referrals | 500 | Refer 5 active users |
| **Ambassador** | SILVER | `referrals_10` | Silver Ambassador | `referrals_10` | 10 referrals | 1,000 | Refer 10 active users |
| **Ambassador** | GOLD | `referrals_25` | Gold Ambassador | `referrals_25` | 25 referrals | 2,500 | Refer 25 active users |
| **Action** | BRONZE | `profile_complete` | Profile Pro | `profile_complete` | 80% complete | — | Complete profile to 80%+ |
| **Action** | BRONZE | `voice_bio` | Voice Identity | `voice_bio` | 1 recording | — | Add a voice bio |
| **Action** | BRONZE | `first_gift` | Generous Heart | `first_gift` | 1 gift | — | Send your first gift |
| **Premium** | PLATINUM | `premium` | Premium Member | `premium` | 1 subscription | — | Subscribe to any premium plan |
| **Premium** | SPECIAL | `verified` | Verified User | `verified` | — | — | Get verified on the platform |

> **Note:** Coin bonuses are configurable via admin. The values above are defaults. Always use the `coin_bonus` field from the API response.

---

## 5. Leaderboard Types

| Key | Display Label | What It Ranks | Score Metric | Update Cadence |
|:----|:-------------|:-------------|:-------------|:---------------|
| `most_active` | Most Active | Overall platform activity | Posts + comments + check-ins | Real-time |
| `top_streaks` | Streak Masters | Longest active streak | Current streak days | Daily |
| `top_referrers` | Top Ambassadors | Most successful referrals | Completed referral count | Real-time |
| `top_gifters` | Top Gifters | Most generous users | Total gifts sent | Real-time |
| `weekly_active` | Weekly Active | Weekly engagement window | Weekly activity score | Weekly reset |

### Leaderboard Query Filters

When calling `GET /gamification/leaderboard/`:

| Parameter | Type | Default | Options |
|-----------|------|---------|---------|
| `period` | string | `weekly` | `daily`, `weekly`, `monthly`, `all-time` |
| `type` | string | `global` | `global`, `active`, `streaks`, `referrers` |
| `limit` | int | `50` | Any positive integer |

### Type Mapping

| `type` query value | Maps to Backend Key |
|:-------------------|:-------------------|
| `global` | `most_active` |
| `active` | `most_active` |
| `streaks` | `top_streaks` |
| `referrers` | `top_referrers` |

---

## 6. Mission Types

| Type Key | Display Name | Reset Cycle | Behavior |
|:---------|:------------|:-----------|:---------|
| `ONETIME` | One-Time (Newcomer) | Never | Completed once, stays forever |
| `DAILY` | Daily | Every 24 hours | Resets at midnight, new progress |
| `WEEKLY` | Weekly | Every 7 days | Resets weekly, new progress |

### Mission Statuses

| Status | Display | Can Claim? | UI Action |
|:-------|:--------|:----------:|:----------|
| `PENDING` | In Progress | ✗ | Show progress bar |
| `CLAIMABLE` | Ready to Claim | ✓ | Show glowing "Claim" button |
| `CLAIMED` | Claimed | ✗ | Show checkmark ✅ |
| `EXPIRED` | Expired | ✗ | Greyed out (daily/weekly only) |

### Mission Triggers

These are the actions that auto-increment mission progress:

| Trigger Key | Display Label | When It Fires |
|:------------|:-------------|:--------------|
| `profile_complete` | Complete Profile | Profile completeness ≥ 80% |
| `upload_photo` | Upload Photo | Photo added to gallery |
| `voice_bio` | Record Voice Bio | Voice bio field populated |
| `vent_post` | Post in Vent Room | New VentPost created |
| `community_post` | Post in Community | New community Post created |
| `comment` | Leave a Comment | New VentComment created |
| `receive_like` | Receive a Like | Someone likes your content |
| `first_message` | Send First Message | User's 1st chat message ever |
| `send_gift` | Send a Gift | Gift sent (1st time triggers badge too) |
| `daily_login` | Daily Login | User logs in for the day |
| `referral` | Refer a Friend | Referral status → COMPLETED |
| `manual` | Manual Trigger | Admin-initiated |

---

## 7. Daily Reward Cycle (7-Day Calendar)

The streak system uses a repeating 7-day cycle. Every 7 days the cycle restarts with escalating rewards.

| Day in Cycle | Coins | Bonus | Total | Visual Indicator |
|:------------:|:-----:|:-----:|:-----:|:----------------|
| 1 | 10 | 0 | **10** | Small coin icon |
| 2 | 15 | 0 | **15** | Small coin icon |
| 3 | 20 | 0 | **20** | Medium coin icon |
| 4 | 30 | 0 | **30** | Medium coin icon |
| 5 | 40 | 0 | **40** | Large coin icon |
| 6 | 50 | 0 | **50** | Large coin icon |
| 7 | 100 | 50 | **150** | 🎉 **JACKPOT** — Treasure chest animation |

> **Total per full cycle:** 265 + 50 bonus = **315 coins**

### Calendar Status Values

| Status | Meaning | UI |
|:-------|:--------|:---|
| `claimed` | Already claimed for this day | ✅ Checkmark, greyed out |
| `current` | Today's reward, ready to claim | 🟡 Glowing, animated |
| `locked` | Future day, not yet reachable | 🔒 Locked, dimmed |

---

## 8. Referral Milestones

| Referrals Needed | Reward | Badge Earned |
|:----------------:|:-------|:-------------|
| 5 | 500 coins + Badge | Bronze Ambassador |
| 10 | 1,000 coins + Badge | Silver Ambassador |
| 25 | 2,500 coins + Badge | Gold Ambassador |
| 50 | Badge only | Elite Ambassador |

---

## 9. Fetching This Data Dynamically

**Don't hardcode any of these values.** Use the public `/ranks/` endpoint to fetch everything at app startup:

```
GET /ranks/
```

This returns subscription tiers, badge tiers, and leaderboard types directly from the database, so any admin changes are immediately reflected in your app. See the [README](./README.md) for the full response format.
