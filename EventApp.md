# Event App Backend - API Documentation

## System Specifications

**Database:** PostgreSQL with Django ORM  
**Cache:** LocMemCache (in-memory)  
**Channel Layer:** InMemoryChannelLayer (WebSocket)  
**Server:** Daphne (ASGI)  
**Deployment:** Render  
**Static Files:** WhiteNoise

## System Requirements
```
Django==4.2.7
djangorestframework
djangorestframework-simplejwt
python-decouple
psycopg2-binary
channels
channels-redis
daphne
django-cors-headers
whitenoise
firebase-admin
dj-database-url
gunicorn
setuptools
```
## Base URLs

```
Production API: https://eventapp-backend-api.onrender.com/api
Production WS:  wss://eventapp-backend-api.onrender.com/ws
Development:    http://localhost:8000/api
```

## Authentication

All protected endpoints require:
```
Authorization: Bearer <token>
```

---

# Authentication

## Register User
`POST /api/user/register/`

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!",
  "name": "John Doe"
}
```

**Response (201):**
```json
{
  "message": "User registered successfully",
  "role": "user"
}
```

**Errors:** `400` Invalid email/password, email exists

---

## Register Admin
`POST /api/admin/register/`

**Request:**
```json
{
  "name": "Admin Name",
  "email": "admin@example.com",
  "password": "SecurePass123!",
  "organisation_name": "Company Inc"
}
```

**Response (201):**
```json
{
  "message": "Admin registered successfully",
  "role": "admin"
}
```

---

## Login User
`POST /api/user/login/`

**Request:**
```json
{
  "email": "user@example.com",
  "password": "SecurePass123!"
}
```

**Response (200):**
```json
{
  "message": "Login successful",
  "token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "refresh_token": "eyJ0eXAiOiJKV1QiLCJhbGc...",
  "role": "user"
}
```

**Errors:** `400` Already logged in, `401` Invalid password, `404` Email not found

---

## Login Admin
`POST /api/admin/login/`

**Request/Response:** Same as user login with `role: "admin"`

---

## Logout
`POST /api/logout/`

**Headers:** `Authorization: Bearer <token>`

**Response (200):**
```json
{
  "message": "Successfully logged out"
}
```

---

# Profile

## Get Profile
`GET /api/profile/`

**Headers:** `Authorization: Bearer <token>`

**Response (200) - User:**
```json
{
  "id": 1,
  "name": "John Doe",
  "email": "user@example.com",
  "user_type": "user",
  "subscription_type": "pro",
  "subscription_expiry": "2026-12-31T23:59:59Z",
  "professional_tagline": "Expert Speaker",
  "bio": "Experienced speaker",
  "website_url": "https://example.com",
  "profile_photo_url": "https://example.com/photo.jpg",
  "speaking_reel_url": "https://youtube.com/...",
  "location": "New York, NY",
  "expertise_areas": ["AI", "ML"],
  "past_engagements": [],
  "social_links": {}
}
```

**Response (200) - Admin:**
```json
{
  "id": 1,
  "name": "Admin Name",
  "email": "admin@example.com",
  "organisation_name": "Company Inc",
  "user_type": "admin"
}
```

---

## Update Profile
`PUT /api/profile/update/`

**Headers:** `Authorization: Bearer <token>`

**Request (User):**
```json
{
  "name": "John Doe",
  "professional_tagline": "Senior Speaker",
  "bio": "Updated bio",
  "location": "San Francisco, CA",
  "expertise_areas": ["AI", "ML", "Data Science"]
}
```

**Request (Admin):**
```json
{
  "name": "New Admin Name",
  "organisation_name": "New Company LLC"
}
```

**Response (200):**
```json
{
  "message": "Profile updated successfully",
  "profile": { }
}
```

---

# Subscription (User Only)

## Update Subscription
`POST /api/subscription/update/`

**Headers:** `Authorization: Bearer <token>`

**Request:**
```json
{
  "subscription_type": "pro",
  "subscription_expiry": "2026-12-31T23:59:59"
}
```

**Types:** `none`, `pro`, `ultra_pro`  
**Valid Transitions:** `none→pro/ultra_pro`, `pro→ultra_pro`, `any→none`

**Response (200):**
```json
{
  "message": "Successfully subscribed to pro",
  "previous_subscription": "none",
  "current_subscription": "pro",
  "subscription_expiry": "2026-12-31T23:59:59Z"
}
```

**Errors:** 
- `400` Same subscription, downgrade attempt
- `403` `{"error": "This endpoint is only available for users, not admins"}`

---

# Events (Admin)

## Create Event
`POST /api/events/create/`

**Headers:** `Authorization: Bearer <token>` (Admin)

**Errors:** 
- `400` Missing required fields
- `403` `{"error": "This endpoint is only available for admins"}`

**Request:**
```json
{
  "event_title": "Tech Conference 2026",
  "organization_name": "Tech Corp",
  "event_type": "keynote",
  "description": "Annual tech conference",
  "speaker_requirements": "5+ years experience",
  "event_date": "2026-06-15",
  "location": "New York Convention Center",
  "is_remote": false,
  "topics": ["AI", "ML"],
  "application_deadline": "2026-05-01T23:59:59Z"
}
```

**Types:** `keynote`, `panel`, `workshop`, `remote`, `other`

**Response (201):**
```json
{
    "message": "Event created as draft successfully",
    "event": {
        "id": 2,
        "organizer": 23,
        "organizer_name": "Admin Name",
        "organizer_email": "admin@example.com",
        "event_title": "Tech Conference 2026",
        "organization_name": "Tech Corp",
        "event_type": "keynote",
        "description": "Annual tech conference",
        "speaker_requirements": "5+ years experience",
        "event_date": "2026-06-15",
        "location": "New York Convention Center",
        "is_remote": false,
        "status": "draft",
        "topics": [
            "AI",
            "ML"
        ],
        "application_deadline": "2026-05-01T23:59:59Z",
        "created_at": "2026-01-18T10:25:58.284834Z",
        "updated_at": "2026-01-18T10:25:58.284844Z",
        "applications_count": 0,
        "is_saved": false,
        "user_application_status": null
    }
}
```

---

## Publish Event
`POST /api/events/<event_id>/publish/`

**Response (200):**
```json
{
  "message": "Event published successfully",
  "event": { }
}
```

---

## Unpublish Event
`POST /api/events/<event_id>/unpublish/`

**Response (200):**
```json
{
  "message": "Event unpublished successfully",
  "event": { }
}
```

---

## Update Event
`PUT /api/events/<event_id>/update/`

**Request:**
```json
{
  "event_title": "Updated Title",
  "description": "Updated description"
}
```

**Response (200):**
```json
{
  "message": "Event updated successfully",
  "event": { }
}
```

---

## Delete Event
`DELETE /api/events/<event_id>/delete/`

**Response (200):**
```json
{
  "message": "Event \"Tech Conference 2026\" deleted successfully"
}
```

---

## List Admin Events
`GET /api/events/admin/list/?status=published&event_type=keynote`

**Response (200):**
```json
{
  "success": true,
  "count": 5,
  "events": [ ]
}
```

---

# Events (User)

## Browse Events
`GET /api/events/browse/?event_type=keynote&location=New York&search=tech`

**Headers:** `Authorization: Bearer <token>` (User)

**Errors:** 
- `403` `{"error": "This endpoint is only available for users, not admins"}`

**Response (200):**
```json
{
  "success": true,
  "count": 10,
  "events": [
    {
      "id": 1,
      "event_title": "Tech Conference 2026",
      "status": "published",
      "is_saved": true,
      "user_application_status": "applied",
      "applications_count": 12
    }
  ]
}
```

---

## Get Event Details
`GET /api/events/<event_id>/`

**Response (200):**
```json
{
  "success": true,
  "event": { }
}
```

---

## Save Event
`POST /api/events/<event_id>/save/`

**Headers:** `Authorization: Bearer <token>` (User)

**Errors:** 
- `400` Event already saved
- `403` `{"error": "This endpoint is only available for users, not admins"}`
- `404` Event not found

**Response (201):**
```json
{
  "message": "Event saved successfully",
  "saved_event": { }
}
```

---

## Unsave Event
`DELETE /api/events/<event_id>/unsave/`

**Response (200):**
```json
{
  "message": "Event removed from saved list"
}
```

---

## List Saved Events
`GET /api/events/saved/`

**Response (200):**
```json
{
  "success": true,
  "count": 3,
  "saved_events": [ ]
}
```

---

# Applications (User)

## Apply to Event
`POST /api/events/<event_id>/apply/`

**Headers:** `Authorization: Bearer <token>` (User)

**Response (201):**
```json
{
  "message": "Application submitted successfully",
  "application": {
    "id": 1,
    "status": "applied",
    "event_title": "Tech Conference 2026"
  }
}
```

**Errors:** 
- `400` Not accepting, deadline passed, already applied
- `403` `{"error": "This endpoint is only available for users, not admins"}`

**Note:** Admin receives push notification

---

## Withdraw Application
`POST /api/applications/<application_id>/withdraw/`

**Response (200):**
```json
{
  "message": "Application withdrawn successfully",
  "application": { }
}
```

---

## List My Applications
`GET /api/applications/my-applications/?status=accepted`

**Response (200):**
```json
{
  "success": true,
  "count": 5,
  "applications": [ ]
}
```

---

# Applications (Admin)

## List Incoming Applications
`GET /api/applications/incoming/?status=applied&event_id=1`

**Headers:** `Authorization: Bearer <token>` (Admin)

**Errors:** 
- `403` `{"error": "This endpoint is only available for admins"}`

**Response (200):**
```json
{
  "success": true,
  "count": 8,
  "applications": [ ]
}
```

---

## Mark as Reviewed
`POST /api/applications/<application_id>/mark-reviewed/`

**Headers:** `Authorization: Bearer <token>` (Admin)

**Errors:** 
- `403` `{"error": "This endpoint is only available for admins"}`
- `404` Application not found

**Response (200):**
```json
{
  "message": "Application marked as reviewed",
  "application": { }
}
```

---

## Accept Application
`POST /api/applications/<application_id>/accept/`

**Response (200):**
```json
{
  "message": "Application accepted successfully",
  "application": { }
}
```

**Note:** Speaker receives push notification

---

## Reject Application
`POST /api/applications/<application_id>/reject/`

**Response (200):**
```json
{
  "message": "Application rejected",
  "application": { }
}
```

**Note:** Speaker receives push notification

---

## List Event Applications
`GET /api/events/<event_id>/applications/?status=applied`

**Response (200):**
```json
{
  "success": true,
  "event_id": 1,
  "count": 12,
  "applications": [ ]
}
```

---

# Chat

## Start Conversation
`POST /api/conversations/start/`

**Request:**
```json
{
  "to_type": "user",
  "to_id": 5
}
```

**Response (200):**
```json
{
  "success": true,
  "conversation_id": "a1b2c3d4e5f6g7h8",
  "created": true,
  "other_user": {
    "id": 5,
    "type": "user",
    "name": "Jane Smith"
  }
}
```

**Supported:** User↔User, Admin↔User, Admin↔Admin

---

## Get Conversations
`GET /api/conversations/`

**Response (200):**
```json
{
  "success": true,
  "conversations": [
    {
      "conversation_id": "a1b2c3d4",
      "other_participant": {
        "is_online": true,
        "last_seen": "2026-01-16T10:30:00Z"
      },
      "last_message": { },
      "unread_count": 3
    }
  ]
}
```

**Online:** `last_seen` within 2 minutes

---

## Get Messages
`GET /api/conversations/<conversation_id>/messages/`

**Response (200):**
```json
{
  "success": true,
  "conversation_id": "a1b2c3d4",
  "messages": [
    {
      "id": 1,
      "content": "Hello!",
      "status": "read",
      "is_mine": true
    }
  ]
}
```

**Statuses:** `sent`, `delivered`, `read`

---

## Search Users
`GET /api/users/search/?q=john&type=user`

**Response (200):**
```json
{
  "success": true,
  "results": [
    {
      "id": 5,
      "type": "user",
      "name": "John Smith",
      "is_online": true
    }
  ]
}
```

---

# WebSocket Chat

## Connect
```
wss://eventapp-backend-api.onrender.com/ws/chat/<conversation_id>/?token=<token>
```

## Client → Server

**Send Message:**
```json
{"type": "message", "message": "Hello!"}
```

**Typing:**
```json
{"type": "typing", "is_typing": true}
```

**Mark Read:**
```json
{"type": "mark_read", "message_ids": [1, 2, 3]}
```

## Server → Client

**New Message:**
```json
{"type": "new_message", "data": { }}
```

**Typing:**
```json
{"type": "typing", "is_typing": true}
```

**User Online/Offline:**
```json
{"type": "user_online", "user_id": 5}
```

**Error Codes:** `4001` Auth failed, `4003` Access denied, `1000` Normal close

---

# Push Notifications

## Save FCM Token
`POST /api/notifications/save-token/`

**Request:**
```json
{
  "fcm_token": "device_token",
  "device_type": "android"
}
```

**Response (200):**
```json
{
  "success": true,
  "message": "Token saved"
}
```

---

## Send Notification
`POST /api/notifications/send/`

**Request:**
```json
{
  "to_user_id": 5,
  "to_user_type": "user",
  "title": "Test",
  "body": "Message",
  "data": {}
}
```

**Response (200):**
```json
{
  "sent": true,
  "message": "Notification sent successfully"
}
```

---

## Automatic Notifications

**New Message:** Sent when recipient is NOT in chat  
**New Application:** Admin notified when user applies  
**Application Accepted:** User notified with 🎉  
**Application Rejected:** User notified politely

---

# Frontend Integration

## Authentication Service
```javascript
const API_BASE = 'https://eventapp-backend-api.onrender.com/api';

