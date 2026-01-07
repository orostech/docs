# Ventroom API

Base Path: `/ventroom/`

## Vent Posts

### 1. List Posts (Feed)
`GET /ventroom/posts/`
Returns active posts. Filters out hidden posts.

**Query Params**
- `category`: string (optional) - Filter by category

### 2. Create Post
`POST /ventroom/posts/`
Limit: 2 posts per day per user.

**Body**
```json
{
  "title": "Title",
  "content": "Venting...",
  "category": "stress",
  "anonymous": true
}
```

### 3. Special Lists
- `GET /ventroom/posts/trending/` (Most views last 24h)
- `GET /ventroom/posts/most_viewed/` (All time)

### 4. Post Details with Comments
`GET /ventroom/posts/{id}/with_comments/`

Returns post + paginated comments.

### 5. Mark Viewed (Telemetry)
`POST /ventroom/posts/mark_as_viewed/`
Body: `{"post_ids": [1, 2, 3]}`

## Vent Comments

### Create Comment
`POST /ventroom/comments/`
Body: `{"post_id": 1, "text": "...", "parent": null}`

## Reactions

### React to Post/Comment
`POST /ventroom/reactions/`

**Body**
```json
{
  "content_type": "ventpost" | "ventcomment",
  "object_id": 123,
  "reaction_type": "hug"
}
```

## Reports

### Report Post
`POST /ventroom/posts/{id}/report/`

### Report Comment
`POST /ventroom/comments/{id}/report/`

## WebSocket Events
The Ventroom relies heavily on real-time updates via Django Channels.

### Subscription Channels
- `ventroom_feed`: Public feed updates.
- `vent_post_{id}`: Updates for a specific post (comments, reactions).
- `user_{id}`: Private notifications.

### Event Types
- `new_ventroom_post`: A new post appeared in the feed.
- `ventroom_post_updated` / `deleted`
- `new_ventroom_comment`: A new comment on a post you are watching.
- `ventroom_reaction_updated`: Reaction counts changed.
