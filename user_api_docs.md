# Users Profile API Documentation

---

## Table of Contents
1. [Authentication Endpoints](#authentication-endpoints)
2. [Profile Management](#profile-management)
3. [Photo Management](#photo-management)
4. [User Blocking](#user-blocking)
5. [Ratings](#ratings)
6. [Social Data](#social-data)
7. [Error Responses](#error-responses)

---

## Authentication Endpoints

### 1. Register User
Create a new user account.

**Endpoint:** `POST /auth/register/`

**Authentication:** None required

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "strongpassword123",
  "device_platform": "ANDROID",
  "referral_code": "ABC12345"
}
```

**Parameters:**
- `email` (required): Valid email address
- `password` (required): User password (min 8 characters recommended)
- `device_platform` (optional): One of `ANDROID`, `IOS`, `WEB` (default: `WEB`)
- `referral_code` (optional): Referral code from another user

**Success Response (201):**
```json
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "id": "uuid-string",
  "username": "user-123",
  "display_name": "Unknown",
  "email_verified": false,
  "bio": "",
  "voice_bio": "",
  "photos": [],
  ...
}
```

**Error Response (400):**
```json
{
  "error": "Email already in use."
}
```

**Rate Limit:** 5 requests per minute

---

### 2. Login
Authenticate an existing user.

**Endpoint:** `POST /auth/login/`

**Authentication:** None required

**Request Body:**
```json
{
  "email": "user@example.com",
  "password": "strongpassword123",
  "device_platform": "ANDROID"
}
```

**Success Response (200):**
```json
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "id": "uuid-string",
  "username": "user-123",
  ...
}
```

**Error Response (401):**
```json
{
  "error": "Invalid credentials"
}
```

**Rate Limit:** 5 requests per minute

---

### 3. Google Login
Authenticate using Google OAuth.

**Endpoint:** `POST /auth/google-login/`

**Authentication:** None required

**Request Body:**
```json
{
  "token": "google-id-token-string",
  "referral_code": "ABC12345"
}
```

**Parameters:**
- `token` (required): Google ID token from OAuth flow
- `referral_code` (optional): Referral code from another user

**Success Response (200):**
```json
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "access": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "id": "uuid-string",
  "email_verified": true,
  ...
}
```

**Rate Limit:** 5 requests per minute

---

### 4. Refresh Token
Get a new access token using refresh token.

**Endpoint:** `POST /auth/token/refresh/`

**Authentication:** None required

**Request Body:**
```json
{
  "refresh": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

**Success Response (200):**
```json
{
  "access": "eyJ0eXAiOiJKV1QiLCJhbGc..."
}
```

---

### 5. Password Reset Request
Request a password reset OTP.

**Endpoint:** `POST /auth/password-reset/request/`

**Authentication:** None required

**Request Body:**
```json
{
  "email": "user@example.com"
}
```

**Success Response (200):**
```json
{
  "message": "OTP sent to your email."
}
```

**Rate Limit:** 5 requests per minute

---

### 6. Password Reset Verify
Verify the OTP code.

**Endpoint:** `POST /auth/password-reset/verify/`

**Authentication:** None required

**Request Body:**
```json
{
  "email": "user@example.com",
  "code": "123456"
}
```

**Success Response (200):**
```json
{
  "message": "OTP verified."
}
```

**Error Response (400):**
```json
{
  "error": "Invalid or expired OTP."
}
```

---

### 7. Password Reset Confirm
Set new password after verification.

**Endpoint:** `POST /auth/password-reset/confirm/`

**Authentication:** None required

**Request Body:**
```json
{
  "email": "user@example.com",
  "code": "123456",
  "new_password": "newstrongpassword123"
}
```

**Success Response (200):**
```json
{
  "message": "Password reset successfully."
}

```

---


### 8 . Change Password
Change password for authenticated user.
**Endpoint:** `POST /auth/change-password/`
**Authentication:** Required
**Request Body:**
```json
{
  "old_password": "oldpassword123",
  "new_password": "newstrongpassword123"
}
``` 
**Success Response (200):**
```json
{
  "message": "Password changed successfully."
}
``` 
**Error Response (400):**
```json
{
  "error": "Old password is incorrect."
}
```
---

## Profile Management

### 10. Get Current User Profile
Retrieve authenticated user's profile.

**Endpoint:** `GET /profiles/me/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "id": "uuid-string",
  "username": "john_doe",
  "first_name": "John",
  "last_name": "Doe",
  "display_name": "John D",
  "bio": "Love traveling and music",
  "voice_bio": "...",
  "date_of_birth": "1995-05-15",
  "age": 29,
  "gender": "M",
  "photos": [
    {
      "id": 1,
      "image": "....",
      "is_primary": true,
      "order": 0,
      "caption": ""
    }
  ],
  "interests": ["Travel", "Music", "Sports"],
  "profession": "Software Engineer",
  "relationship_goal": "LTR",
  "relationship_status": "S",
  "body_type": "AT",
  "complexion": "MD",
  "height": 175,
  "weight": 70,
  "latlng": {
    "latitude": 7.3775,
    "longitude": 3.9470
  },
  "address": "Ibadan, Nigeria",
  "state": "Oyo",
  "country": "Nigeria",
  "selected_country": "NG",
  "is_premium": false,
  "online_status": "ONLINE",
  "is_online": true,
  "average_rating": 4.2,
  "total_matches": 12,
  "total_likes_given": 45,
  "total_visits": 89,
  "subscription": null,
  "vanish_mode": false, // New field added (default: false),
  "show_online_status": true, // New field added (default: true),
  "show_last_seen": true, // New field added (default: true),
  "show_distance": true, // New field added (default: true),
  "profile_visibility": "VE", // New field added (default: "VE")
  "wallet": {
    "balance": "0.00"
  }
}
```

---

### 9. Update Current User Profile
Update authenticated user's profile.

**Endpoint:** `POST /profiles/updateme/`

**Authentication:** Required

**Content-Type:** `multipart/form-data` (for file uploads) or `application/json`

**Request Body (FormData for voice bio):**
```
display_name: "John Doe"
bio: "Updated bio text"
voice_bio: <audio_file>
date_of_birth: "1995-05-15"
profession: "Engineer"
latitude: 7.3775
longitude: 3.9470
interests: ["Travel", "Music"]
```

**Request Body (JSON for text updates):**
```json
{
  "display_name": "John Doe",
  "bio": "Updated bio text",
  "profession": "Engineer",
  "relationship_goal": "LTR",
  "height": 175,
  "weight": 70,
  "latitude": 7.3775,
  "longitude": 3.9470,
  "interests": ["Travel", "Music"],
  "selected_state": 1,
  "selected_lga": 5
}
```

**Voice Bio Requirements:**
- **Formats:** MP3, WAV, M4A, OGG, WEBM
- **Max Size:** 3MB (~15 seconds)
- **Field Name:** `voice_bio`

**Success Response (200):**
```json
{
  "id": "uuid-string",
  "display_name": "John Doe",
  "bio": "Updated bio text",
  "voice_bio": "..",
  ...
}
```

**Error Response (400):**
```json
{
  "voice_bio": [
    "Voice bio file size should not exceed 3MB (approximately 15 seconds)."
  ]
}
```

---

### 10. Delete Voice Bio
Remove voice bio from profile.

**Endpoint:** `DELETE /profiles/delete-voice-bio/`

**Authentication:** Required

**Success Response (200):**
```json
{
  "message": "Voice bio deleted successfully"
}
```

**Error Response (404):**
```json
{
  "error": "No voice bio to delete"
}
```

---

### 11. Get User Profile by ID
View another user's profile.

**Endpoint:** `GET /profiles/{user_id}/`

**Authentication:** Required

**Parameters:**
- `user_id`: UUID of the user

**Success Response (200):**
```json
{
  "id": "uuid-string",
  "username": "jane_smith",
  "display_name": "Jane",
  "bio": "Adventure seeker",
  "voice_bio": "...",
  "age": 27,
  "gender": "F",
  "photos": [...],
  "interests": ["Adventure", "Photography"],
  "is_premium": true,
  "online_status": "ONLINE",
  "average_rating": 4.5,
  "interaction_status": "LIKE"
}
```

**Note:** This endpoint automatically records a visit when viewing another user's profile.

---

### 12. Search Users
Search for users by username or display name.

**Endpoint:** `GET /profiles/search_users/?q=john`

**Authentication:** Required

**Query Parameters:**
- `q` (required): Search query (min 1 character)
- `page` (optional): Page number
- `page_size` (optional): Results per page

**Success Response (200):**
```json
{
  "count": 15,
  "next": "https://api.com/profiles/search_users/?page=2&q=john",
  "previous": null,
  "results": [
    {
      "id": "uuid-string",
      "username": "john_doe",
      "display_name": "John",
      "profile_photo": "https://domain.com/media/photo.jpg",
      "age": 29,
      "last_active_time": "Online",
      "is_premium": false,
      "latlng": {
        "latitude": 7.3775,
        "longitude": 3.9470
      },
      "address": "Ibadan, Nigeria"
    }
  ]
}
```

---

### 13. Change Account Status
Deactivate or delete account.

**Endpoint:** `POST /auth/account-status/`

**Authentication:** Required

**Request Body:**
```json
{
  "status": "D",
  "reason": "Taking a break"
}
```

**Parameters:**
- `status` (required): `D` (Deactivate) or `DE` (Delete permanently)
- `reason` (optional): Reason for status change

**Success Response (200):**
```json
{
  "status": "User deletion scheduled."
}
```

---

## Photo Management

### 14. List User Photos
Get all photos for authenticated user.

**Endpoint:** `GET /photos/`

**Authentication:** Required

**Success Response (200):**
```json
[
  {
    "id": 1,
    "image": "",
    "is_primary": true,
    "order": 0,
    "caption": "At the beach",
    "created_at": "2025-01-15T10:30:00Z",
    "updated_at": "2025-01-15T10:30:00Z"
  },
  {
    "id": 2,
    "image": "",
    "is_primary": false,
    "order": 1,
    "caption": "",
    "created_at": "2025-01-16T14:20:00Z",
    "updated_at": "2025-01-16T14:20:00Z"
  }
]
```

---

### 15. Upload Photo
Add a new photo to user's profile.

**Endpoint:** `POST /photos/`

**Authentication:** Required

**Content-Type:** `multipart/form-data`

**Request Body:**
```
image: <file>
caption: "My new photo"
is_primary: false
order: 2
```

**Photo Limits:**
- **Free users:** 2 photos maximum
- **Premium users:** 6 photos maximum
- **Formats:** JPG, JPEG, PNG

**Success Response (201):**
```json
{
  "id": 3,
  "image": "",
  "is_primary": false,
  "order": 2,
  "caption": "My new photo",
  "created_at": "2025-01-17T09:15:00Z",
  "updated_at": "2025-01-17T09:15:00Z"
}
```

**Error Response (400):**
```json
{
  "detail": "Max 2 photos allowed. Upgrade for more."
}
```

---

### 16. Update Photo
Update photo details.

**Endpoint:** `PATCH /photos/{photo_id}/`

**Authentication:** Required

**Request Body:**
```json
{
  "caption": "Updated caption",
  "is_primary": true,
  "order": 0
}
```

**Success Response (200):**
```json
{
  "id": 3,
  "image": "",
  "is_primary": true,
  "order": 0,
  "caption": "Updated caption"
}
```

---

### 17. Delete Photo
Remove a photo from profile.

**Endpoint:** `DELETE /photos/{photo_id}/`

**Authentication:** Required

**Success Response (204):** No content

---

## User Blocking

### 18. List Blocked Users
Get all users blocked by authenticated user.

**Endpoint:** `GET /blocks/`

**Authentication:** Required

**Success Response (200):**
```json
[
  {
    "id": 1,
    "blocked_user": "uuid-of-blocked-user",
    "reason": "Inappropriate behavior",
    "created_at": "2025-01-10T12:00:00Z"
  }
]
```

---

### 19. Block User
Block another user.

**Endpoint:** `POST /blocks/`

**Authentication:** Required

**Request Body:**
```json
{
  "blocked_user": "uuid-of-user-to-block",
  "reason": "Spam messages"
}
```

**Success Response (201):**
```json
{
  "id": 2,
  "blocked_user": "uuid-of-blocked-user",
  "reason": "Spam messages",
  "created_at": "2025-01-17T15:30:00Z"
}
```

---

### 20. Unblock User
Remove a user from block list.

**Endpoint:** `DELETE /blocks/{block_id}/`

**Authentication:** Required

**Success Response (204):** No content

---

## Ratings

### 21. Create/Update Rating
Rate another user.

**Endpoint:** `POST /ratings/`

**Authentication:** Required

**Request Body:**
```json
{
  "rated_user": "uuid-of-user",
  "value": 5
}
```

**Parameters:**
- `rated_user` (required): UUID of user to rate
- `value` (required): Integer from 1 to 5
- `chat_id` (optional): Related chat ID

**Success Response (201/200):**
```json
{
  "id": 1,
  "rated_user": "uuid-of-rated-user",
  "value": 5,
  "created_at": "2025-01-17T16:00:00Z"
}
```

---

### 22. List User Ratings
Get ratings given and received by authenticated user.

**Endpoint:** `GET /ratings/`

**Authentication:** Required

**Success Response (200):**
```json
[
  {
    "id": 1,
    "rated_user": "uuid-string",
    "value": 5,
    "created_at": "2025-01-17T16:00:00Z"
  }
]
```

---

## Social Data

### 23. List States
Get all available states.

**Endpoint:** `GET /states/`

**Authentication:** Optional

**Query Parameters:**
- `page` (optional): Page number
- `page_size` (optional): Results per page (default: 50, max: 100)

**Success Response (200):**
```json
{
  "count": 37,
  "next": null,
  "previous": null,
  "results": [
    {
      "id": 1,
      "name": "Lagos"
    },
    {
      "id": 2,
      "name": "Oyo"
    }
  ]
}
```

---

### 24. List LGAs
Get Local Government Areas.

**Endpoint:** `GET /lgas/`

**Authentication:** Optional

**Query Parameters:**
- `state_id` (optional): Filter by state ID
- `page` (optional): Page number
- `page_size` (optional): Results per page

**Success Response (200):**
```json
{
  "count": 33,
  "results": [
    {
      "id": 1,
      "name": "Ibadan North",
      "state": 2
    },
    {
      "id": 2,
      "name": "Ibadan South",
      "state": 2
    }
  ]
}
```

---

### 25. List Interests
Get all available interests.

**Endpoint:** `GET /interests/`

**Authentication:** Optional

**Success Response (200):**
```json
{
  "data": [
    "Travel",
    "Music",
    "Sports",
    "Photography",
    "Cooking",
    "Reading",
    "Gaming"
  ]
}
```

---

### 26. Reverse Geocode
Convert coordinates to address.

**Endpoint:** `GET /reverse_geocode/`

**Authentication:** None required

**Query Parameters:**
- `lat` (required): Latitude
- `lon` (required): Longitude

**Example:**
```
GET /reverse_geocode/?lat=7.3775&lon=3.9470
```

**Success Response (200):**
```json
{
  "place_id": 123456,
  "lat": "7.3775",
  "lon": "3.9470",
  "display_name": "Ibadan, Oyo State, Nigeria",
  "address": {
    "city": "Ibadan",
    "state": "Oyo State",
    "country": "Nigeria",
    "country_code": "ng"
  }
}
```

---

### 27. Update User Data
Update device token and location.

**Endpoint:** `POST /update-data/`

**Authentication:** Required

**Request Body:**
```json
{
  "device_token": "firebase-device-token-string",
  "location": {
    "latitude": 7.3775,
    "longitude": 3.9470
  }
}
```

**Success Response (200):**
```json
{
  "status": "success"
}
```

---

## Error Responses

### Common Error Codes

**400 Bad Request**
```json
{
  "error": "Invalid data provided"
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
  "error": "Profile not found"
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
  "error": "An unexpected error occurred"
}
```

---

## Field Enumerations

### Gender Choices
- `M` - Male
- `F` - Female
- `O` - Other

### Relationship Goal
- `LTR` - Long Term Relationship
- `STR` - Short Term Relationship
- `F` - Friendship
- `NSR` - Not Sure

### Body Type
- `SL` - Slim
- `AT` - Athletic
- `AV` - Average
- `HV` - Heavy
- `MW` - Muscular/Well-built

### Complexion
- `LT` - Light
- `MD` - Medium
- `DK` - Dark

### Device Types
- `ANDROID`
- `IOS`
- `WEB`

### Online Status
- `ONLINE` - Active within 6 hours
- `AWAY` - Active within 8 hours
- `OFFLINE` - Inactive for 8+ hours

### Interaction Types
- `LIKE` - User liked
- `SUPER_LIKE` - User super liked
- `DISLIKE` - User disliked

---

## Rate Limiting

- **Authentication endpoints:** 5 requests per minute
- **Profile endpoints:** 10 requests per minute
- **Other endpoints:** Standard rate limits apply

---

## Notes

1. All timestamps are in UTC format (ISO 8601)
2. UUIDs are used for user identification
3. File uploads use `multipart/form-data`
4. JSON requests use `application/json`
5. Voice bios are automatically deleted when uploading a new one
6. Profile visits are tracked asynchronously
7. Deleted accounts cannot reuse the same email address

---