export const authService = {
  login: async (email, password, role = 'user') => {
    const endpoint = role === 'admin' ? '/admin/login/' : '/user/login/';
    const res = await fetch(`${API_BASE}${endpoint}`, {
      method: 'POST',
      headers: {'Content-Type': 'application/json'},
      body: JSON.stringify({email, password})
    });
    const data = await res.json();
    localStorage.setItem('token', data.token);
    return data;
  },

  getProfile: async () => {
    const res = await fetch(`${API_BASE}/profile/`, {
      headers: {'Authorization': `Bearer ${localStorage.getItem('token')}`}
    });
    return res.json();
  }
};
```

## WebSocket Chat
```javascript
export class ChatWebSocket {
  constructor(conversationId, token) {
    this.conversationId = conversationId;
    this.token = token;
  }

  connect(onMessage) {
    const url = `wss://eventapp-backend-api.onrender.com/ws/chat/${this.conversationId}/?token=${this.token}`;
    this.ws = new WebSocket(url);
    
    this.ws.onmessage = (e) => {
      const data = JSON.parse(e.data);
      onMessage(data);
    };
  }

  sendMessage(content) {
    this.ws.send(JSON.stringify({
      type: 'message',
      message: content
    }));
  }

  disconnect() {
    this.ws.close(1000);
  }
}
```

---

# Password Requirements

- Minimum 8 characters
- At least one uppercase letter
- At least one lowercase letter
- At least one digit
- At least one special character (@$!%*?&)

---

# Rate Limits

**Cache-based deduplication:** 3-second window for notifications  
**Connection pooling:** 600 seconds max age  
**Active chat tracking:** 5-minute expiry

---

# Error Handling

All endpoints return JSON with `error` field on failure:
```json
{"error": "Descriptive error message"}
```

Common status codes:
- `200` Success
- `201` Created
- `400` Bad Request
- `401` Unauthorized
- `403` Forbidden
- `404` Not Found
- `500` Server Error


# API Documentation - Applicant Profile Endpoint

## View Applicant Profile (Admin Only)

### Endpoint
```
GET /api/applications/<application_id>/applicant-profile/
```

### Description
Allows event organizers (admins) to view the full profile of users who have applied to their events. This endpoint provides comprehensive information about the applicant including their professional details, subscription status, and application information.

### Authentication
- **Required:** Yes
- **Type:** Bearer Token (Admin only)
- **Header:** `Authorization: Bearer <admin_access_token>`

### Authorization Rules
- Only admins can access this endpoint
- Admin must be the organizer of the event that the applicant applied to
- Cannot view profiles of applicants to other admins' events

### Path Parameters
| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `application_id` | integer | Yes | The ID of the application |

### Request Example
```bash
curl -X GET "http://localhost:8000/api/applications/123/applicant-profile/" \
  -H "Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9..."
