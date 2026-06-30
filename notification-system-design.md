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


# Stage 2

## Database Choice

I recommend PostgreSQL as the persistent database because it is reliable, ACID-compliant, supports indexing, transactions, and scales well for notification systems.

## Database Schema

### Students Table

| Column | Type |
|--------|------|
| student_id | UUID (Primary Key) |
| name | VARCHAR(100) |
| email | VARCHAR(100) |

### Notifications Table

| Column | Type |
|--------|------|
| notification_id | UUID (Primary Key) |
| student_id | UUID (Foreign Key) |
| notification_type | ENUM('Placement','Result','Event') |
| message | TEXT |
| is_read | BOOLEAN |
| created_at | TIMESTAMP |

## Problems as Data Grows

- Slow queries
- Large table size
- Increased storage
- High read latency

## Solutions

- Create indexes on frequently queried columns.
- Use pagination instead of loading all notifications.
- Partition tables by date.
- Cache frequently accessed notifications using Redis.
- Archive old notifications.

## Sample SQL Queries

### Get unread notifications

SELECT *
FROM notifications
WHERE student_id = ?
AND is_read = false
ORDER BY created_at DESC;

### Mark notification as read

UPDATE notifications
SET is_read = true
WHERE notification_id = ?;

### Insert notification

INSERT INTO notifications
(student_id, notification_type, message, is_read, created_at)
VALUES
(?, ?, ?, false, NOW());

### Delete notification

DELETE FROM notifications
WHERE notification_id = ?;



# Stage 3

## Query Analysis

### Given Query

```sql
SELECT *
FROM notifications
WHERE studentID = 1042
AND isRead = false
ORDER BY createdAt ASC;
```

### Is the Query Accurate?

Yes. The query correctly fetches unread notifications for a specific student ordered by creation time.

### Why is it Slow?

- The table contains millions of records.
- Using `SELECT *` fetches unnecessary columns.
- Without a composite index, the database scans many rows.
- Sorting by `createdAt` becomes expensive if the matching rows are not already indexed.

### Improvements

- Select only the required columns instead of using `SELECT *`.
- Create a composite index on `(studentID, isRead, createdAt)`.
- Use pagination (`LIMIT` and `OFFSET`) when displaying notifications.

### Optimized Query

```sql
SELECT notification_id,
       notification_type,
       message,
       created_at
FROM notifications
WHERE studentID = 1042
AND isRead = false
ORDER BY createdAt ASC;
```

### Computation Cost

- Without indexes: O(N)
- With composite index: approximately O(log N) to locate matching rows, plus the cost of returning the result set.

### Should Every Column Have an Index?

No.

Adding indexes on every column is not recommended because:

- It increases storage usage.
- INSERT, UPDATE, and DELETE operations become slower.
- Many indexes are never used.
- Only columns frequently used in WHERE, JOIN, or ORDER BY clauses should be indexed.

### Query to Find Students Who Received Placement Notifications in the Last 7 Days

```sql
SELECT DISTINCT studentID
FROM notifications
WHERE notification_type = 'Placement'
AND createdAt >= NOW() - INTERVAL '7 days';
```
