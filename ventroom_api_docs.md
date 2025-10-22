# Vent Room API Documentation

## Overview
The Vent Room is a real-time community feature where users can share posts, comment, and react. It supports multimedia content (images, videos, audio) and provides real-time updates via WebSocket connections for an interactive experience.

---

## Table of Contents
1. [WebSocket Connection](#websocket-connection)
2. [Posts Management](#posts-management)
3. [Comments Management](#comments-management)
4. [Reactions Management](#reactions-management)
5. [WebSocket Events Reference](#websocket-events-reference)
6. [Error Responses](#error-responses)
7. [Best Practices](#best-practices)

---

## WebSocket Connection

### Connecting to Vent Room
To receive real-time updates, establish a WebSocket connection to the server.

**WebSocket URL:** `wss://api.joinhafar.com/ws/`

**Authentication:** Required (JWT token in connection params or headers)

### Joining the Vent Room Feed

After connecting, join the general Vent Room feed to receive updates about new posts.

**Client sends:**
```json
{
  "type": "join_ventroom"
}
```

**Server response:**
```json
{
  "type": "ventroom_joined",
  "data": {
    "message": "Successfully joined Ventroom",
    "user_id": "uuid-string"
  },
  "timestamp": "2025-01-17T10:00:00Z"
}
```

### Leaving the Vent Room Feed

**Client sends:**
```json
{
  "type": "leave_ventroom"
}
```

### Joining a Specific Post

To receive real-time updates for a specific post (comments, reactions, etc.), join the post channel.

**Client sends:**
```json
{
  "type": "join_ventroom_post",
  "data": {
    "post_id": 1
  }
}
```

**Server response:**
```json
{
  "type": "ventroom_post_joined",
  "data": {
    "post_id": 1,
    "message": "Successfully joined Ventroom post"
  },
  "timestamp": "2025-01-17T10:01:00Z"
}
```

### Leaving a Specific Post

**Client sends:**
```json
{
  "type": "leave_ventroom_post",
  "data": {
    "post_id": 1
  }
}
```

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
title: "My Story"
content: "Sharing my thoughts today..."
media: <file>
media_type: "image"
```

**Request Body (text only):**
```json
{
  "title": "My Thoughts",
  "content": "Just needed to vent about something..."
}
```

**Valid Media Types:**
- `image` - Image files (JPG, PNG, GIF, etc.)
- `video` - Video files (MP4, MOV, etc.)
- `audio` - Audio files (MP3, WAV, etc.)

**Success Response (201):**
```json
{
  "id": 2,
  "user": {
    "id": "uuid-string",
    "display_name": "John Doe",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "title": "My Story",
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

**Real-time Broadcast:**

When a post is created, two WebSocket events are sent:

1. **To the ventroom_feed group (all users in Ventroom):**
```json
{
  "type": "new_ventroom_post",
  "data": {
    "id": 2,
    "user": {...},
    "title": "My Story",
    "content": "Sharing my thoughts today...",
    "media": "https://api.joinhafar.com/media/vent_media/2025/01/image.jpg",
    "media_type": "image",
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

2. **To the post creator (user_{user_id} group):**
```json
{
  "type": "new_ventroom_post",
  "data": {
    "id": 2,
    "user": {...},
    "title": "My Story",
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
  "title": "Updated Title",
  "content": "Updated post content..."
}
```

**Success Response (200):**
```json
{
  "id": 1,
  "user": {...},
  "title": "Updated Title",
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

**Real-time Broadcast:**

Two events are sent when a post is updated:

1. **To vent_post_{post_id} group:**
```json
{
  "type": "ventroom_post_updated",
  "data": {
    "id": 1,
    "user": {...},
    "title": "Updated Title",
    "content": "Updated post content...",
    ...
  },
  "timestamp": "2025-01-17T12:00:00Z"
}
```

2. **To ventroom_feed group:**
```json
{
  "type": "ventroom_post_updated",
  "data": {...},
  "timestamp": "2025-01-17T12:00:00Z"
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

---

## Comments Management

### 6. List Post Comments
Get all comments for a specific post, including replies.

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
    "replies": [
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
    ],
    "reaction_summary": [
      {
        "reaction_type": "❤️",
        "count": 5
      }
    ],
    "user_reaction": null
  }
]
```

**Notes:**
- Returns all comments ordered by most recent first
- Top-level comments have `parent: null`
- Replies have `parent` set to the parent comment ID
- Comments include nested `replies` array

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

**Real-time Broadcasts:**

When a comment is created, multiple WebSocket events are sent:

1. **To vent_post_{post_id} group (all users viewing the post):**
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

2. **To the comment creator (user_{user_id} group):**
```json
{
  "type": "new_ventroom_comment",
  "data": {...},
  "timestamp": "2025-01-17T11:00:00Z"
}
```

3. **If replying to a comment - to parent comment owner:**
```json
{
  "type": "new_reply_on_user_comment",
  "data": {
    "id": 3,
    "post": 1,
    "user": {...},
    "text": "This is a reply",
    "parent": 1,
    ...
  },
  "timestamp": "2025-01-17T11:00:00Z"
}
```

4. **If commenting on someone else's post - to post owner:**
```json
{
  "type": "new_comment_on_user_post",
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

**Push Notifications:**

- **When replying to a comment:** Parent comment owner receives: "{User} replied to your comment."
- **When commenting on a post:** Post owner receives: "{User} commented on your post."

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
  "media": null,
  "media_type": null,
  "created_at": "2025-01-17T11:00:00Z",
  "updated_at": "2025-01-17T11:30:00Z",
  "parent": null,
  "replies": [],
  "reaction_summary": [],
  "user_reaction": null
}
```

**Real-time Broadcast:**

Sent to vent_post_{post_id} group:
```json
{
  "type": "ventroom_comment_updated",
  "data": {
    "id": 3,
    "post": 1,
    "user": {...},
    "text": "Updated comment text",
    ...
  },
  "timestamp": "2025-01-17T11:30:00Z"
}
```

---

### 9. Delete Comment
Delete a comment (owner or admin only).

**Endpoint:** `DELETE /posts/{post_id}/comments/{comment_id}/`

**Authentication:** Required (Owner or Admin)

**Success Response (204):** No content

**Real-time Broadcast:**

Sent to vent_post_{post_id} group:
```json
{
  "type": "ventroom_comment_deleted",
  "data": {
    "comment_id": 3,
    "post_id": 1
  },
  "timestamp": "2025-01-17T11:45:00Z"
}
```

---

## Reactions Management

### 10. Add/Update Reaction
React to a post or comment. If the user already has a reaction, it will be updated.

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

**Real-time Broadcasts:**

When a reaction is added/updated, multiple WebSocket events are sent:

1. **To vent_post_{post_id} group (all users viewing the post):**
```json
{
  "type": "ventroom_reaction_updated",
  "data": {
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
    "user_reaction": "👍",
    "user_id": "uuid-string"
  },
  "timestamp": "2025-01-17T11:15:00Z"
}
```

2. **To the reactor (user_{user_id} group):**
```json
{
  "type": "ventroom_reaction_updated",
  "data": {...},
  "timestamp": "2025-01-17T11:15:00Z"
}
```

3. **To the content owner (if different from reactor):**
```json
{
  "type": "new_reaction_on_user_post",
  "data": {
    "content_type": "ventpost",
    "object_id": 1,
    "reaction_summary": [...],
    "user_reaction": "👍",
    "user_id": "uuid-string"
  },
  "timestamp": "2025-01-17T11:15:00Z"
}
```
*Note: Event type changes to `new_reaction_on_user_comment` if reacting to a comment.*

**Push Notification:**

If reacting to someone else's content:
- **Title:** "New Reaction"
- **Message:** "{User} reacted to your {post/comment}."

**Notes:**
- Uses `update_or_create` - only one reaction per user per content
- Changing reaction type updates the existing reaction rather than creating a new one

**Error Response (400):**
```json
{
  "content_type": [
    "Invalid content type. Must be 'ventpost' or 'ventcomment'."
  ]
}
```

---

## WebSocket Events Reference

### Connection Management Events

| Event Type | Direction | Description |
|------------|-----------|-------------|
| `ventroom_joined` | Server → Client | Confirmation of joining Ventroom feed |
| `ventroom_post_joined` | Server → Client | Confirmation of joining a specific post |

### Post Events

| Event Type | Direction | Description | Broadcast To |
|------------|-----------|-------------|--------------|
| `new_ventroom_post` | Server → Client | New post created | `ventroom_feed` group + post creator |
| `ventroom_post_updated` | Server → Client | Post was updated | `vent_post_{id}` + `ventroom_feed` groups |
| `ventroom_post_deleted` | Server → Client | Post was deleted | `vent_post_{id}` + `ventroom_feed` groups |

### Comment Events

| Event Type | Direction | Description | Broadcast To |
|------------|-----------|-------------|--------------|
| `new_ventroom_comment` | Server → Client | New comment created (general) | `vent_post_{id}` group + comment creator |
| `new_comment_on_user_post` | Server → Client | Someone commented on your post | Post owner |
| `new_reply_on_user_comment` | Server → Client | Someone replied to your comment | Parent comment owner |
| `ventroom_comment_updated` | Server → Client | Comment was updated | `vent_post_{id}` group |
| `ventroom_comment_deleted` | Server → Client | Comment was deleted | `vent_post_{id}` group |

### Reaction Events

| Event Type | Direction | Description | Broadcast To |
|------------|-----------|-------------|--------------|
| `ventroom_reaction_updated` | Server → Client | Reaction added/updated (general) | `vent_post_{id}` group + reactor |
| `new_reaction_on_user_post` | Server → Client | Someone reacted to your post | Post owner |
| `new_reaction_on_user_comment` | Server → Client | Someone reacted to your comment | Comment owner |

### WebSocket Groups

| Group Name | Purpose | Members |
|------------|---------|---------|
| `ventroom_feed` | General Ventroom updates | All users who joined Ventroom |
| `vent_post_{post_id}` | Post-specific updates | All users viewing a specific post |
| `user_{user_id}` | User-specific notifications | Individual user's connection(s) |

---

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

### WebSocket Connection

1. **Maintain Persistent Connection:** Keep WebSocket connection alive for real-time updates
2. **Implement Reconnection Logic:** Auto-reconnect on disconnect with exponential backoff
3. **Join Appropriate Groups:**
   - Join `ventroom_feed` when user opens Ventroom section
   - Join `vent_post_{id}` when user views a specific post
   - Leave groups when navigating away to conserve resources

### Real-time Event Handling

1. **Event Types:** Handle all relevant event types in your client
2. **UI Updates:** Update UI immediately upon receiving WebSocket events
3. **Optimistic Updates:** Show user actions immediately, rollback on error
4. **Deduplicate Events:** Same event may arrive via multiple groups - deduplicate by ID

### API Usage

1. **Pagination:** Always use pagination for list endpoints
2. **Caching:** Leverage 5-minute cache for feed data
3. **View Tracking:** GET requests to post detail automatically increment views
4. **Permissions:** Check user permissions before showing edit/delete options

### Media Uploads

1. **File Size:** Keep media files under 10MB for optimal performance
2. **Formats:** Use standard formats (MP4 for video, MP3 for audio, JPG/PNG for images)
3. **Compression:** Compress media before upload
4. **Preview:** Show upload progress and preview to user

### Performance Optimization

1. **Lazy Loading:** Load comments on-demand when user scrolls
2. **Image Optimization:** Use thumbnails for feed, full size for detail view
3. **Batch Updates:** Group multiple reaction updates if receiving rapidly
4. **Connection Pooling:** Reuse WebSocket connection across components

### Notifications

1. **Push Notifications:** Automatically sent for comments and reactions
2. **In-app Notifications:** Show via WebSocket for real-time updates
3. **Notification Preferences:** Respect user notification settings
4. **Rate Limiting:** Throttle notifications to avoid overwhelming users

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
