# Community Notifications

## Overview

Community interactions trigger notifications that are delivered via:

1. **WebSocket**: For real-time in-app updates (when user is online).
2. **Push Notification (FCM)**: When user is offline.

All notifications are sent via the unified `ws/realtime/` WebSocket endpoint.

## Notification Types

The `event_type` field in the WebSocket message determines the kind of interaction.

| Event Type | Description |
|Args|Args|
| `post_like` | Someone liked your post |
| `comment` | Someone commented on your post |
| `comment_reply` | Someone replied to your comment |
| `comment_like` | Someone liked your comment |

## WebSocket Payload Structure

When a notification occurs, the server broadcasts a message with `type: "broadcast"` containing the specific event details.

### 1. Post Like

```json
{
  "type": "post_like",
  "data": {
    "post_id": "123",
    "community_id": "456",
    "liker_name": "John Doe",
    "liker_avatar": "https://..."
  },
  "title": "👍 Someone liked your post",
  "body": "John Doe liked your post in Flutter Devs"
}
```

### 2. New Comment

```json
{
  "type": "comment",
  "data": {
    "post_id": "123",
    "comment_id": "789",
    "community_id": "456",
    "commenter_name": "Jane Smith"
  },
  "title": "💬 New comment on your post",
  "body": "Jane Smith commented: This looks great..."
}
```

### 3. Reply to Comment

```json
{
  "type": "comment_reply",
  "data": {
    "post_id": "123",
    "comment_id": "790",
    "parent_id": "789",
    "replier_name": "Bob"
  },
  "title": "↩️ Reply to your comment",
  "body": "Bob replied: I agree with you!"
}
```

### 4. Comment Like

```json
{
  "type": "comment_like",
  "data": {
    "post_id": "123",
    "comment_id": "789",
    "liker_name": "Alice"
  },
  "title": "❤️ Someone liked your comment",
  "body": "Alice liked your comment"
}
```

## Handling in Frontend

1. **Connect** to `wss://api.joinhafar.com/ws/realtime/?token=...`
2. **Listen** for messages with `type` matching the event types above.
3. **Display** user-friendly toast or update notification badge.
4. **Navigate** using `data.post_id` or `data.community_id` when tapped.
