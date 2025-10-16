# Chat System API Documentation

## Overview
The Chat System enables real-time messaging between users with support for message requests, direct chats, group chats, and multimedia content. It includes WebSocket support for real-time updates and follows an Instagram-like message request flow for non-matched users.

---

## Table of Contents
1. [Chat Management](#chat-management)
2. [Message Management](#message-management)
3. [Message Requests](#message-requests)
4. [Group Chat Management](#group-chat-management)
5. [Reactions Management](#reactions-management)
6. [WebSocket Events](#websocket-events)
7. [Error Responses](#error-responses)

---

## Chat Management

### 1. List All Chats (Unified View)
Retrieve all chats (private, groups, and community rooms) in a unified list.

**Endpoint:** `GET /v2/unified_chat/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "count": 0,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "type": "private",
      "other_user": {
        "id": "uuid-string",
        "display_name": "Jane Doe",
        "profile_photo": "https://api.joinhafar.com/media/photo.jpg",
        "bio": "Hello there!",
        "is_online": true
      },
      "last_message": {
        "id": 45,
        "sender": {
          "id": "uuid-string",
          "display_name": "Jane Doe"
        },
        "content": "Hey! How are you?",
        "content_type": "TEXT",
        "created_at": "2025-01-17T10:30:00Z",
        "read_at": null,
        "reactions": {
          "user-id-1": "❤️"
        },
        "reply_to": null
      },
      "unread_count": 3,
      "last_activity": "2025-01-17T10:30:00Z",
      "is_active": true,
      "is_accepted": true,
      "rate_user": false,
      "participants_count": 2
    },
    {
      "id": 2,
      "type": "group",
      "name": "Weekend Hangout",
      "description": "Planning our weekend activities",
      "avatar": "https://api.joinhafar.com/media/group_avatars/photo.jpg",
      "admin": {
        "id": "uuid-string",
        "display_name": "John Smith"
      },
      "last_message": {
        "id": 78,
        "sender": {
          "id": "uuid-string",
          "display_name": "Mike Johnson"
        },
        "content": "Let's meet at 3pm",
        "content_type": "TEXT",
        "created_at": "2025-01-17T09:15:00Z"
      },
      "unread_count": 0,
      "last_activity": "2025-01-17T09:15:00Z",
      "participants_count": 5,
      "is_active": true
    }
  ]
}
```

**Notes:**
- Combines private chats, group chats, and community rooms
- Sorted by most recent activity
- Cache is not applied for real-time accuracy

---

### 2. List Private Chats Only
Get only private (one-on-one) chats.

**Endpoint:** `GET /chatsv2/`

**Authentication:** Required

**Query Parameters:**
- `page` (optional): Page number
- `page_size` (optional): Results per page

**Success Response (200):**
```json
{
  "count": 15,
  "next": "https://api.com/chat/chatsv2/?page=2",
  "previous": null,
  "results": [
    {
      "id": 1,
      "other_user": {
        "id": "uuid-string",
        "display_name": "Jane Doe",
        "profile_photo": "https://api.joinhafar.com/media/photo.jpg",
        "bio": "Hello there!",
        "is_online": true
      },
      "last_message": {
        "id": 45,
        "sender": {
          "id": "uuid-string",
          "display_name": "Jane Doe"
        },
        "content": "Hey! How are you?",
        "content_type": "TEXT",
        "created_at": "2025-01-17T10:30:00Z",
        "read_at": null,
        "reactions": {},
        "reply_to": null
      },
      "unread_count": 3,
      "last_activity": "2025-01-17T10:30:00Z",
      "is_active": true,
      "is_accepted": true,
      "rate_user": false
    }
  ]
}
```

---

### 3. Get Single Chat
Retrieve details of a specific chat.

**Endpoint:** `GET /chatsv2/{chat_id}/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "id": 1,
  "other_user": {
    "id": "uuid-string",
    "display_name": "Jane Doe",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg",
    "bio": "Hello there!",
    "is_online": true
  },
  "last_message": {
    "id": 45,
    "sender": {
      "id": "uuid-string",
      "display_name": "Jane Doe"
    },
    "content": "Hey! How are you?",
    "content_type": "TEXT",
    "created_at": "2025-01-17T10:30:00Z",
    "read_at": null,
    "reactions": {},
    "reply_to": null
  },
  "unread_count": 3,
  "last_activity": "2025-01-17T10:30:00Z",
  "is_active": true,
  "is_accepted": true,
  "rate_user": false
}
```

---

### 4. Accept Chat Request
Accept a pending chat (when requires_acceptance is true).

**Endpoint:** `POST /chatsv2/{chat_id}/accept/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "status": "chat accepted"
}
```

**Error Response (403):**
```json
{
  "error": "Not authorized"
}
```

---

### 5. Block User
Block a user and deactivate the chat.

**Endpoint:** `POST /chatsv2/{chat_id}/block/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "status": "user blocked"
}
```

**Notes:**
- Creates a UserBlock record
- Deactivates the chat (is_active = false)
- Blocked user cannot send messages

---

## Message Management

### 6. List Messages
Retrieve messages from a specific chat.

**Endpoint:** `GET /messages/?chatId={chat_id}`

**Authentication:** Required

**Query Parameters:**
- `chatId` (required): The chat ID
- `page` (optional): Page number
- `page_size` (optional): Results per page

**Success Response (200):**
```json
{
  "count": 150,
  "next": "https://api.com/chat/messages/?chatId=1&page=2",
  "previous": null,
  "results": [
    {
      "id": 45,
      "sender": {
        "id": "uuid-string",
        "display_name": "Jane Doe",
        "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
      },
      "chat": 1,
      "content": "Hey! How are you?",
      "content_type": "TEXT",
      "created_at": "2025-01-17T10:30:00Z",
      "read_at": null,
      "reactions": {
        "user-id-1": "❤️",
        "user-id-2": "👍"
      },
      "reply_to": null,
      "is_mine": false
    },
    {
      "id": 46,
      "sender": {
        "id": "uuid-string-2",
        "display_name": "John Smith",
        "profile_photo": "https://api.joinhafar.com/media/photo2.jpg"
      },
      "chat": 1,
      "content": "https://api.joinhafar.com/media/chats/1/image/uuid.jpg |Check this out!",
      "content_type": "IMAGE",
      "created_at": "2025-01-17T10:32:00Z",
      "read_at": "2025-01-17T10:33:00Z",
      "reactions": {},
      "reply_to": {
        "id": 45,
        "sender": {
          "id": "uuid-string",
          "display_name": "Jane Doe"
        },
        "content": "Hey! How are you?",
        "content_type": "TEXT"
      },
      "is_mine": true
    }
  ]
}
```

**Notes:**
- Messages are ordered by most recent first (descending)
- Automatically marks unread messages as read when retrieved
- Supports pagination for performance

---

### 7. Send Text Message
Send a text message in a chat.

**Endpoint:** `POST /messages/`

**Authentication:** Required

**Content-Type:** `application/json`

**Request Body:**
```json
{
  "chat_id": 1,
  "content": "Hello! How are you doing?",
  "content_type": "TEXT",
  "reply_to": null
}
```

**Success Response (201):**
```json
{
  "id": 47,
  "sender": {
    "id": "uuid-string",
    "display_name": "John Smith",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "chat": 1,
  "content": "Hello! How are you doing?",
  "content_type": "TEXT",
  "created_at": "2025-01-17T10:35:00Z",
  "read_at": null,
  "reactions": {},
  "reply_to": null,
  "is_mine": true
}
```

**WebSocket Broadcast:**
Message is broadcast to both chat participants:
```json
{
  "type": "chat_message_sent",
  "data": {
    "id": 47,
    "sender": {...},
    "chat": 1,
    "content": "Hello! How are you doing?",
    "content_type": "TEXT",
    "created_at": "2025-01-17T10:35:00Z",
    "local_id": "temp-uuid-123"
  },
  "timestamp": "2025-01-17T10:35:00Z"
}
```

---

### 8. Send Media Message
Send a message with media (image, video, audio, GIF).

**Endpoint:** `POST /messages/`

**Authentication:** Required

**Content-Type:** `multipart/form-data`

**Request Body:**
```
chat_id: 1
content_type: IMAGE
file: <image_file>
caption: Check this out!
reply_to: null
```

**Media Constraints:**
- **IMAGE**: Max 10MB, formats: .jpg, .jpeg, .png, .gif, .webp
- **VIDEO**: Max 100MB, formats: .mp4, .mov, .avi, .webm, .mkv
- **AUDIO**: Max 10MB, formats: .mp3, .wav, .ogg, .m4a, .aac, .opus, .wma, .webm
- **GIF**: Max 5MB, format: .gif

**Success Response (201):**
```json
{
  "id": 48,
  "sender": {
    "id": "uuid-string",
    "display_name": "John Smith",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "chat": 1,
  "content": "https://api.joinhafar.com/media/chats/1/image/uuid.jpg |Check this out!",
  "content_type": "IMAGE",
  "created_at": "2025-01-17T10:40:00Z",
  "read_at": null,
  "reactions": {},
  "reply_to": null,
  "is_mine": true
}
```

**Error Response (400):**
```json
{
  "error": "File too large. Max size for IMAGE: 10.0MB"
}
```

```json
{
  "error": "Invalid file type. Allowed extensions: .jpg, .jpeg, .png, .gif, .webp"
}
```

---

### 9. Send Initial Message (First Contact)
Send the first message to a user (creates chat or message request).

**Endpoint:** `POST /send-initial-message/`

**Authentication:** Required

**Request Body:**
```json
{
  "receiver_id": "uuid-string",
  "content": "Hi! I'd like to connect with you.",
  "content_type": "TEXT"
}
```

**Success Response (200) - Direct Chat Created:**
```json
{
  "type": "CHAT",
  "chat": {
    "id": 5,
    "other_user": {
      "id": "uuid-string",
      "display_name": "Jane Doe",
      "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
    },
    "last_message": {
      "id": 50,
      "content": "Hi! I'd like to connect with you.",
      "content_type": "TEXT",
      "created_at": "2025-01-17T11:00:00Z"
    },
    "is_accepted": true,
    "last_activity": "2025-01-17T11:00:00Z"
  },
  "message": {
    "id": 50,
    "content": "Hi! I'd like to connect with you.",
    "content_type": "TEXT",
    "created_at": "2025-01-17T11:00:00Z"
  }
}
```

**Success Response (201) - Message Request Created:**
```json
{
  "type": "NEW_REQUEST",
  "message_request": {
    "id": 10,
    "sender": {
      "id": "uuid-string",
      "display_name": "John Smith",
      "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
    },
    "receiver": {
      "id": "uuid-string-2",
      "display_name": "Jane Doe",
      "profile_photo": "https://api.joinhafar.com/media/photo2.jpg"
    },
    "message": "Hi! I'd like to connect with you.",
    "status": "PENDING",
    "created_at": "2025-01-17T11:00:00Z"
  }
}
```

**Notes:**
- Direct chat is created if users are matched or sender is premium
- Otherwise, creates a message request
- If chat already exists and is pending, returns 403 error

---

### 10. Check Chat/Request Status
Check the status of a conversation with a specific user.

**Endpoint:** `GET /message-requestsv2/status/?receiver_id={user_id}`

**Authentication:** Required

**Query Parameters:**
- `receiver_id` (required): ID of the other user

**Success Response (200) - Active Chat:**
```json
{
  "type": "chat",
  "data": {
    "id": 1,
    "other_user": {...},
    "last_message": {...},
    "is_accepted": true,
    "last_activity": "2025-01-17T10:30:00Z"
  }
}
```

**Success Response (200) - Pending Request:**
```json
{
  "type": "request",
  "data": {
    "id": 10,
    "sender": {...},
    "receiver": {...},
    "message": "Hi! I'd like to connect with you.",
    "status": "PENDING",
    "created_at": "2025-01-17T11:00:00Z"
  }
}
```

**Success Response (200) - No Connection:**
```json
{
  "type": "NONE"
}
```

---

## Message Requests

### 11. List Pending Requests (Received)
Get all pending message requests sent to the current user.

**Endpoint:** `GET /message-requestsv2/pending/`

**Authentication:** Required

**Query Parameters:**
- `sort` (optional): `recent` (default) or `az` (alphabetical)
- `filter` (optional): `ratedAccount` or `premiumAccount`

**Success Response (200):**
```json
{
  "count": 8,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 10,
      "sender": {
        "id": "uuid-string",
        "display_name": "John Smith",
        "profile_photo": "https://api.joinhafar.com/media/photo.jpg",
        "bio": "Software developer",
        "is_premium": false
      },
      "receiver": {
        "id": "uuid-string-2",
        "display_name": "Jane Doe"
      },
      "message": "Hi! I'd like to connect with you.",
      "status": "PENDING",
      "created_at": "2025-01-17T11:00:00Z",
      "updated_at": "2025-01-17T11:00:00Z"
    }
  ]
}
```

**Filter Options:**
- `ratedAccount`: Shows only requests from users with average rating ≥ 4.0
- `premiumAccount`: Shows only requests from premium users

---

### 12. List Sent Requests
Get all message requests sent by the current user.

**Endpoint:** `GET /message-requestsv2/sent/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "count": 3,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 11,
      "sender": {
        "id": "uuid-string",
        "display_name": "John Smith"
      },
      "receiver": {
        "id": "uuid-string-2",
        "display_name": "Sarah Johnson",
        "profile_photo": "https://api.joinhafar.com/media/photo3.jpg"
      },
      "message": "Hello! Would love to connect.",
      "status": "PENDING",
      "created_at": "2025-01-17T09:00:00Z",
      "updated_at": "2025-01-17T09:00:00Z"
    }
  ]
}
```

---

### 13. Respond to Message Request
Accept or reject a message request.

**Endpoint:** `POST /message-requestsv2/{request_id}/respond/`

**Authentication:** Required

**Request Body (Accept):**
```json
{
  "action": "accepted"
}
```

**Request Body (Reject):**
```json
{
  "action": "rejected"
}
```

**Success Response (200) - Accepted:**
```json
{
  "type": "CHAT",
  "data": {
    "id": 6,
    "other_user": {
      "id": "uuid-string",
      "display_name": "John Smith",
      "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
    },
    "last_message": {
      "id": 52,
      "content": "Hi! I'd like to connect with you.",
      "content_type": "TEXT",
      "created_at": "2025-01-17T11:00:00Z"
    },
    "is_accepted": true,
    "last_activity": "2025-01-17T11:30:00Z"
  }
}
```

**Success Response (200) - Rejected:**
```json
{
  "status": "request rejected"
}
```

**Error Response (403):**
```json
{
  "error": "Not authorized"
}
```

---

### 14. Batch Actions on Requests
Perform bulk actions on multiple message requests.

**Endpoint:** `POST /message-requestsv2/batch_action/`

**Authentication:** Required

**Request Body:**
```json
{
  "ids": [10, 11, 12],
  "action": "ACCEPT"
}
```

**Actions:**
- `ACCEPT`: Accept all requests and create chats
- `REJECT`: Reject all requests
- `DELETE`: Delete all requests

**Success Response (200) - Batch Accept:**
```json
{
  "type": "BATCH_CHAT",
  "data": [
    {
      "id": 6,
      "other_user": {...},
      "last_message": {...},
      "is_accepted": true
    },
    {
      "id": 7,
      "other_user": {...},
      "last_message": {...},
      "is_accepted": true
    }
  ]
}
```

**Success Response (200) - Batch Reject:**
```json
{
  "status": "Requests rejected",
  "type": "BATCH_REJECT"
}
```

---

## Group Chat Management

### 15. List Group Chats
Get all group chats the user is part of.

**Endpoint:** `GET /groups/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "count": 5,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 2,
      "name": "Weekend Hangout",
      "description": "Planning our weekend activities",
      "admin": {
        "id": "uuid-string",
        "display_name": "John Smith",
        "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
      },
      "avatar": "https://api.joinhafar.com/media/group_avatars/photo.jpg",
      "participants": [
        {
          "id": "uuid-string-1",
          "display_name": "John Smith"
        },
        {
          "id": "uuid-string-2",
          "display_name": "Jane Doe"
        }
      ],
      "participants_count": 5,
      "created_at": "2025-01-15T10:00:00Z",
      "last_activity": "2025-01-17T09:15:00Z",
      "is_public": false,
      "invite_code": null
    }
  ]
}
```

---

### 16. Create Group Chat
Create a new group chat.

**Endpoint:** `POST /groups/`

**Authentication:** Required

**Content-Type:** `multipart/form-data`

**Request Body:**
```
name: Weekend Hangout
description: Planning our weekend activities
avatar: <image_file>
is_public: false
```

**Success Response (201):**
```json
{
  "id": 3,
  "name": "Weekend Hangout",
  "description": "Planning our weekend activities",
  "admin": {
    "id": "uuid-string",
    "display_name": "John Smith"
  },
  "avatar": "https://api.joinhafar.com/media/group_avatars/uuid.jpg",
  "participants": [
    {
      "id": "uuid-string",
      "display_name": "John Smith"
    }
  ],
  "participants_count": 1,
  "created_at": "2025-01-17T12:00:00Z",
  "last_activity": "2025-01-17T12:00:00Z",
  "is_public": false,
  "invite_code": null
}
```

**Notes:**
- Creator automatically becomes admin and participant
- Group avatar is optional

---

### 17. List Group Messages
Get messages from a specific group chat.

**Endpoint:** `GET /group_messages/?groupId={group_id}`

**Authentication:** Required

**Query Parameters:**
- `groupId` (required): The group ID

**Success Response (200):**
```json
{
  "count": 50,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 78,
      "group": 2,
      "sender": {
        "id": "uuid-string",
        "display_name": "Mike Johnson",
        "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
      },
      "content": "Let's meet at 3pm",
      "content_type": "TEXT",
      "created_at": "2025-01-17T09:15:00Z",
      "read_by": [
        {
          "id": "uuid-string-1",
          "display_name": "John Smith"
        }
      ],
      "reply_to": null,
      "is_mine": false
    }
  ]
}
```

**Notes:**
- Messages automatically marked as read when retrieved
- Returns messages in descending order (newest first)

---

### 18. Send Group Message
Send a message to a group chat.

**Endpoint:** `POST /group_messages/`

**Authentication:** Required

**Request Body:**
```json
{
  "group_id": 2,
  "content": "Hey everyone! What's the plan?",
  "content_type": "TEXT"
}
```

**Success Response (201):**
```json
{
  "id": 79,
  "group": 2,
  "sender": {
    "id": "uuid-string",
    "display_name": "John Smith",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "content": "Hey everyone! What's the plan?",
  "content_type": "TEXT",
  "created_at": "2025-01-17T12:30:00Z",
  "read_by": [],
  "reply_to": null,
  "is_mine": true
}
```

**Error Response (403):**
```json
{
  "error": "You are not a participant of this group"
}
```

---

## Reactions Management

### 19. Add Reaction to Message
Add or update a reaction on a message (WhatsApp-style: one reaction per user).

**WebSocket Only** - Use WebSocket event `reaction_add`

**WebSocket Request:**
```json
{
  "type": "reaction_add",
  "data": {
    "message_id": 45,
    "emoji": "❤️"
  }
}
```

**WebSocket Response:**
Broadcast to chat participants:
```json
{
  "type": "reaction_updated",
  "data": {
    "id": 45,
    "sender": {...},
    "chat": 1,
    "content": "Hey! How are you?",
    "reactions": {
      "user-id-1": "❤️",
      "user-id-2": "👍"
    },
    "created_at": "2025-01-17T10:30:00Z"
  },
  "timestamp": "2025-01-17T12:35:00Z"
}
```

**Notes:**
- One reaction per user per message
- If user reacts with different emoji, previous reaction is removed
- Supports multiple users reacting with different emojis

---

### 20. Remove Reaction from Message
Remove your reaction from a message.

**WebSocket Only** - Use WebSocket event `reaction_remove`

**WebSocket Request:**
```json
{
  "type": "reaction_remove",
  "data": {
    "message_id": 45,
    "emoji": "❤️"
  }
}
```

**WebSocket Response:**
```json
{
  "type": "reaction_updated",
  "data": {
    "id": 45,
    "sender": {...},
    "chat": 1,
    "content": "Hey! How are you?",
    "reactions": {
      "user-id-2": "👍"
    },
    "created_at": "2025-01-17T10:30:00Z"
  },
  "timestamp": "2025-01-17T12:36:00Z"
}
```

---

## WebSocket Events

### Connection
Connect to the WebSocket server to receive real-time updates:

**WebSocket URL:** `wss://api.joinhafar.com/ws/`

**Authentication:** Include auth token in connection headers or query params

---

### Event Types

#### 1. Join Chat Room
Join a chat to receive real-time messages.

**Client Sends:**
```json
{
  "type": "join_chat",
  "data": {
    "chat_id": 1
  }
}
```

**Server Response:**
```json
{
  "type": "chat_joined",
  "data": {
    "chat_id": 1,
    "rate_user": false
  },
  "timestamp": "2025-01-17T12:00:00Z"
}
```

**Notes:**
- Required before sending messages via WebSocket
- `rate_user` indicates if the user can rate the other participant (requires 10+ messages from them)

---

#### 2. Send Chat Message (via WebSocket)
Send a message through WebSocket connection.

**Client Sends:**
```json
{
  "type": "chat_message",
  "data": {
    "chat_id": 1,
    "content": "Hello there!",
    "content_type": "TEXT",
    "reply_to": null,
    "local_id": "temp-uuid-123"
  }
}
```

**Server Broadcasts:**
```json
{
  "type": "chat_message_sent",
  "data": {
    "id": 47,
    "sender": {
      "id": "uuid-string",
      "display_name": "John Smith",
      "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
    },
    "chat": 1,
    "content": "Hello there!",
    "content_type": "TEXT",
    "created_at": "2025-01-17T12:05:00Z",
    "read_at": null,
    "reactions": {},
    "reply_to": null,
    "local_id": "temp-uuid-123"
  },
  "timestamp": "2025-01-17T12:05:00Z"
}
```

**Notes:**
- `local_id` is optional but useful for optimistic UI updates
- Message is broadcast to both chat participants
- Also sent to each user's personal channel (`user_{user_id}`)

---

#### 3. Typing Indicators
Notify other user when typing starts/stops.

**Typing Start:**
```json
{
  "type": "typing_start",
  "data": {
    "chat_id": 1
  }
}
```

**Server Broadcasts:**
```json
{
  "type": "user_typing_start",
  "data": {
    "user_id": "uuid-string",
    "chat_id": 1
  },
  "timestamp": "2025-01-17T12:06:00Z"
}
```

**Typing Stop:**
```json
{
  "type": "typing_stop",
  "data": {
    "chat_id": 1
  }
}
```

**Server Broadcasts:**
```json
{
  "type": "user_typing_stop",
  "data": {
    "user_id": "uuid-string",
    "chat_id": 1
  },
  "timestamp": "2025-01-17T12:06:30Z"
}
```

**Notes:**
- Only sent to the other participant (not to sender)
- Useful for showing "User is typing..." indicators

---

#### 4. Send First Message (via WebSocket)
Send the first message to a user (creates chat or message request).

**Client Sends:**
```json
{
  "type": "send_first_message",
  "data": {
    "recipient_id": "uuid-string",
    "content": "Hi! I'd like to connect with you.",
    "content_type": "TEXT",
    "local_id": "temp-uuid-456"
  }
}
```

**Server Response (Direct Chat):**
```json
{
  "type": "chat_created",
  "data": {
    "chat": {
      "id": 5,
      "other_user": {...},
      "last_message": {
        "id": 50,
        "content": "Hi! I'd like to connect with you.",
        "content_type": "TEXT",
        "created_at": "2025-01-17T11:00:00Z"
      },
      "is_accepted": true,
      "last_activity": "2025-01-17T11:00:00Z"
    },
    "message": {...}
  },
  "timestamp": "2025-01-17T11:00:00Z"
}
```

**Server Response (Message Request):**
```json
{
  "type": "message_request_sent",
  "data": {
    "request": {
      "id": 10,
      "sender": {...},
      "receiver": {...},
      "message": "Hi! I'd like to connect with you.",
      "status": "PENDING",
      "created_at": "2025-01-17T11:00:00Z"
    }
  },
  "timestamp": "2025-01-17T11:00:00Z"
}
```

**Recipient Receives (New Chat):**
```json
{
  "type": "new_chat_received",
  "data": {
    "chat": {
      "id": 5,
      "other_user": {...},
      "last_message": {...},
      "unread_count": 1
    }
  },
  "timestamp": "2025-01-17T11:00:00Z"
}
```

**Recipient Receives (Message Request):**
```json
{
  "type": "message_request_received",
  "data": {
    "request": {
      "id": 10,
      "sender": {...},
      "message": "Hi! I'd like to connect with you.",
      "status": "PENDING",
      "created_at": "2025-01-17T11:00:00Z"
    }
  },
  "timestamp": "2025-01-17T11:00:00Z"
}
```

---

#### 5. Accept Message Request (via WebSocket)
Accept a pending message request.

**Client Sends:**
```json
{
  "type": "accept_message_request",
  "data": {
    "request_id": 10
  }
}
```

**Server Response (Accepter):**
```json
{
  "type": "message_request_accepted",
  "data": {
    "request": {
      "id": 10,
      "sender": {...},
      "status": "ACCEPTED"
    },
    "chat": {
      "id": 6,
      "other_user": {...},
      "last_message": {...},
      "is_accepted": true
    },
    "message": {...}
  },
  "timestamp": "2025-01-17T11:30:00Z"
}
```

**Sender Receives:**
```json
{
  "type": "message_request_accepted",
  "data": {
    "request": {
      "id": 10,
      "status": "ACCEPTED"
    },
    "chat": {
      "id": 6,
      "other_user": {...},
      "last_message": {...}
    },
    "message": {...}
  },
  "timestamp": "2025-01-17T11:30:00Z"
}
```

---

#### 6. Reject Message Request (via WebSocket)
Reject a pending message request.

**Client Sends:**
```json
{
  "type": "reject_message_request",
  "data": {
    "request_id": 10
  }
}
```

**Server Response (Rejecter):**
```json
{
  "type": "message_request_rejected",
  "data": {
    "request": {
      "id": 10,
      "sender": {...},
      "status": "REJECTED",
      "updated_at": "2025-01-17T11:35:00Z"
    }
  },
  "timestamp": "2025-01-17T11:35:00Z"
}
```

**Sender Receives:**
```json
{
  "type": "message_request_rejected",
  "data": {
    "request": {
      "id": 10,
      "receiver": {...},
      "status": "REJECTED"
    }
  },
  "timestamp": "2025-01-17T11:35:00Z"
}
```

---

#### 7. Reaction Updates
Real-time reaction updates on messages.

**Add Reaction (Client):**
```json
{
  "type": "reaction_add",
  "data": {
    "message_id": 45,
    "emoji": "❤️"
  }
}
```

**Remove Reaction (Client):**
```json
{
  "type": "reaction_remove",
  "data": {
    "message_id": 45,
    "emoji": "❤️"
  }
}
```

**Server Broadcasts (Both Cases):**
```json
{
  "type": "reaction_updated",
  "data": {
    "id": 45,
    "sender": {...},
    "chat": 1,
    "content": "Hey! How are you?",
    "content_type": "TEXT",
    "reactions": {
      "user-id-1": "❤️",
      "user-id-2": "👍"
    },
    "created_at": "2025-01-17T10:30:00Z"
  },
  "timestamp": "2025-01-17T12:40:00Z"
}
```

**Notes:**
- Broadcast to chat group and personal channels of both users
- Reactions object maps user IDs to emoji strings
- One reaction per user per message (WhatsApp-style)

---

#### 8. Mark Message as Read
Mark a specific message as read.

**Client Sends:**
```json
{
  "type": "mark_message_read",
  "data": {
    "message_id": 45
  }
}
```

**Notes:**
- No broadcast response for this event
- Updates the `read_at` timestamp on the message
- Useful for single-message read receipts

---

#### 9. Ping/Pong (Connection Health)
Keep connection alive and check connectivity.

**Client Sends:**
```json
{
  "type": "ping",
  "data": {}
}
```

**Server Response:**
```json
{
  "type": "pong",
  "timestamp": "2025-01-17T12:45:00Z"
}
```

---

#### 10. Error Events
Server sends error events when operations fail.

**Error Structure:**
```json
{
  "type": "error",
  "error": "Chat does not exist",
  "timestamp": "2025-01-17T12:50:00Z"
}
```

**Common Errors:**
- `"Missing chat_id or content"` - Required fields missing
- `"Chat does not exist"` - Invalid chat ID
- `"No access to this chat"` - User not authorized
- `"Cannot send message to this chat"` - Chat inactive or not accepted
- `"Failed to send message"` - General message creation error
- `"Recipient not found"` - Invalid recipient ID
- `"Failed to accept request"` - Error accepting message request
- `"Failed to reject request"` - Error rejecting message request

---

## Error Responses

### HTTP Error Codes

#### 400 Bad Request
Missing or invalid parameters.

```json
{
  "error": "chat_id is required"
}
```

```json
{
  "error": "File too large. Max size for IMAGE: 10.0MB"
}
```

```json
{
  "content": ["This field is required."]
}
```

---

#### 403 Forbidden
User not authorized to perform action.

```json
{
  "error": "Not authorized"
}
```

```json
{
  "error": "Chat pending acceptance"
}
```

```json
{
  "error": "You are not a participant of this group"
}
```

---

#### 404 Not Found
Resource does not exist.

```json
{
  "error": "Chat not found or access denied"
}
```

```json
{
  "error": "User not found"
}
```

---

#### 500 Internal Server Error
Server-side error occurred.

```json
{
  "error": "Failed to create message"
}
```

---

## Data Models

### Chat Object
```json
{
  "id": 1,
  "other_user": {
    "id": "uuid-string",
    "display_name": "Jane Doe",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg",
    "bio": "Hello there!",
    "is_online": true
  },
  "last_message": {
    "id": 45,
    "sender": {...},
    "content": "Hey! How are you?",
    "content_type": "TEXT",
    "created_at": "2025-01-17T10:30:00Z",
    "read_at": null,
    "reactions": {},
    "reply_to": null
  },
  "unread_count": 3,
  "last_activity": "2025-01-17T10:30:00Z",
  "is_active": true,
  "is_accepted": true,
  "rate_user": false
}
```

**Fields:**
- `id` (integer): Unique chat identifier
- `other_user` (object): The other participant's details
- `last_message` (object/null): Most recent message in chat
- `unread_count` (integer): Number of unread messages
- `last_activity` (datetime): Last activity timestamp
- `is_active` (boolean): Whether chat is active
- `is_accepted` (boolean): Whether chat request was accepted
- `rate_user` (boolean): Can current user rate the other participant

---

### Message Object
```json
{
  "id": 45,
  "sender": {
    "id": "uuid-string",
    "display_name": "Jane Doe",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "chat": 1,
  "content": "Hey! How are you?",
  "content_type": "TEXT",
  "created_at": "2025-01-17T10:30:00Z",
  "read_at": null,
  "reactions": {
    "user-id-1": "❤️",
    "user-id-2": "👍"
  },
  "reply_to": {
    "id": 44,
    "sender": {...},
    "content": "Hi there!",
    "content_type": "TEXT"
  },
  "is_mine": false
}
```

**Fields:**
- `id` (integer): Unique message identifier
- `sender` (object): Message sender details
- `chat` (integer): Chat ID this message belongs to
- `content` (string): Message content (text or media URL)
- `content_type` (string): Type of message (TEXT, IMAGE, VIDEO, AUDIO, GIF, STICKER)
- `created_at` (datetime): Creation timestamp
- `read_at` (datetime/null): When message was read
- `reactions` (object): Map of user IDs to emoji reactions
- `reply_to` (object/null): Message being replied to
- `is_mine` (boolean): Whether current user sent this message

**Content Format for Media:**
- Format: `{media_url} |{caption}`
- Example: `https://api.joinhafar.com/media/chats/1/image/uuid.jpg |Check this out!`
- Caption is optional (can be just the URL)

---

### Message Request Object
```json
{
  "id": 10,
  "sender": {
    "id": "uuid-string",
    "display_name": "John Smith",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg",
    "bio": "Software developer",
    "is_premium": false
  },
  "receiver": {
    "id": "uuid-string-2",
    "display_name": "Jane Doe",
    "profile_photo": "https://api.joinhafar.com/media/photo2.jpg"
  },
  "message": "Hi! I'd like to connect with you.",
  "status": "PENDING",
  "created_at": "2025-01-17T11:00:00Z",
  "updated_at": "2025-01-17T11:00:00Z"
}
```

**Fields:**
- `id` (integer): Unique request identifier
- `sender` (object): User who sent the request
- `receiver` (object): User who received the request
- `message` (string): Initial message content
- `status` (string): Request status (PENDING, ACCEPTED, REJECTED)
- `created_at` (datetime): Creation timestamp
- `updated_at` (datetime): Last update timestamp

---

### Group Chat Object
```json
{
  "id": 2,
  "name": "Weekend Hangout",
  "description": "Planning our weekend activities",
  "admin": {
    "id": "uuid-string",
    "display_name": "John Smith",
    "profile_photo": "https://api.joinhafar.com/media/photo.jpg"
  },
  "avatar": "https://api.joinhafar.com/media/group_avatars/photo.jpg",
  "participants": [
    {
      "id": "uuid-string-1",
      "display_name": "John Smith"
    },
    {
      "id": "uuid-string-2",
      "display_name": "Jane Doe"
    }
  ],
  "participants_count": 5,
  "created_at": "2025-01-15T10:00:00Z",
  "last_activity": "2025-01-17T09:15:00Z",
  "is_public": false,
  "invite_code": null
}
```

**Fields:**
- `id` (integer): Unique group identifier
- `name` (string): Group name
- `description` (string): Group description
- `admin` (object): Group admin/creator
- `avatar` (string/null): Group avatar URL
- `participants` (array): List of group members
- `participants_count` (integer): Total number of participants
- `created_at` (datetime): Creation timestamp
- `last_activity` (datetime): Last activity timestamp
- `is_public` (boolean): Whether group is public
- `invite_code` (string/null): Invite code for joining group

---

## Content Types

### Message Content Types
- `TEXT` - Plain text message
- `IMAGE` - Image file
- `VIDEO` - Video file
- `AUDIO` - Audio/voice message
- `GIF` - Animated GIF
- `STICKER` - Sticker image

### Media File Constraints

#### IMAGE
- **Max Size:** 10MB
- **Formats:** .jpg, .jpeg, .png, .gif, .webp

#### VIDEO
- **Max Size:** 100MB
- **Formats:** .mp4, .mov, .avi, .webm, .mkv

#### AUDIO
- **Max Size:** 10MB
- **Formats:** .mp3, .wav, .ogg, .m4a, .aac, .opus, .wma, .webm

#### GIF
- **Max Size:** 5MB
- **Format:** .gif

---

## Best Practices

### Performance Optimization
1. **WebSocket Connection:** Maintain persistent connection for real-time updates
2. **Pagination:** Always use pagination for message lists
3. **Caching:** Cache user profiles and chat lists when appropriate
4. **Media Compression:** Compress media files before upload
5. **Lazy Loading:** Load messages on demand as user scrolls

### Real-time Updates
1. **Join Chat Rooms:** Always join chat room before sending messages via WebSocket
2. **Reconnection Logic:** Implement automatic reconnection on disconnect
3. **Event Handling:** Handle all WebSocket event types appropriately
4. **Optimistic Updates:** Use `local_id` for optimistic UI updates

### Message Request Flow
1. **Check Status First:** Use `/status/` endpoint before sending first message
2. **Handle Both Cases:** Be prepared for both direct chat and message request responses
3. **UI Feedback:** Show clear UI for pending, accepted, and rejected states
4. **Batch Operations:** Use batch actions for bulk accepting/rejecting requests

### Security
1. **Authentication:** Always include valid auth tokens
2. **Authorization:** Verify user has access to chats before operations
3. **Input Validation:** Validate file types and sizes before upload
4. **Rate Limiting:** Respect rate limits (see below)

### User Experience
1. **Typing Indicators:** Implement typing start/stop events
2. **Read Receipts:** Show message read status clearly
3. **Reactions:** Support one-tap reactions with emoji picker
4. **Media Preview:** Show media thumbnails in chat list
5. **Unread Badges:** Display unread message counts prominently

---

## Rate Limiting

### Standard Endpoints
- **List/Retrieve:** 100 requests per minute
- **Create Message:** 10 requests per minute (via REST API)
- **Send Message:** Unlimited (via WebSocket)
- **Reactions:** 60 requests per minute
- **Message Requests:** 20 create requests per minute

### Rate Limit Response
When rate limit is exceeded:

**Status Code:** 429 Too Many Requests

```json
{
  "detail": "Request was throttled. Expected available in 45 seconds."
}
```

### Recommendations
- Use WebSocket for sending messages (no rate limit)
- Implement client-side throttling for reactions
- Cache frequently accessed data
- Use batch operations when available

---

## Pagination

All list endpoints support pagination:

### Query Parameters
- `page`: Page number (default: 1)
- `page_size`: Results per page (default: varies by endpoint)

### Response Format
```json
{
  "count": 150,
  "next": "https://api.com/chat/messages/?chatId=1&page=2",
  "previous": null,
  "results": [...]
}
```

### Recommended Page Sizes
- **Messages:** 50 per page
- **Chats:** 20 per page
- **Message Requests:** 20 per page
- **Group Messages:** 50 per page

---

## Notification Integration

### Push Notifications
Automatic push notifications are sent for:

1. **New Messages:** When user receives a new message (if offline)
2. **Message Requests:** When user receives a new message request
3. **Request Accepted:** When your message request is accepted
4. **Group Messages:** When someone messages in a group you're in

### Notification Payload
```json
{
  "title": "New Message",
  "body": "Jane Doe: Hey! How are you?",
  "data": {
    "type": "chat_message",
    "chat_id": 1,
    "message_id": 45,
    "sender_id": "uuid-string"
  }
}
```

### Handling Notifications
- Deep link to specific chat when notification tapped
- Update unread counts in real-time
- Respect user notification preferences
- Group multiple messages from same chat

---

## Testing Guide

### Postman Examples

#### 1. Send Text Message (REST)
```http
POST /messages/
Authorization: Bearer {token}
Content-Type: application/json

{
  "chat_id": 1,
  "content": "Hello! How are you?",
  "content_type": "TEXT"
}
```

#### 2. Send Image Message (REST)
```http
POST /messages/
Authorization: Bearer {token}
Content-Type: multipart/form-data

chat_id: 1
content_type: IMAGE
file: <image_file>
caption: Check this out!
```

#### 3. WebSocket Message
```json
{
  "type": "chat_message",
  "data": {
    "chat_id": 1,
    "content": "Hello via WebSocket!",
    "content_type": "TEXT",
    "local_id": "temp-123"
  }
}
```

#### 4. Accept Message Request
```http
POST /message-requestsv2/10/respond/
Authorization: Bearer {token}
Content-Type: application/json

{
  "action": "accepted"
}
```

### WebSocket Testing (Postman)
1. Create new WebSocket request
2. URL: `wss://api.joinhafar.com/ws/`
3. Add auth token in headers or query params
4. Send JSON messages as shown in examples above

---

## Troubleshooting

### Common Issues

#### Messages Not Sending
- **Check:** Is chat active and accepted?
- **Check:** Is user authenticated?
- **Check:** Is WebSocket connected?
- **Solution:** Join chat room before sending

#### Reactions Not Updating
- **Check:** Is message_id valid?
- **Check:** Is emoji supported?
- **Solution:** Use WebSocket events, not REST API

#### Media Upload Fails
- **Check:** File size within limits?
- **Check:** File extension allowed?
- **Check:** Using multipart/form-data?
- **Solution:** Compress media before upload

#### Message Request Not Creating Chat
- **Check:** Is action "accepted" (not "accept")?
- **Check:** Is user the receiver of the request?
- **Solution:** Verify request_id and user authorization

#### WebSocket Disconnects Frequently
- **Check:** Network stability
- **Check:** Implement ping/pong keepalive
- **Solution:** Send ping every 30-60 seconds

---

## Migration Guide

### From V1 to V2

#### Chat Endpoints
- **Old:** `GET /chats/`
- **New:** `GET /chatsv2/`
- **Changes:** Updated serializer with more user details

#### Message Requests
- **Old:** `GET /message-requests/`
- **New:** `GET /message-requestsv2/`
- **Changes:** Improved filtering and sorting options

#### Status Check
- **Old:** Returns different response structure
- **New:** Consistent response with `type` field

#### Key Improvements in V2
1. Better serialization with more user details
2. Consistent response formats
3. Improved error handling
4. Enhanced filtering options
5. Better WebSocket integration

---

## Support & Resources

### Documentation
- Main API Docs: https://docs.api.joinhafar.com
- WebSocket Guide: https://docs.api.joinhafar.com/websocket
- Authentication: https://docs.api.joinhafar.com/auth

### Support
- Email: support@joinhafar.com
- Developer Portal: https://developer.joinhafar.com

### Status Page
- System Status: https://status.joinhafar.com
- API Uptime: 99.9% SLA