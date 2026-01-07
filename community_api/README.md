# Community API Documentation

## Overview
This directory contains documentation for the Hafar Community module, covering all social features including communities, feed posts, comments, news, and the anonymous ventroom.

## Modules

| Module | Description | File |
| :--- | :--- | :--- |
| **Communities** | Management of communities, memberships, and discovery. | [communities.md](./communities.md) |
| **Posts** | Main feed posts, rich media, and social interactions (like/save). | [posts.md](./posts.md) |
| **Comments** | Threaded discussions on posts. | [comments.md](./comments.md) |
| **Ventroom** | Anonymous safe space with real-time features and content moderation. | [ventroom.md](./ventroom.md) |
| **News** | Aggregated news articles. | [news.md](./news.md) |

## Base Configuration

### Base URL
`https://api.joinhafar.com` (Production)
`http://localhost:8000` (Development)

Note: Most community endpoints are mounted at the root of the API or under specific prefixes.
- Communities: `/communities/`
- Posts: `/posts/`
- Comments: `/comments/`
- Ventroom: `/ventroom/`

### Authentication
All endpoints require authentication unless explicitly valid for public access (rare).
Include the JWT Access Token in the request header:

```http
Authorization: Bearer <your_access_token>
```

### Standard Response Format
**Success (200 OK / 201 Created)**
```json
{
  "id": "123",
  "message": "Operation successful",
  ...data
}
```

**Error (400 Bad Request / 403 Forbidden)**
```json
{
  "error": "Error description",
  "detail": "Detailed error message"
}
```

### Pagination
List endpoints support standard pagination query parameters.
- `page`: Page number (default: 1)
- `page_size`: Items per page (default: 20)

**Paginated Response:**
```json
{
  "count": 100,
  "next": "http://api.../?page=3",
  "previous": "http://api.../?page=1",
  "results": [ ... ]
}
```
