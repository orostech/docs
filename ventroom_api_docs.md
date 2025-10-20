# Vent Room API Documentation

## Overview
The Vent Room is a community feature where users can share posts, comment, and react anonymously or openly. It supports multimedia content (images, videos, audio) and real-time updates via WebSocket.

---

## Table of Contents
1. [Posts Management](#posts-management)
2. [Comments Management](#comments-management)
3. [Reactions Management](#reactions-management)
4. [WebSocket Events](#websocket-events)
5. [Error Responses](#error-responses)

---

## Posts Management

### 1. List All Posts
Retrieve all vent room posts (paginated).

**Endpoint:** `GET /posts/`

**Authentication:** Required

**Query Parameters:**
- `page` (optional): Page number
- `page_size` (optional): Results per page

**Success Response (200):**
```json
{
  "count": 150,
  "next": "https://api.com/ventroom/posts/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "user": {
        "id": "uuid-string",
        "display_name": "John Doe",
        "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
      },
      "title":"my boy"
      "content": "Just needed to share this...",
      "media": "https://api.joinhafar.com/media/vent_media/2025/01/video.mp4",
      "media_type": "video",
      "created_at": "2025-01-17T10:30:00Z",
      "updated_at": "2025-01-17T10:30:00Z",
      "views": 45,
      "total_comment": 12,
      "reaction_summary": [
        {
          "reaction_type": "👍",
          "count": 23
        },
        {
          "reaction_type": "❤️",
          "count": 15
        }
      ],
      "user_reaction": "👍"
    }
  ]
}
```

**Notes:**
- Results are cached for 5 minutes
- Posts are ordered by most recent first
- Cache key: `ventroom_feed_{user_id}`

---

### 2. Create Post
Create a new vent room post.

**Endpoint:** `POST /posts/`

**Authentication:** Required

**Content-Type:** `multipart/form-data` (for media uploads) or `application/json`

**Request Body (with media):**
```
title:"Tilte"
content: "Sharing my thoughts today..."
media: <file>
media_type: "image"
```

**Request Body (title and content only):**
```json
{
  "title":"my boy",
  "content": "Just needed to vent about something..."
}
```

**Media Types:**
- `image` - Image files
- `video` - Video files
- `audio` - Audio files

**Success Response (201):**
```json
{
  "id": 2,
  "user": {
    "id": "uuid-string",
    "display_name": "John Doe",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "title":"my boy",
  "content": "Sharing my thoughts today...",
  "media": "https://api.joinhafar.com/media/vent_media/2025/01/image.jpg",
  "media_type": "image",
  "created_at": "2025-01-17T11:00:00Z",
  "updated_at": "2025-01-17T11:00:00Z",
  "views": 0,
  "total_comment": 0,
  "reaction_summary": [],
  "user_reaction": null
}
```

**WebSocket Broadcast:**
When a post is created, a WebSocket event is sent to the creator:
```json
{
  "type": "new_ventroom_post",
  "data": {
    "id": 2,
    "user": {...},
    "title":"my boy",
    "content": "Sharing my thoughts today...",
    ...
  },
  "timestamp": "2025-01-17T11:00:00Z"
}
```

---

### 3. Get Single Post
Retrieve details of a specific post.

**Endpoint:** `GET /posts/{post_id}/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "id": 1,
  "user": {
    "id": "uuid-string",
    "display_name": "John Doe",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "text": "Just needed to share this...",
  "media": "https://api.joinhafar.com/media/vent_media/2025/01/video.mp4",
  "media_type": "video",
  "created_at": "2025-01-17T10:30:00Z",
  "updated_at": "2025-01-17T10:30:00Z",
  "views": 46,
  "total_comment": 12,
  "reaction_summary": [
    {
      "reaction_type": "👍",
      "count": 23
    }
  ],
  "user_reaction": "👍"
}
```

**Notes:**
- View count is automatically incremented when retrieving a post
- Each retrieval adds +1 to the views counter

---

### 4. Update Post
Update an existing post (owner or admin only).

**Endpoint:** `PUT /posts/{post_id}/` or `PATCH /posts/{post_id}/`

**Authentication:** Required (Owner or Admin)

**Request Body:**
```json
{
  "title":"my boy",
  "content": "Updated post content..."
}
```

**Success Response (200):**
```json
{
  "id": 1,
  "user": {...},
  "title":"my boy",
  "content": "Updated post content...",
  "media": "https://api.joinhafar.com/media/vent_media/2025/01/video.mp4",
  "media_type": "video",
  "created_at": "2025-01-17T10:30:00Z",
  "updated_at": "2025-01-17T12:00:00Z",
  "views": 46,
  "total_comment": 12,
  "reaction_summary": [...],
  "user_reaction": "👍"
}
```

**Error Response (403):**
```json
{
  "detail": "You do not have permission to perform this action."
}
```

---

### 5. Delete Post
Delete a post (owner or admin only).

**Endpoint:** `DELETE /posts/{post_id}/`

**Authentication:** Required (Owner or Admin)

**Success Response (204):** No content

---

## Comments Management

### 6. List Post Comments
Get all comments for a specific post.

**Endpoint:** `GET /posts/{post_id}/comments/`

**Authentication:** Required

**Success Response (200):**
```json
[
  {
    "id": 1,
    "post": 1,
    "user": {
      "id": "uuid-string",
      "display_name": "Jane Smith",
      "profile_photo": "https://api.joinhafar.com/media/photo2.jpg"
    },
    "text": "Great post! I totally agree.",
    "media": null,
    "media_type": null,
    "created_at": "2025-01-17T10:35:00Z",
    "updated_at": "2025-01-17T10:35:00Z",
    "parent": null,
    "replies": [],
    "reaction_summary": [
      {
        "reaction_type": "❤️",
        "count": 5
      }
    ],
    "user_reaction": null
  },
  {
    "id": 2,
    "post": 1,
    "user": {
      "id": "uuid-string-2",
      "display_name": "Mike Johnson",
      "profile_photo": "https://api.joinhafar.com/media/photo3.jpg"
    },
    "text": "This is a reply to the first comment",
    "media": null,
    "media_type": null,
    "created_at": "2025-01-17T10:40:00Z",
    "updated_at": "2025-01-17T10:40:00Z",
    "parent": 1,
    "replies": [],
    "reaction_summary": [],
    "user_reaction": null
  }
]
```

**Notes:**
- Comments include both top-level comments and replies
- Filter by `parent__isnull=True` to get only top-level comments
- Replies have a `parent` field pointing to the parent comment ID

---

### 7. Create Comment
Add a comment to a post or reply to another comment.

**Endpoint:** `POST /posts/{post_id}/comments/`

**Authentication:** Required

**Content-Type:** `multipart/form-data` (for media) or `application/json`

**Request Body (top-level comment):**
```json
{
  "text": "This is my comment on the post"
}
```

**Request Body (reply to comment):**
```json
{
  "text": "This is a reply",
  "parent": 1
}
```

**Request Body (with media):**
```
text: "Check this out"
media: <file>
media_type: "image"
parent: null
```

**Success Response (201):**
```json
{
  "id": 3,
  "post": 1,
  "user": {
    "id": "uuid-string",
    "display_name": "John Doe",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "text": "This is my comment on the post",
  "media": null,
  "media_type": null,
  "created_at": "2025-01-17T11:00:00Z",
  "updated_at": "2025-01-17T11:00:00Z",
  "parent": null,
  "replies": [],
  "reaction_summary": [],
  "user_reaction": null
}
```

**WebSocket Broadcast:**
```json
{
  "type": "new_ventroom_comment",
  "data": {
    "id": 3,
    "post": 1,
    "user": {...},
    "text": "This is my comment on the post",
    ...
  },
  "timestamp": "2025-01-17T11:00:00Z"
}
```

**Push Notification:**
If the commenter is not the post owner, a push notification is sent:
- **Title:** "New Comment"
- **Message:** "{User} commented on your post."

---

### 8. Update Comment
Update a comment (owner or admin only).

**Endpoint:** `PUT /posts/{post_id}/comments/{comment_id}/` or `PATCH /posts/{post_id}/comments/{comment_id}/`

**Authentication:** Required (Owner or Admin)

**Request Body:**
```json
{
  "text": "Updated comment text"
}
```

**Success Response (200):**
```json
{
  "id": 3,
  "post": 1,
  "user": {...},
  "text": "Updated comment text",
  "created_at": "2025-01-17T11:00:00Z",
  "updated_at": "2025-01-17T11:30:00Z",
  ...
}
```

---

### 9. Delete Comment
Delete a comment (owner or admin only).

**Endpoint:** `DELETE /posts/{post_id}/comments/{comment_id}/`

**Authentication:** Required (Owner or Admin)

**Success Response (204):** No content

---

## Reactions Management

### 10. Add/Update Reaction
React to a post or comment.

**Endpoint:** `POST /reactions/`

**Authentication:** Required

**Request Body:**
```json
{
  "content_type": "ventpost",
  "object_id": 1,
  "reaction_type": "👍"
}
```

**Parameters:**
- `content_type` (required): Either `ventpost` or `ventcomment`
- `object_id` (required): ID of the post or comment
- `reaction_type` (required): One of `👍`, `❤️`, `😂`, `😢`

**Success Response (201):**
```json
{
  "content_type": "ventpost",
  "object_id": 1,
  "reaction_type": "👍"
}
```

**WebSocket Broadcast:**
Real-time reaction update sent to connected clients:
```json
{
  "type":"new_ventroom_reaction_update",
  "data": {
    "type": "reaction_update",
    "content_type": "ventpost",
    "object_id": 1,
    "reaction_summary": [
      {
        "reaction_type": "👍",
        "count": 24
      },
      {
        "reaction_type": "❤️",
        "count": 15
      }
    ],
    "user_reaction": "👍"
  },
  "timestamp": "2025-01-17T11:15:00Z"
}
```

**Push Notification:**
If reacting to someone else's content:
- **Title:** "New Reaction"
- **Message:** "{User} reacted to your {content_type}."

**Notes:**
- If a user already has a reaction on the content, it will be updated (not duplicated)
- Uses `update_or_create` to ensure one reaction per user per content

**Error Response (400):**
```json
{
  "content_type": [
    "Invalid content type. Must be 'ventpost' or 'ventcomment'."
  ]
}
```

---

## WebSocket Events

### Connection
Listen to connected WebSocket server to receive real-time updates.


### Event Types

| Event Type                  | Description                                                           |
| --------------------------- | --------------------------------------------------------------------- |
| `new_ventroom_comment`      | General new comment broadcast (used internally for system-wide sync). |
| `new_comment_on_user_post`  | Someone commented on your post.                                       |
| `new_reply_on_user_comment` | Someone replied to your comment.                                      |
| `user_own_comment_created`  | You created a new comment yourself (local confirmation event).        |
| `new_ventroom_post`         | You created a new post yourself (local confirmation event).           |
| `new_ventroom_reaction_update` | Reaction update on your content (post or




#### 1. New Post Event
Triggered when a new post is created by the user.

**Event Structure:**
```json
{
  "type": "new_ventroom_post",
  "data": {
    "id": 2,
    "user": {
      "id": "uuid-string",
      "display_name": "John Doe",
      "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
    },
    "text": "New post content...",
    "media": null,
    "media_type": null,
    "created_at": "2025-01-17T11:00:00Z",
    "updated_at": "2025-01-17T11:00:00Z",
    "views": 0,
    "total_comment": 0,
    "reaction_summary": [],
    "user_reaction": null
  },
  "timestamp": "2025-01-17T11:00:00Z"
}
```

---

#### 2. New Comment Event
Triggered when a new comment is created by the user.

**Event Structure:**
```json
{
  "type":"new_ventroom_comment",
  "data": {
    "id": 3,
    "post": 1,
    "user": {
      "id": "uuid-string",
      "display_name": "John Doe",
      "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
    },
    "text": "This is my comment",
    "media": null,
    "media_type": null,
    "created_at": "2025-01-17T11:05:00Z",
    "updated_at": "2025-01-17T11:05:00Z",
    "parent": null,
    "replies": [],
    "reaction_summary": [],
    "user_reaction": null
  },
  "timestamp": "2025-01-17T11:05:00Z"
}
```

---

#### 3. Reaction Update Event
Triggered when someone reacts to content.

**Event Structure:**
```json
{
  "type": "new_ventroom_reaction_update",
  "data": {
    "type": "reaction_update",
    "content_type": "ventpost",
    "object_id": 1,
    "reaction_summary": [
      {
        "reaction_type": "👍",
        "count": 25
      },
      {
        "reaction_type": "❤️",
        "count": 16
      },
      {
        "reaction_type": "😂",
        "count": 3
      }
    ],
    "user_reaction": "👍"
  },
  "timestamp": "2025-01-17T11:10:00Z"
}
```


### Comment Fields
- `id` (integer): Unique comment identifier
- `post` (integer): ID of parent post
- `user` (object): Comment author information
- `parent` (integer/null): ID of parent comment (for replies)
- `text` (string): Comment content (required)
- `media` (string/url): Media file URL (optional)
- `media_type` (string): Type of media
- `created_at` (datetime): Creation timestamp
- `updated_at` (datetime): Last update timestamp
- `replies` (array): Nested replies to this comment
- `reaction_summary` (array): Aggregated reaction counts
- `user_reaction` (string): Current user's reaction (if any)

### Reaction Types
- `👍` - Like
- `❤️` - Love
- `😂` - Haha
- `😢` - Sad

### Media Types
- `image` - Image files (JPG, PNG, GIF, etc.)
- `video` - Video files (MP4, MOV, etc.)
- `audio` - Audio files (MP3, WAV, etc.)

---

## Best Practices

### Performance
1. **Caching:** Feed results are cached for 5 minutes
2. **Pagination:** Always use pagination for list endpoints
3. **Prefetching:** Use `select_related` and `prefetch_related` in queries

### Real-time Updates
1. **WebSocket Connection:** Maintain persistent connection for real-time updates
2. **Reconnection Logic:** Implement automatic reconnection on disconnect
3. **Event Handling:** Handle all event types appropriately

### Media Uploads
1. **File Size:** Keep media files under 10MB
2. **Formats:** Use standard formats (MP4 for video, MP3 for audio, JPG/PNG for images)
3. **Compression:** Compress media before upload for better performance

### Notifications
1. **Push Notifications:** Automatically sent for comments and reactions
2. **Notification Preferences:** Respect user notification settings
3. **Rate Limiting:** Consider notification frequency to avoid spam

---

## Rate Limiting

- **Standard endpoints:** 100 requests per minute
- **Create endpoints:** 20 requests per minute
- **Reaction endpoint:** 60 requests per minute

Exceeded rate limits will return:
```json
{
  "detail": "Request was throttled. Expected available in 45 seconds."
}
```
