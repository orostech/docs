# Referrals API

Base Endpoint: `/referral/`

Tracks user-driven app growth, managing invitation links, referral coin issuance, and the Ambassador Badge Milestones directly corresponding to the `Referrers` Leaderboard.

## Endpoints

### 1. Referral Stats Summary & Milestones
Fetches aggregate invite earnings, their invite string, and specific progression thresholds toward Ambassador Badges. 

- **URL**: `/referral/summary/`
- **Method**: `GET`
- **Auth**: Required

**Response**
```json
{
  "total_referrals": 14,
  "total_earnings": 12426,
  "referral_link": "https://www.joinhafar.com/?ref=U193JD",
  "milestones": [
    {"count": 5, "reward": "Bronze Ambassador badge + 500 coins", "reached": true},
    {"count": 10, "reward": "Silver Ambassador badge + 1000 coins", "reached": true},
    {"count": 25, "reward": "Gold Ambassador badge + 2500 coins", "reached": false},
    {"count": 50, "reward": "Elite Ambassador badge", "reached": false}
  ],
  "next_milestone": {"count": 25, "reward": "Gold Ambassador badge + 2500 coins", "progress_percent": 56}
}
```

### 2. User Referral List
Fetches all members currently sourced via their profile code.

- **URL**: `/referral/list/`
- **Method**: `GET`
- **Auth**: Required
- **Pagination Options**: `page`, `page_size`

**Response**
Wait for paginated records containing:
```json
{
  "username": "InvitedUser123",
  "coins_earned": 50,
  "status": "COMPLETED",
  "created_at": "2026-03-31T00:00:00Z"
}
```
