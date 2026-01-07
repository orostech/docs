# Comments API

Base Endpoint: `/comments/`

## 1. List Comments
`GET /comments/`

**Query Parameters**
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `post` | UUID | **Yes** (for top level) | ID of the post |
| `parent` | UUID | No | ID of parent comment (to fetch replies) |
| `page` | int | No | Page number |

## 2. Create Comment
`POST /comments/`

**Body**
```json
{
  "post": "uuid",
  "parent": "uuid",  // Optional (for replies)
  "content": "Great post!",
  "is_anonymous": false
}
```

## 3. Like Comment
- **URL**: `/comments/{id}/like/`
- **Method**: `POST`
- **Response**:
```json
{
  "action": "liked",
  "like_count": 5
}
```
