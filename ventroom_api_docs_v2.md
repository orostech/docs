# Vent Room API Documentation

## Overview
The Vent Room is a real-time community feature where users can share posts, comment, and react. It supports multimedia content (images, videos, audio) and provides real-time updates via WebSocket connections for an interactive experience. **NEW: Telegram-style view tracking** - views are automatically counted when posts appear in feeds or when users open post details.

---

## Table of Contents
1. [WebSocket Connection](#websocket-connection)
2. [Posts Management](#posts-management)
3. [Comments Management](#comments-management)
4. [Reactions Management](#reactions-management)
5. [View Tracking System](#view-tracking-system)
6. [WebSocket Events Reference](#websocket-events-reference)
7. [Error Responses](#error-responses)
8. [Best Practices](#best-practices)

---

## WebSocket Connection

### Connecting to Vent Room
To receive real-time updates, establish a WebSocket connection to the server.


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

To receive real-time updates for a specific post (comments, reactions, view counts, etc.), join the post channel.

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
Retrieve all vent room posts (paginated). **Views are automatically tracked for all posts in the response.**

**Endpoint:** `GET /api/ventroom/posts/`

**Authentication:** Required

**Query Parameters:**
- `page` (optional): Page number
- `page_size` (optional): Results per page

**Success Response (200):**
```json
{
  "count": 150,
  "next": "https://api.joinhafar.com/api/ventroom/posts/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "user": {
        "id": "uuid-string",
        "display_name": "John Doe",
        "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
      },
      "title": "my boy",
      "content": "Just needed to share this...",
      "media": "https://api.joinhafar.com/media/vent_media/2025/01/video.mp4",
      "media_type": "video",
      "created_at": "2025-01-17T10:30:00Z",
      "updated_at": "2025-01-17T10:30:00Z",
      "views": 45,
      "has_viewed": true,
      "comment_count": 12,
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
- Results are cached per user
- Posts are ordered by most recent first
- Views are tracked asynchronously in the background (no performance impact)
- Each user can only add ONE view per post (enforced by database constraint)
- View tracking uses 24-hour cache to prevent duplicate counting

---

### 2. Create Post
Create a new vent room post.

**Endpoint:** `POST /api/ventroom/posts/`

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
  "has_viewed": false,
  "comment_count": 0,
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
    "has_viewed": false,
    "comment_count": 0,
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
Retrieve details of a specific post. **View is automatically tracked.**

**Endpoint:** `GET /api/ventroom/posts/{post_id}/`

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
  "title": "my boy",
  "content": "Just needed to share this...",
  "media": "https://api.joinhafar.com/media/vent_media/2025/01/video.mp4",
  "media_type": "video",
  "created_at": "2025-01-17T10:30:00Z",
  "updated_at": "2025-01-17T10:30:00Z",
  "views": 46,
  "has_viewed": true,
  "comment_count": 12,
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
- **Auto View Tracking:** View count is automatically tracked when retrieving a post
- Each user can only add ONE view per post (duplicate views are prevented)
- View tracking happens asynchronously (no impact on response time)
- `has_viewed` field indicates if the current user has already viewed this post

---

### 3b. Get Post with Comments (Recommended for Comment Screen)
**NEW:** Get post details and paginated comments in a single request. **View is automatically tracked.**

**Endpoint:** `GET /api/ventroom/posts/{post_id}/with_comments/`

**Authentication:** Required

**Query Parameters:**
- `page` (optional): Page number for comments (default: 1)
- `page_size` (optional): Comments per page (default: 20, max: 100)

**Success Response (200):**
```json
{
  "post": {
    "id": 1,
    "user": {
      "id": "uuid-string",
      "display_name": "John Doe",
      "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
    },
    "title": "my boy",
    "content": "Just needed to share this...",
    "media": "https://api.joinhafar.com/media/vent_media/2025/01/video.mp4",
    "media_type": "video",
    "created_at": "2025-01-17T10:30:00Z",
    "updated_at": "2025-01-17T10:30:00Z",
    "views": 47,
    "has_viewed": true,
    "comment_count": 12,
    "reaction_summary": [
      {
        "reaction_type": "👍",
        "count": 23
      }
    ],
    "user_reaction": "👍"
  },
   "comments": {
        "count": 0,
        "next": null,
        "previous": null,
        "results": [
    {
      "id": 1,
      "user": {
        "id": "uuid-string",
        "display_name": "Jane Smith",
        "profile_photo": "https://api.joinhafar.com/media/photo2.jpg"
      },
      "text": "Great post!",
      "media": null,
      "media_type": null,
      "created_at": "2025-01-17T10:35:00Z",
      "updated_at": "2025-01-17T10:35:00Z",
      "replies": [
        {
          "id": 2,
          "user": {...},
          "text": "I agree!",
          "created_at": "2025-01-17T10:40:00Z",
          ...
        }
      ],
      "reaction_summary": [],
      "user_reaction": null
    }
  ]},
}
```

**Use Case:**
- Perfect for "View Comments" screen in mobile apps
- Single API call gets everything needed
- Automatically tracks view when user opens comment screen
- Returns post context along with paginated comments

---

### 3c. Batch Track Views (Manual Tracking)
**NEW:** Manually mark multiple posts as viewed. Useful for custom view tracking logic.

**Endpoint:** `POST /api/ventroom/posts/mark_as_viewed/`

**Authentication:** Required

**Request Body:**
```json
{
  "post_ids": [1, 2, 3, 4, 5]
}
```

**Success Response (200):**
```json
{
  "status": "success",
  "tracked_posts": 5,
  "message": "Views tracked successfully"
}
```

**Notes:**
- Useful for tracking views when implementing custom scrolling logic
- Views are tracked asynchronously (batch processing)
- Duplicate views are automatically prevented
- Each user can only view each post once

**Error Response (400):**
```json
{
  "error": "post_ids array is required"
}
```

---

### 4. Update Post
Update an existing post (owner or admin only).

**Endpoint:** `PUT /api/ventroom/posts/{post_id}/` or `PATCH /api/ventroom/posts/{post_id}/`

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
  "has_viewed": true,
  "comment_count": 12,
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

**Endpoint:** `DELETE /api/ventroom/posts/{post_id}/`

**Authentication:** Required (Owner or Admin)

**Success Response (204):** No content

**Real-time Broadcast:**

Two events are sent when a post is deleted:

1. **To vent_post_{post_id} group:**
```json
{
  "type": "ventroom_post_deleted",
  "data": {
    "post_id": 1
  },
  "timestamp": "2025-01-17T12:00:00Z"
}
```

2. **To ventroom_feed group:**
```json
{
  "type": "ventroom_post_deleted",
  "data": {
    "post_id": 1
  },
  "timestamp": "2025-01-17T12:00:00Z"
}
```

---

### 6. Get Trending Posts
**NEW:** Get posts with the most views in the last 24 hours.

**Endpoint:** `GET /api/ventroom/posts/trending/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "count": 20,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 15,
      "user": {...},
      "title": "Hot Topic",
      "content": "Everyone is talking about this...",
      "views": 1250,
      "has_viewed": false,
      "comment_count": 89,
      "created_at": "2025-01-17T08:00:00Z",
      ...
    }
  ]
}
```

**Notes:**
- Returns top 20 trending posts
- Sorted by recent views (last 24 hours), then total views
- Views are automatically tracked for posts in response

---

### 7. Get View Statistics
**NEW:** Get detailed view analytics for a specific post.

**Endpoint:** `GET /api/ventroom/posts/{post_id}/view_stats/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "post_id": 1,
  "total_views": 156,
  "unique_viewers": 142,
  "recent_views_24h": 23,
  "views_last_7_days": [
    {
      "date": "2025-01-16",
      "views": 18
    },
    {
      "date": "2025-01-15",
      "views": 12
    },
    {
      "date": "2025-01-14",
      "views": 25
    },
    {
      "date": "2025-01-13",
      "views": 31
    },
    {
      "date": "2025-01-12",
      "views": 19
    },
    {
      "date": "2025-01-11",
      "views": 14
    },
    {
      "date": "2025-01-10",
      "views": 8
    }
  ],
  "has_user_viewed": true
}
```

**Use Case:**
- Post analytics for content creators
- View trends over time
- Engagement metrics

---

## Comments Management

### 8. List Post Comments
Get all comments for a specific post, including replies. **Post view is automatically tracked.**

**Endpoint:** `GET /api/ventroom/posts/{post_id}/comments/`

**Authentication:** Required

**Query Parameters:**
- `page` (optional): Page number (default: 1)
- `page_size` (optional): Results per page (default: 20, max: 100)

**Success Response (200):**
```json
{
  "count": 12,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
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
      "replies": [
        {
          "id": 2,
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
}
```

**Notes:**
- **Auto View Tracking:** Post view is tracked when accessing comments
- Returns paginated top-level comments only
- Each comment includes nested `replies` array (limited to 3 in list view)
- Comments ordered by most recent first

---

### 9. Create Comment
Add a comment to a post or reply to another comment.

**Endpoint:** `POST /api/ventroom/posts/{post_id}/comments/`

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

### 10. Update Comment
Update a comment (owner or admin only).

**Endpoint:** `PUT /api/ventroom/posts/{post_id}/comments/{comment_id}/` or `PATCH /api/ventroom/posts/{post_id}/comments/{comment_id}/`

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
  "user": {...},
  "text": "Updated comment text",
  "media": null,
  "media_type": null,
  "created_at": "2025-01-17T11:00:00Z",
  "updated_at": "2025-01-17T11:30:00Z",
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
    "user": {...},
    "text": "Updated comment text",
    ...
  },
  "timestamp": "2025-01-17T11:30:00Z"
}
```

---

### 11. Delete Comment
Delete a comment (owner or admin only).

**Endpoint:** `DELETE /api/ventroom/posts/{post_id}/comments/{comment_id}/`

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

### 12. Add/Update Reaction
React to a post or comment. If the user already has a reaction, it will be updated.

**Endpoint:** `POST /api/ventroom/reactions/`

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

## View Tracking System

### Overview
The Vent Room implements Telegram-style view tracking where views are counted automatically when:
1. Posts appear in the feed list
2. Users open a specific post
3. Users access the comment screen

### How It Works

**Automatic Tracking:**
- ✅ Views tracked when fetching post list
- ✅ Views tracked when opening single post
- ✅ Views tracked when accessing comments
- ✅ One view per user per post (enforced by database)
- ✅ 24-hour cache prevents duplicate counting
- ✅ Asynchronous processing (no performance impact)

**View Counting Rules:**
1. Each user can only view a post ONCE (database constraint)
2. Views are cached for 24 hours to prevent duplicates
3. View tracking happens in background (Celery tasks)
4. Real-time updates broadcast to connected clients
5. Anonymous views are NOT counted (authentication required)

### Real-time View Updates

When a view is counted, a WebSocket event is broadcast:

**To vent_post_{post_id} group:**
```json
{
  "type": "ventroom_post_view_updated",
  "data": {
    "post_id": 1,
    "views": 157
  },
  "timestamp": "2025-01-17T14:30:00Z"
}
```

**To ventroom_feed group:**
```json
{
  "type": "ventroom_post_view_updated",
  "data": {
    "post_id": 1,
    "views": 157
  },
  "timestamp": "2025-01-17T14:30:00Z"
}
```

### View Statistics API

**Detailed Analytics:** `GET /api/ventroom/posts/{post_id}/view_stats/`

Returns comprehensive view data including:
- Total views
- Unique viewers
- Recent views (24h)
- Daily breakdown (7 days)
- Whether current user has viewed

**Trending Posts:** `GET /api/ventroom/posts/trending/`

Returns posts ranked by recent views (last 24 hours)

### Manual View Tracking

**Batch Tracking:** `POST /api/ventroom/posts/mark_as_viewed/`

Useful for custom implementations where you want manual control over view tracking.

```json
{
  "post_ids": [1, 2, 3, 4, 5]
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
| `ventroom_post_view_updated` | Server → Client | **NEW:** Post view count updated | `vent_post_{id}` + `ventroom_feed` groups |

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

## Data Models

### Post Fields
- `id` (integer): Unique post identifier
- `user` (object): Post author information
  - `id` (uuid): User ID
  - `display_name` (string): User's display name
  - `profile_photo` (url): User's profile photo URL
- `title` (string): Post title (optional, max 150 chars)
- `content` (string): Post content (required)
- `media` (string/url): Media file URL (optional)
- `media_type` (string): Type of media (`image`, `video`, `audio`)
- `created_at` (datetime): Creation timestamp
- `updated_at` (datetime): Last update timestamp
- `views` (integer): **Total view count**
- `has_viewed` (boolean): **NEW: Whether current user has viewed this post**
- `comment_count` (integer): **NEW: Total number of comments**
- `reaction_summary` (array): Aggregated reaction counts
- `user_reaction` (string): Current user's reaction (if any)

### Comment Fields
- `id` (integer): Unique comment identifier
- `user` (object): Comment author information
- `parent` (integer/null): ID of parent comment (for replies)
- `text` (string): Comment content (required)
- `media` (string/url): Media file URL (optional)
- `media_type` (string): Type of media
- `created_at` (datetime): Creation timestamp
- `updated_at` (datetime): Last update timestamp
- `replies` (array): Nested replies to this comment (limited to 3 in list view)
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

## Error Responses

### Common Error Codes

**400 Bad Request**
```json
{
  "field_name": [
    "Error message describing what went wrong"
  ]
}
```

**401 Unauthorized**
```json
{
  "detail": "Authentication credentials were not provided."
}
```

**403 Forbidden**
```json
{
  "detail": "You do not have permission to perform this action."
}
```

**404 Not Found**
```json
{
  "detail": "Not found."
}
```

**429 Too Many Requests**
```json
{
  "detail": "Request was throttled. Expected available in 45 seconds."
}
```

**500 Internal Server Error**
```json
{
  "detail": "Internal server error. Please try again later."
}
```

---

## Best Practices

### WebSocket Connection

1. **Maintain Persistent Connection:** Keep WebSocket connection alive for real-time updates
2. **Implement Reconnection Logic:** Auto-reconnect on disconnect with exponential backoff
3. **Join Appropriate Groups:**
   - Join `ventroom_feed` when user opens Ventroom section
   - Join `vent_post_{id}` when user views a specific post
   - Leave groups when navigating away to conserve resources
4. **Handle View Updates:** Listen for `ventroom_post_view_updated` events to update view counts in real-time

### Real-time Event Handling

1. **Event Types:** Handle all relevant event types in your client
2. **UI Updates:** Update UI immediately upon receiving WebSocket events
3. **Optimistic Updates:** Show user actions immediately, rollback on error
4. **Deduplicate Events:** Same event may arrive via multiple groups - deduplicate by ID
5. **View Count Updates:** Update post view counts when receiving `ventroom_post_view_updated` events

### API Usage

1. **Pagination:** Always use pagination for list endpoints
2. **Caching:** Leverage 5-minute cache for feed data
3. **View Tracking:** 
   - Views are automatically tracked on all GET requests
   - No need to manually track views in most cases
   - Use `mark_as_viewed` endpoint only for custom implementations
4. **Permissions:** Check user permissions before showing edit/delete options
5. **Use `with_comments` Endpoint:** 
   - Prefer `/posts/{id}/with_comments/` for comment screens
   - Single API call reduces latency and server load

### View Tracking Best Practices

1. **Automatic Tracking:** 
   - Let the API handle view tracking automatically
   - Views counted when posts appear in feed
   - Views counted when opening post details
   - Views counted when accessing comments
2. **No Duplicate Tracking:**
   - Each user can only view a post once
   - 24-hour cache prevents duplicate requests
   - Database constraint ensures data integrity
3. **Real-time Updates:**
   - Subscribe to `vent_post_{id}` group to receive view updates
   - Update UI when `ventroom_post_view_updated` event received
4. **Performance:**
   - View tracking is asynchronous (no API slowdown)
   - Batch processing for feed scrolling
   - Cache layer prevents unnecessary DB queries

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
5. **View Tracking:**
   - Use automatic tracking (built-in to API)
   - Avoid manual `mark_as_viewed` calls unless necessary
   - Trust the 24-hour cache to prevent duplicates

### Notifications

1. **Push Notifications:** Automatically sent for comments and reactions
2. **In-app Notifications:** Show via WebSocket for real-time updates
3. **Notification Preferences:** Respect user notification settings
4. **Rate Limiting:** Throttle notifications to avoid overwhelming users

### Mobile App Recommendations

1. **Feed Screen:**
   - Display posts from `GET /api/ventroom/posts/`
   - Views automatically tracked for visible posts
   - Show `views` count on each post card
   - Use `has_viewed` to show visual indicator (e.g., eye icon)

2. **Comment Screen:**
   - Use `GET /api/ventroom/posts/{id}/with_comments/` endpoint
   - Single API call gets post + paginated comments
   - Join `vent_post_{id}` WebSocket group
   - Listen for new comments and view updates

3. **Real-time Updates:**
   - Update view counts when receiving WebSocket events
   - Show visual feedback when view count increases
   - Animate view counter for better UX

4. **Offline Handling:**
   - Cache posts locally for offline viewing
   - Queue view tracking for when connection returns
   - Show cached view counts with visual indicator

---

## Rate Limiting

- **Standard endpoints:** 100 requests per minute
- **Create endpoints:** 20 requests per minute
- **Reaction endpoint:** 60 requests per minute
- **View tracking:** No rate limit (handled by cache layer)

Exceeded rate limits will return:
```json
{
  "detail": "Request was throttled. Expected available in 45 seconds."
}
```

---

## API Endpoints Summary

### Posts
- `GET /api/ventroom/posts/` - List all posts (auto-tracks views)
- `POST /api/ventroom/posts/` - Create new post
- `GET /api/ventroom/posts/{id}/` - Get single post (auto-tracks view)
- `GET /api/ventroom/posts/{id}/with_comments/` - **NEW:** Get post with comments (auto-tracks view)
- `PUT/PATCH /api/ventroom/posts/{id}/` - Update post
- `DELETE /api/ventroom/posts/{id}/` - Delete post
- `GET /api/ventroom/posts/trending/` - **NEW:** Get trending posts
- `GET /api/ventroom/posts/{id}/view_stats/` - **NEW:** Get view statistics
- `POST /api/ventroom/posts/mark_as_viewed/` - **NEW:** Manually mark posts as viewed

### Comments
- `GET /api/ventroom/posts/{post_id}/comments/` - List comments (auto-tracks post view)
- `POST /api/ventroom/posts/{post_id}/comments/` - Create comment
- `PUT/PATCH /api/ventroom/posts/{post_id}/comments/{id}/` - Update comment
- `DELETE /api/ventroom/posts/{post_id}/comments/{id}/` - Delete comment

### Reactions
- `POST /api/ventroom/reactions/` - Add/update reaction

---

## Migration Guide

If you're updating from a previous version without view tracking:

### Backend Changes Required

1. **Run Migrations:**
```bash
python manage.py makemigrations
python manage.py migrate
```

2. **Update Celery:**
   - Ensure Celery is running with the new tasks
   - Add `cleanup_old_views` to Celery Beat schedule

3. **Optional - Sync Existing Data:**
```python
python manage.py shell

from community.models import VentPost, VentPostView
for post in VentPost.objects.all():
    actual_count = VentPostView.objects.filter(post=post).count()
    if post.views != actual_count:
        post.views = actual_count
        post.save(update_fields=['views'])
```

### Frontend Changes Required

1. **Update Post Model:**
   - Add `has_viewed` field (boolean)
   - Add `comment_count` field (integer)

2. **Handle New WebSocket Event:**
   - Listen for `ventroom_post_view_updated` event
   - Update view count in UI when event received

3. **UI Updates:**
   - Display view count on post cards
   - Show visual indicator if user has viewed post
   - Animate view counter when it updates

4. **Use New Endpoint:**
   - Switch to `/with_comments/` for comment screens
   - Single API call improves performance

---

