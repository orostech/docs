# Ventroom API

Base Path: `/ventroom/`

This is Hafar's anonymous safe space for venting and emotional support. It features real-time updates via WebSockets and content moderation.

---

## 🔐 Authentication
All endpoints require JWT Bearer token.

---

## Vent Posts

### 1. List Posts (Feed)
```http
GET /ventroom/posts/
```
Returns active posts, filters out hidden/reported posts.

**Query Params**
| Parameter | Type | Description |
|-----------|------|-------------|
| `category` | string | Filter by category (optional) |
| `page` | int | Page number |

**Response**
```json
{
  "count": 50,
  "results": [
    {
      "id": 1,
      "user": {
        "id": "uuid",
        "display_name": "Anonymous User",
        "is_anonymous": true
      },
      "title": "Feeling stressed",
      "content": "...",
      "category": "stress",
      "anonymous": true,
      "views": 125,
      "total_comment": 5,
      "reaction_summary": [
        {"reaction_type": "hug", "count": 10}
      ],
      "user_reaction": "hug",
      "created_at": "2026-01-07T10:00:00Z"
    }
  ]
}
```

### 2. Create Post
```http
POST /ventroom/posts/
```
**Limit**: 2 posts per day per user.

**Request Body**
```json
{
  "title": "Need to vent",
  "content": "Today was really tough...",
  "category": "stress",
  "anonymous": true
}
```

**Anonymous posts require Premium subscription.**

### 3. Trending & Popular
```http
GET /ventroom/posts/trending/       # Most views in 24h
GET /ventroom/posts/most_viewed/    # All-time most viewed
```

### 4. Post with Comments
```http
GET /ventroom/posts/{id}/with_comments/
```
Returns post details with paginated comments.

### 5. Post Limit Status
```http
GET /ventroom/posts/post_limit_status/
```
Returns user's remaining posts for today.

**Response**
```json
{
  "posts_made": 1,
  "posts_remaining": 1,
  "can_post": true,
  "limit": 2
}
```

---

## Vent Comments

### Create Comment
```http
POST /ventroom/comments/
```
**Request Body**
```json
{
  "post_id": 1,
  "text": "I feel you, stay strong!",
  "parent": null,
  "anonymous": false
}
```

### Report Comment
```http
POST /ventroom/comments/{id}/report/
```

---

## Reactions

### React to Post/Comment
```http
POST /ventroom/reactions/
```
**Request Body**
```json
{
  "content_type": "ventpost",
  "object_id": 123,
  "reaction_type": "hug"
}
```
| content_type | Description |
|--------------|-------------|
| `ventpost` | React to a post |
| `ventcomment` | React to a comment |

---

## Reports

### Report Post
```http
POST /ventroom/posts/{id}/report/
```

### Report Comment
```http
POST /ventroom/comments/{id}/report/
```

---

## 🔌 WebSocket Real-Time API

### Connection
```
wss://api.joinhafar.com/ws/realtime/
```
**Authentication**: Pass JWT token as query param or in handshake headers.

Upon connection, the server:
1. Auto-joins user to `user_{id}` personal group
2. Auto-joins `notifications` group
3. Sends `initial_data` payload with app state

---

### Message Format

**Client → Server**
```json
{
  "type": "join_ventroom",
  "data": {}
}
```

**Server → Client**
```json
{
  "type": "ventroom_joined",
  "data": {
    "message": "Successfully joined Ventroom",
    "post_limit": {
      "posts_made": 1,
      "posts_remaining": 1,
      "can_post": true,
      "limit": 2
    }
  }
}
```

---

### Client Commands (Send to Server)

| Command | Data | Description |
|---------|------|-------------|
| `join_ventroom` | `{}` | Subscribe to feed updates |
| `leave_ventroom` | `{}` | Unsubscribe from feed |
| `join_ventroom_post` | `{"post_id": 1}` | Subscribe to post's comments/reactions |
| `leave_ventroom_post` | `{"post_id": 1}` | Unsubscribe from post |
| `ping` | `{}` | Health check (returns `pong`) |

---

### Server Events (Receive from Server)

| Event Type | Trigger | Payload Contains |
|------------|---------|------------------|
| `ventroom_joined` | After join_ventroom | `post_limit` info |
| `ventroom_post_joined` | After join_ventroom_post | `post_id`, confirmation |
| `new_ventroom_post` | New post in feed | Full post object |
| `ventroom_post_updated` | Post edited | Updated post object |
| `ventroom_post_deleted` | Post removed | `post_id` |
| `new_ventroom_comment` | New comment on watched post | Comment object |
| `ventroom_comment_updated` | Comment edited | Updated comment |
| `ventroom_comment_deleted` | Comment removed | `comment_id` |
| `ventroom_reaction_updated` | Reaction changed | `reaction_summary`, `user_reaction` |
| `new_comment_on_user_post` | Someone commented on your post | Comment + post reference |
| `new_reply_on_user_comment` | Someone replied to your comment | Reply object |

---

### Channel Groups

| Group Name | Purpose |
|------------|---------|
| `ventroom_feed` | All new posts broadcast here |
| `vent_post_{id}` | Comments/reactions for specific post |
| `user_{id}` | Private notifications for user |

---

### Example: Full Flow

```javascript
// 1. Connect
const ws = new WebSocket('wss://api.joinhafar.com/ws/realtime/?token=JWT_TOKEN');

// 2. Join Ventroom feed
ws.send(JSON.stringify({ type: 'join_ventroom', data: {} }));

// 3. Listen for new posts
ws.onmessage = (event) => {
  const msg = JSON.parse(event.data);
  if (msg.type === 'new_ventroom_post') {
    console.log('New post:', msg.data);
  }
};

// 4. View specific post (get comment updates)
ws.send(JSON.stringify({ type: 'join_ventroom_post', data: { post_id: 123 } }));

// 5. Leave when done
ws.send(JSON.stringify({ type: 'leave_ventroom_post', data: { post_id: 123 } }));
```
