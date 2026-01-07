# News API

## 1. Get News Feed
`GET /load-news/`

Fetches active news articles from the database, typically filtered to the last 7 days.

**Response**
```json
[
  {
    "id": 1,
    "title": "Tech News",
    "content": "...",
    "image_url": "...",
    "source_url": "...",
    "published_at": "2023-10-27T10:00:00Z",
    "time_since": "2 hours ago"
  }
]
```

## 2. Trigger Fetch (Admin/System)
`POST /fetch-news/`

Triggers the backend scraper to fetch new articles from external sources.

**Response**
```json
{
    "message": "News fetched successfully"
}
```
