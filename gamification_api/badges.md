# Badges API

Base Endpoint: `/gamification/badges/`

Badges represent user achievements such as completing 50 Vent Room posts, checking in for 100 consecutive days, or reaching "Elite Ambassador" status natively by referring 50 active users.

## Endpoints

### 1. Unearned Badges "Progress"
Get the exact percentage toward unlocking all unearned badges.

- **URL**: `/gamification/badges/progress/`
- **Method**: `GET`
- **Auth**: Required

**Response**
```json
[
  {
    "slug": "prolific_poster",
    "name": "Prolific Poster",
    "description": "50 posts total across the platform.",
    "tier": "GOLD",
    "icon_name": "trophy-gold",
    "color": "#FFD700",
    "threshold": 50,
    "current_value": 35,
    "progress_percent": 70, // Render a progress bar using this exactly!
    "coin_bonus": 2500
  }
]
```

### 2. My Badges
Get the current user's earned badges.

- **URL**: `/gamification/badges/`
- **Method**: `GET`
- **Auth**: Required

### 3. All Badges Library
Get all available badges across the system.

- **URL**: `/gamification/badges/all/`
- **Method**: `GET`

### 4. Fetch Badges for User Profile
Get earned badges for another specific user's public view.

- **URL**: `/gamification/badges/user/{user_id}/`
- **Method**: `GET`

---

## Real-Time "Congratulations" Modals (WebSockets)

Whenever an action inside the app breaches a Badge Threshold, the server emits a real-time event that requires a celebratory popup modal! All rich visual fields are returned over the socket.

**Listen for event:** `badge_earned`
```json
{
    "type": "badge_earned",
    "data": {
        "badge_name": "Prolific Poster",
        "badge_tier": "GOLD",
        "badge_icon_name": "trophy-gold",
        "badge_color": "#FFD700",
        "badge_description": "You completed 50 posts!",
        "coin_bonus": 500
    }
}
```
