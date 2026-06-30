# Stage 1

## Notification REST APIs

### 1. Get All Notifications

GET /api/notifications

Headers

Authorization: Bearer <token>

Response

{
  "notifications": []
}

---

### 2. Get Notification by ID

GET /api/notifications/{id}

Headers

Authorization: Bearer <token>

Response

{
  "id": "123",
  "type": "Placement",
  "message": "Amazon Hiring",
  "isRead": false
}

---

### 3. Create Notification

POST /api/notifications

Request

{
  "type": "Placement",
  "message": "Amazon Hiring"
}

Response

{
  "message": "Notification Created"
}

---

### 4. Mark Notification as Read

PATCH /api/notifications/{id}/read

Response

{
  "message": "Marked as Read"
}

---

### 5. Delete Notification

DELETE /api/notifications/{id}

Response

{
  "message": "Notification Deleted"
}

---

## Real-Time Notification Design

Use WebSockets to push notifications from the server to connected clients in real time. When a new notification is created, the server broadcasts it instantly to all relevant users without requiring the client to repeatedly poll the server.