```

### Success Response (200 OK)
```json
{
  "message": "Applicant profile retrieved successfully",
  "profile": {
    "id": 45,
    "name": "John Doe",
    "email": "john.doe@example.com",
    "user_type": "user",
    "subscription_type": "pro",
    "subscription_expiry": "2026-12-31T23:59:59",
    "professional_tagline": "Technology Expert & Keynote Speaker",
    "bio": "Professional speaker with 10+ years of experience in technology and innovation. Passionate about sharing knowledge and inspiring audiences.",
    "website_url": "https://johndoe.com",
    "profile_photo_url": "https://example.com/photos/john.jpg",
    "speaking_reel_url": "https://youtube.com/watch?v=example",
    "location": "New York, USA",
    "expertise_areas": [
      "Artificial Intelligence",
      "Cloud Computing",
      "Leadership",
      "Digital Transformation"
    ],
    "past_engagements": [
      {
        "event_name": "Tech Summit 2025",
        "role": "Keynote Speaker",
        "date": "2025-06-15",
        "location": "San Francisco, CA"
      },
      {
        "event_name": "Innovation Conference 2024",
        "role": "Panelist",
        "date": "2024-11-20",
        "location": "Boston, MA"
      }
    ],
    "social_links": {
      "linkedin": "https://linkedin.com/in/johndoe",
      "twitter": "https://twitter.com/johndoe",
      "github": "https://github.com/johndoe"
    },
    "application_info": {
      "application_id": 123,
      "event_title": "Tech Conference 2026",
      "status": "applied",
      "applied_at": "2026-01-15T10:30:00",
      "updated_at": "2026-01-15T10:30:00"
    }
  }
}
```

### Error Responses

#### 401 Unauthorized - No Token
```json
{
  "error": "No token provided"
}
```

#### 401 Unauthorized - Invalid Token
```json
{
  "error": "Invalid or expired token"
}
```

#### 403 Forbidden - User Account (Not Admin)
```json
{
  "error": "This endpoint is only available for admins"
}
```

#### 403 Forbidden - Not Event Organizer
```json
{
  "error": "You can only view profiles of applicants to your own events"
}
```

#### 404 Not Found - Application Not Found
```json
{
  "error": "Application not found"
}
```

### Response Fields

#### Profile Object
| Field | Type | Description |
|-------|------|-------------|
| `id` | integer | User's unique identifier |
| `name` | string | User's full name |
| `email` | string | User's email address |
| `user_type` | string | Always "user" for applicants |
| `subscription_type` | string | Subscription level: "none", "pro", or "ultra_pro" |
| `subscription_expiry` | string (ISO 8601) | Subscription expiration date/time (null if none) |
| `professional_tagline` | string | User's professional headline |
| `bio` | string | User's biography (max 500 characters) |
| `website_url` | string | User's website URL |
| `profile_photo_url` | string | URL to profile photo |
| `speaking_reel_url` | string | URL to speaking reel video |
| `location` | string | User's location |
| `expertise_areas` | array | List of expertise areas |
| `past_engagements` | array | List of past speaking engagements |
| `social_links` | object | Social media profile links |
| `application_info` | object | Application-specific information |

#### Application Info Object
| Field | Type | Description |
|-------|------|-------------|
| `application_id` | integer | Application ID |
| `event_title` | string | Title of the event applied to |
| `status` | string | Application status: "applied", "reviewed", "accepted", "rejected", "withdrawn" |
| `applied_at` | string (ISO 8601) | Application submission timestamp |
| `updated_at` | string (ISO 8601) | Last update timestamp |

### Use Cases

1. **Review Applicant Qualifications**
   - Admin reviews applicant's expertise areas and past engagements
   - Checks speaking reel and portfolio

2. **Contact Applicant**
   - Access email and social media links for communication
   - View website for additional information

3. **Verify Subscription Status**
   - Check if applicant has active subscription
   - View subscription tier for premium features

4. **Application Management**
   - View application details alongside profile
   - Make informed decisions about acceptance/rejection

### Notes

- The applicant's subscription expiry is automatically checked and updated if expired
- All profile fields may be null if not provided by the user
- The endpoint uses `select_related` for optimized database queries
- Only returns profiles for applications to events organized by the requesting admin

### Related Endpoints

- `GET /api/applications/incoming/` - List all incoming applications
- `GET /api/events/<event_id>/applications/` - List applications for specific event
- `POST /api/applications/<application_id>/accept/` - Accept an application
- `POST /api/applications/<application_id>/reject/` - Reject an application
- `POST /api/applications/<application_id>/mark-reviewed/` - Mark application as reviewed

### Security Considerations

1. **Authorization Check**: Endpoint verifies that the requesting admin is the organizer of the event
2. **Admin-Only Access**: Uses `require_admin_only()` helper to ensure only admins can access
3. **Data Privacy**: Only admins who own the event can view applicant profiles
4. **Token Validation**: Checks for valid, non-blacklisted authentication tokens

### Implementation Details

```python
# Example usage in your application
async function viewApplicantProfile(applicationId, adminToken) {
  const response = await fetch(
    `http://localhost:8000/api/applications/${applicationId}/applicant-profile/`,
    {
      method: 'GET',
      headers: {
        'Authorization': `Bearer ${adminToken}`,
        'Content-Type': 'application/json'
      }
    }
  );
  
  if (response.ok) {
    const data = await response.json();
    return data.profile;
  } else {
    const error = await response.json();
    throw new Error(error.error);
  }
}
```

### Testing Examples

#### Python (requests library)
```python
import requests

