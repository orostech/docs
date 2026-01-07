# Communities API

Base Endpoint: `/communities/`

## Endpoints

### 1. List Communities
Get a paginated list of all active communities, ordered by member count.

- **URL**: `/communities/`
- **Method**: `GET`
- **Auth**: Required

**Query Parameters**
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `search` | string | No | Search communities by name |
| `page` | int | No | Page number |

**Response**
```json
{
  "results": [
    {
      "id": "uuid",
      "name": "Tech Enthusiasts",
      "slug": "tech-enthusiasts",
      "description": "...",
      "member_count": 150,
      "is_member": true,
      "cover_image_url": "https://..."
    }
  ]
}
```

### 2. Create Community
Create a new community.

- **URL**: `/communities/`
- **Method**: `POST`
- **Auth**: Required

**Body**
```json
{
  "name": "Design Hub",
  "description": "For UI/UX designers",
  "is_private": false,
  "require_approval": false
}
```

### 3. Get Community Details
- **URL**: `/communities/{id}/`
- **Method**: `GET`

### 4. Join Community
- **URL**: `/communities/{id}/join/`
- **Method**: `POST`

**Response**
```json
{
  "message": "Successfully joined community"
}
```

### 5. Leave Community
- **URL**: `/communities/{id}/leave/`
- **Method**: `POST`

### 6. Get Recommended Communities
AI-driven recommendations based on user profile.

- **URL**: `/communities/recommended/`
- **Method**: `GET`

### 7. My Communities
Get list of communities the current user has joined.

- **URL**: `/communities/my_communities/`
- **Method**: `GET`

### 8. Get Members
List members of a specific community.

- **URL**: `/communities/{id}/members/`
- **Method**: `GET`
- **Query Params**: `page`, `page_size`
