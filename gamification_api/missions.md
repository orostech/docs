# Missions API

Base Endpoint: `/gamification/missions/`

Missions run in the background (One-Time Newcomer, Daily recurring, or Weekly recurring). They carry an `action_route` which serves as a deep link destination when the user taps "Go". Progress increments automatically alongside triggers and user state updates.

## Endpoints

### 1. List Missions
Get all active missions grouped by categorization. 

- **URL**: `/gamification/missions/`
- **Method**: `GET`
- **Auth**: Required

**Response**
```json
{
  "newcomer": [
    {
      "id": "uuid",
      "slug": "add_voice_bio",
      "title": "Add a Voice Bio",
      "description": "...",
      "coin_reward": 50,
      "target_count": 1,
      "icon_name": "mic-outline",
      "action_route": "/profile/edit", // Deep link logic!
      "progress": 0,
      "progress_percent": 0,
      "status": "PENDING", // PENDING, CLAIMABLE, CLAIMED
      "can_claim": false
    }
  ],
  "daily": [ ... ],
  "weekly": [ ... ]
}
```

### 2. Claim Mission Reward (REST)
Once `status == "CLAIMABLE"`, the user receives the reward explicitly by hitting this endpoint.

- **URL**: `/gamification/missions/{id}/claim/`
- **Method**: `POST`
- **Auth**: Required

**Response**
```json
{
    "success": true,
    "coins_earned": 50,
    "badge_earned": null,
    "message": "You earned 50 coins!"
}
```

---

## Real-Time Tracking (WebSockets)

While exploring the App, missions natively update. E.g. when you post a Vent Room Thread, you instantly receive a "Mission Complete" alert to claim rewards.

**Listen for event:** `mission_complete`
```json
{
    "type": "mission_complete",
    "data": {
        "mission_id": "uuid",
        "mission_title": "Daily Vent Post",
        "coin_reward": 25
    }
}
```

**Claim Mission (WebSocket)**
```json
{
    "type": "claim_mission",
    "data": {
        "mission_id": "uuid"
    }
}
```