application_id = 123
admin_token = "your_admin_access_token"

response = requests.get(
    f"http://localhost:8000/api/applications/{application_id}/applicant-profile/",
    headers={
        "Authorization": f"Bearer {admin_token}"
    }
)

if response.status_code == 200:
    profile = response.json()['profile']
    print(f"Applicant: {profile['name']}")
    print(f"Email: {profile['email']}")
    print(f"Expertise: {', '.join(profile['expertise_areas'])}")
else:
    print(f"Error: {response.json()['error']}")
```

#### JavaScript (Fetch API)
```javascript
const applicationId = 123;
const adminToken = "your_admin_access_token";

fetch(`http://localhost:8000/api/applications/${applicationId}/applicant-profile/`, {
  method: 'GET',
  headers: {
    'Authorization': `Bearer ${adminToken}`,
    'Content-Type': 'application/json'
  }
})
  .then(response => response.json())
  .then(data => {
    if (data.profile) {
      console.log('Applicant Name:', data.profile.name);
      console.log('Subscription:', data.profile.subscription_type);
      console.log('Application Status:', data.profile.application_info.status);
    } else {
      console.error('Error:', data.error);
    }
  })
  .catch(error => console.error('Request failed:', error));
```

---

**Version:** 1.0  
**Last Updated:** January 2026  
**Endpoint Category:** Application Management (Admin)