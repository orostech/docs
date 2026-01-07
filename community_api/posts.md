# Posts API

Base Endpoint: `/posts/`

## 1. Get Feed (List Posts)
`GET /posts/`

**Query Parameters**
| Parameter | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `community` | UUID | No | Filter by community ID |
| `user` | UUID | No | Filter by user ID |
| `page` | int | No | Page number |

## 2. Create Post
`POST /posts/`

**Content-Type**: `multipart/form-data`
(Because of file uploads)

**Fields**
| Field | Type | Required | Description |
| :--- | :--- | :--- | :--- |
| `content` | string | Yes | Text content of the post |
| `community` | UUID | Yes | ID of the community to post in |
| `media_files` | file list | No | List of images/videos |
| `is_anonymous` | boolean | No | Set true for anonymous post (Premium only) |

## 3. Post Actions

### Like Post
- **URL**: `/posts/{id}/like/`
- **Method**: `POST`

### Save Post
- **URL**: `/posts/{id}/save/`
- **Method**: `POST`

### Report Post
- **URL**: `/posts/{id}/report/`
- **Method**: `POST`
- **Body**:
```json
{
  "reason": "spam",
  "description": "Details..."
}
```

## 4. Special Lists

### My Posts
- **URL**: `/posts/my_posts/`
- **Method**: `GET`

### Saved Posts
- **URL**: `/posts/saved/`
- **Method**: `GET`

### Popular Posts
- **URL**: `/posts/popular/`
- **Method**: `GET`
- **Query Params**: 
    - `community`: **Slug** (Required) - e.g. `tech-enthusiasts`
    - `hours`: int (default 24)
