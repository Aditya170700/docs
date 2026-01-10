# API Changes Documentation

**Date:** December 26, 2025  
**Version:** Latest  
**Target:** Mobile Developer Implementation Guide

---

## 📋 Table of Contents

1. [New APIs](#new-apis)
2. [Modified APIs - Response Changes](#modified-apis---response-changes)
3. [Modified APIs - Request Body Changes](#modified-apis---request-body-changes)
4. [API Details](#api-details)
5. [Notification System](#notification-system)
6. [Point System Details](#point-system-details)
7. [Redemption System Details](#redemption-system-details)

---

## 🆕 New APIs

### 1. Master Gift APIs

#### 1.1 Get Master Gift List

**Endpoint:** `GET /api/mobile/gift/list`  
**Authentication:** Not Required  
**Description:** Get paginated list of active master gifts

**Query Parameters:**

-   `limit` (optional, integer, default: 10) - Number of items per page

**Response:**

```json
{
    "status": "success",
    "data": {
        "list": [
            {
                "id": 1,
                "image": "https://example.com/image.jpg",
                "name": "Gift Name",
                "point": 1000
            }
        ],
        "limit": 10,
        "page": 1,
        "total": 50
    }
}
```

#### 1.2 Get Master Gift Detail

**Endpoint:** `GET /api/mobile/gift/detail/{id}`  
**Authentication:** Not Required (but customer_point will be null if not logged in)  
**Description:** Get detail of a specific master gift

**Path Parameters:**

-   `id` (required, integer) - Master gift ID

**Response:**

```json
{
    "status": "success",
    "data": {
        "id": 1,
        "name": "Gift Name",
        "image": "https://example.com/image.jpg",
        "point": 1000,
        "term_n_condition": "Terms and conditions text...",
        "customer_point": 5000
    }
}
```

**Note:** `customer_point` will be `null` if user is not logged in.

---

### 2. Configuration API

#### 2.1 Get Configuration by Feature

**Endpoint:** `GET /api/mobile/configuration?feature={feature_name}`  
**Authentication:** Not Required  
**Description:** Get configuration data by feature name (case insensitive)

**Query Parameters:**

-   `feature` (required, string) - Feature name to search for (case insensitive)

**Response:**

```json
{
    "point": {
        "status": true,
        "type": "active",
        "text": "undermaintenance"
    }
}
```

**Response Format:**

```json
{
  "{feature_name}": {
    "status": boolean,
    "type": "maintenance|coming_soon|active",
    "text": string|null
  }
}
```

**Error Responses:**

-   `400` - Parameter feature diperlukan
-   `404` - Configuration tidak ditemukan

**Notes:**

-   Search is case insensitive (e.g., `point`, `POINT`, `Point` will return the same result)
-   Response key uses the actual feature name from database (not the query parameter)
-   `text` field can be `null` if not set

**Example Usage:**

```bash
# Get configuration for "point" feature
GET /api/mobile/configuration?feature=point

# Case insensitive - all return same result
GET /api/mobile/configuration?feature=POINT
GET /api/mobile/configuration?feature=Point
GET /api/mobile/configuration?feature=poInT
```

**Response Example:**

```json
{
    "point": {
        "status": true,
        "type": "maintenance",
        "text": "Sistem sedang dalam maintenance"
    }
}
```

---

### 3. Redemption APIs

#### 3.1 Get Redemption List

**Endpoint:** `GET /api/mobile/redemption/list`  
**Authentication:** Required (`auth:api`)  
**Description:** Get paginated list of user's redemptions with `created_at` field

**Query Parameters:**

-   `limit` (optional, integer, default: 10) - Number of items per page

**Response:**

```json
{
    "status": "success",
    "data": {
        "list": [
            {
                "id": 1,
                "redemption_code": "RED-ABC12345",
                "status": "processing",
                "created_at": "2025-12-25T14:30:00.000000Z",
                "items": [
                    {
                        "id": 1,
                        "name": "Gift Name",
                        "image": "https://example.com/image.jpg",
                        "point": 1000,
                        "quantity": 1
                    }
                ]
            }
        ],
        "limit": 10,
        "page": 1,
        "total": 5
    }
}
```

**Note:** Items are no longer grouped by date. Each item includes `created_at` field directly.

#### 3.2 Get Redemption Detail

**Endpoint:** `GET /api/mobile/redemption/detail/{id}`  
**Authentication:** Required (`auth:api`)  
**Description:** Get detail of a specific redemption

**Path Parameters:**

-   `id` (required, integer) - Redemption ID

**Response:**

```json
{
    "status": "success",
    "data": {
        "id": 1,
        "redemption_code": "RED-ABC12345",
        "redemption_date": "2025-12-25 14:30:00",
        "total_points_spent": 1000,
        "delivery_note": "Catatan pengiriman...",
        "is_received": false,
        "status": "processing",
        "customer_address_id": 5,
        "shipping_address": {
            "id": 5,
            "name": "John Doe",
            "phone": "081234567890",
            "address": "Jl. Example No. 123",
            "province": "Jakarta",
            "city": "Jakarta Selatan",
            "district": "Kebayoran Baru",
            "subdistrict": "Senayan",
            "postal_code": "12190",
            "is_default": true,
            "fullDetail": "John Doe, 081234567890, Jl. Example No. 123, Jakarta Selatan, Jakarta, Kebayoran Baru, Senayan, 12190"
        },
        "waybill_number": "AWB123456789",
        "gifts": [
            {
                "id": 1,
                "name": "Gift Name",
                "image": "https://example.com/image.jpg",
                "point": 1000,
                "quantity": 1
            }
        ],
        "histories": [
            {
                "id": 1,
                "status": "pending",
                "comment": "Penukaran sedang diproses",
                "created_at": "2025-12-25 14:30:00"
            },
            {
                "id": 2,
                "status": "processing",
                "comment": "Barang sedang dikirim",
                "created_at": "2025-12-26 10:00:00"
            }
        ]
    }
}
```

**Changes:**

-   `shipping_address` is now an object containing full address details from `customer_addresses` table
-   Added `customer_address_id` field
-   `shipping_address` is `null` if no address is set

#### 3.3 Redeem Points

**Endpoint:** `POST /api/mobile/redemption`  
**Authentication:** Required (`auth:api`)  
**Description:** Redeem points for a master gift. Points are NOT deducted immediately. Deduction happens when redemption status becomes `done`.

**Request Body:**

```json
{
    "master_gift_id": 1,
    "store_id": 5
}
```

**Request Body Fields:**

-   `master_gift_id` (required, integer) - Master gift ID to redeem
-   `store_id` (required, integer) - Store ID where the redemption is associated. Must exist in `stores` table.

**Note:** `shipping_address` is no longer required. The system automatically uses the customer's default address (`is_default = true`).

**Response:**

```json
{
    "status": "success",
    "message": "Penukaran berhasil dilakukan",
    "data": {
        "redemption_code": "RED-ABC12345",
        "redemption_id": 1
    }
}
```

**Error Responses:**

-   `400` - Gift tidak ditemukan atau tidak aktif
-   `400` - Stock gift tidak tersedia
-   `400` - Point tidak cukup. Point Anda: {current}, Point yang dibutuhkan: {required}
-   `400` - Validation error if `store_id` is missing or invalid

**Business Rules:**

-   Points are NOT deducted when redemption is created
-   Points are deducted when redemption status changes to `done` (either via admin panel or customer marking as received)
-   System automatically uses customer's default address (`is_default = true`)
-   Push notification and email are sent immediately after redemption is created

#### 3.4 Mark Redemption as Received

**Endpoint:** `POST /api/mobile/redemption/complete/{id}`  
**Authentication:** Required (`auth:api`)  
**Description:** Mark redemption as received by customer. This will deduct points if not already deducted.

**Path Parameters:**

-   `id` (required, integer) - Redemption ID

**Response:**

```json
{
    "status": "success",
    "message": "Penukaran berhasil diselesaikan",
    "data": {
        "redemption_id": 1,
        "is_received": true,
        "status": "done"
    }
}
```

**Error Responses:**

-   `400` - Redemption sudah diterima sebelumnya
-   `400` - Redemption masih dalam status pending. Tidak dapat diselesaikan.
-   `404` - Redemption tidak ditemukan

**Business Rules:**

-   Cannot complete if status is `pending`
-   Can complete if status is `processing` (will update to `done` and create history)
-   Can complete if status is `done` (will only update `is_received`, no history created)
-   Points are deducted when status changes to `done` (only once, with duplicate check)

---

### 4. Point APIs

#### 4.1 Get My Points

**Endpoint:** `GET /api/mobile/user/point/my`  
**Authentication:** Required (`auth:api`)  
**Description:** Get current point balance and expiry date

**Response:**

```json
{
    "status": "success",
    "message": "Point berhasil diambil",
    "data": {
        "point": 5000,
        "valid_until": "2026-12-31T23:59:59.000000Z"
    }
}
```

**Note:** `valid_until` can be `null` if no point history exists.

#### 4.2 Get Point History

**Endpoint:** `GET /api/mobile/user/point/history`  
**Authentication:** Required (`auth:api`)  
**Description:** Get paginated list of point history transactions

**Query Parameters:**

-   `limit` (optional, integer, default: 10) - Number of items per page

**Response:**

```json
{
    "status": "success",
    "message": "History point berhasil diambil",
    "data": {
        "list": [
            {
                "id": 1,
                "title": "Point Masuk",
                "description": "Pembelian: TV",
                "code": "INV-20251225-001",
                "created_at": "2025-12-25T19:30:00.000000Z",
                "point": "+50"
            },
            {
                "id": 2,
                "title": "Redeem Reward",
                "description": "Penukaran: TV",
                "code": "RED-ABC12345",
                "created_at": "2025-12-26T10:00:00.000000Z",
                "point": "-1000"
            }
        ],
        "limit": 10,
        "page": 1,
        "total": 20
    }
}
```

**Point Format:**

-   `+{amount}` for incoming points (from purchases)
-   `-{amount}` for outgoing points (from redemptions or expiry)

**Title Values:**

-   `"Point Masuk"` - For points from orders (reference = "order")
-   `"Redeem Reward"` - For points from redemptions (reference = "redem")

**Code Values:**

-   Order number (e.g., "INV-20251225-001") for purchase transactions
-   Redemption code (e.g., "RED-ABC12345") for redemption transactions
-   `null` for system-generated transactions (e.g., expiry deductions)

---

### 5. Notification APIs

#### 5.1 List Notifications

**Endpoint:** `GET /api/mobile/user/notification/list`

**Headers:**

```
Authorization: Bearer {token}
Accept: application/json
```

**Query Parameters:**

-   `limit` (optional, default: 10): Jumlah item per page
-   `type` (optional): Filter by type (`point_received`, `redemption_status`, `point_expired`)

**Response:**

```json
{
    "success": true,
    "data": {
        "list": [
            {
                "id": 1,
                "type": "point_received",
                "title": "Poin Rewards Berhasil Masuk",
                "message": "Selamat! Anda mendapatkan 50 poin dari pembelian Pembelian: TV",
                "data": {
                    "point_amount": 50,
                    "order_number": "INV-123",
                    "product_name": "TV",
                    "store_name": "Store Name",
                    "transaction_date": "25 Dec 2025 19:30",
                    "current_balance": 150
                },
                "is_read": false,
                "read_at": null,
                "created_at": "2025-12-25T19:30:00.000000Z"
            },
            {
                "id": 2,
                "type": "redemption_status",
                "title": "Permintaan Penukaran Hadiah Berhasil Dikirim",
                "message": "Permintaan penukaran Hadiah TV Anda berhasil dan akan segera diproses",
                "data": {
                    "redemption_id": 1,
                    "redemption_code": "RED-ABC12345",
                    "gift_name": "TV"
                },
                "is_read": false,
                "read_at": null,
                "created_at": "2025-12-25T20:00:00.000000Z"
            },
            {
                "id": 3,
                "type": "point_expired",
                "title": "Poin Rewards Anda Telah Kedaluwarsa",
                "message": "Sebagian poin rewards Anda (500 poin) telah kedaluwarsa pada 31 Des 2025. Sisa poin Anda: 1000 poin.",
                "data": {
                    "expired_points": 500,
                    "expiry_date": "31 Des 2025",
                    "remaining_points": 1000
                },
                "is_read": false,
                "read_at": null,
                "created_at": "2025-12-31T03:00:00.000000Z"
            }
        ],
        "limit": 10,
        "page": 1,
        "total": 3,
        "unread_count": 3
    },
    "message": null
}
```

**Notification Types:**

1. **`point_received`** - Sent when customer receives points from completed order
2. **`redemption_status`** - Sent when redemption status changes (pending → processing → done)
3. **`point_expired`** - Sent when points expire (50% deduction or full wipe out)

**Note:** For `point_expired` type, the message wording changes based on remaining points:

-   If `remaining_points > 0`: "Sebagian poin rewards Anda..."
-   If `remaining_points == 0`: "Semua poin rewards Anda..."

#### 5.2 Unread Count

**Endpoint:** `GET /api/mobile/user/notification/unread-count`

**Response:**

```json
{
    "success": true,
    "data": {
        "unread_count": 5
    },
    "message": null
}
```

#### 5.3 Mark as Read

**Endpoint:** `POST /api/mobile/user/notification/read/{id}`

**Path Parameters:**

-   `id` (required, integer) - Notification ID

**Response:**

```json
{
    "success": true,
    "data": {
        "notification": {
            "id": 1,
            "type": "point_received",
            "title": "Poin Rewards Berhasil Masuk",
            "message": "...",
            "is_read": true,
            "read_at": "2025-12-25T19:35:00.000000Z"
        }
    },
    "message": "Notifikasi berhasil ditandai sebagai dibaca"
}
```

#### 5.4 Mark All as Read

**Endpoint:** `POST /api/mobile/user/notification/read-all`

**Response:**

```json
{
    "success": true,
    "data": {},
    "message": "Semua notifikasi berhasil ditandai sebagai dibaca"
}
```

#### 5.5 Delete Notification

**Endpoint:** `DELETE /api/mobile/user/notification/delete/{id}`

**Path Parameters:**

-   `id` (required, integer) - Notification ID

**Response:**

```json
{
    "success": true,
    "data": {},
    "message": "Notifikasi berhasil dihapus"
}
```

---

## 🔄 Modified APIs - Response Changes

### 1. Product List API

**Endpoint:** `GET /api/mobile/product/list`  
**Changes:** Added `is_promotion` and `terms_n_condition` fields

**New Response Fields:**

```json
{
    "data": {
        "list": [
            {
                "id": 1,
                // ... existing fields ...
                "is_promotion": true,
                "terms_n_condition": "Terms and conditions..."
            }
        ]
    }
}
```

### 2. Product Detail API

**Endpoint:** `GET /api/mobile/product/detail/{id}`  
**Changes:** Added `is_promotion` and `terms_n_condition` fields

**New Response Fields:**

```json
{
    "data": {
        "id": 1,
        // ... existing fields ...
        "is_promotion": true,
        "terms_n_condition": "Terms and conditions..."
    }
}
```

### 3. Cart Detail API

**Endpoint:** `GET /api/mobile/cart/detail`  
**Changes:**

1. Added `is_promotion` field to each item in cart
2. Added `point` calculation object

**New Response Structure:**

```json
{
    "data": {
        "total_item": 5,
        "items": [
            {
                "store_id": 1,
                "store_name": "Store Name",
                "items": [
                    {
                        "id": 1,
                        // ... existing fields ...
                        "is_promotion": true
                    }
                ]
            }
        ],
        // ... existing fields ...
        "point": {
            "point_default": 100,
            "point_promo": 50,
            "total_point": 150
        }
    }
}
```

**Point Calculation Logic:**

-   `point_default`: `floor((grand_total / point_conversions.nominal) * point_conversions.point)`
-   `point_promo`: `floor((grand_total_barang_promo / point_conversions.nominal) * point_conversions.point)`
-   `total_point`: `point_default + point_promo`

### 4. Store List API

**Endpoint:** `GET /api/mobile/store/list`  
**Changes:** Added pagination support

**New Query Parameters:**

-   `limit` (optional, integer, default: 10) - Number of items per page

**New Response Structure:**

```json
{
    "data": {
        "list": [
            {
                "id": 1
                // ... existing store fields ...
            }
        ],
        "limit": 10,
        "page": 1,
        "total": 50
    }
}
```

---

## 🔄 Modified APIs - Request Body Changes

### 1. Login API

**Endpoint:** `POST /api/mobile/auth/login`  
**Changes:** Added optional `fcm_token` field

**New Request Body:**

```json
{
    "email": "user@example.com",
    "password": "password123",
    "fcm_token": "fcm_token_here"
}
```

**Note:**

-   `fcm_token` is optional (nullable)
-   If provided, it will be saved to customer's profile
-   If not provided, no changes will be made
-   FCM token is used for push notifications

---

## 📝 API Details

### Authentication

Most APIs use JWT authentication. Include the token in the Authorization header:

```
Authorization: Bearer {token}
```

### Base URL

```
https://your-api-domain.com/api/mobile
```

### Response Format

All APIs follow this standard response format:

**Success Response:**

```json
{
  "status": "success",
  "message": "Optional message",
  "data": { ... }
}
```

**Error Response:**

```json
{
    "status": "error",
    "message": "Error message",
    "data": []
}
```

### Status Codes

-   `200` - Success
-   `400` - Bad Request / Validation Error
-   `401` - Unauthorized (Not logged in)
-   `404` - Not Found
-   `500` - Internal Server Error

---

## 🔔 Notification System

### Overview

The notification system provides three types of notifications:

1. **Push Notifications** (Firebase Cloud Messaging)
2. **Email Notifications**
3. **In-App Notifications** (stored in database)

All notifications are sent asynchronously via Laravel Queues for best performance.

### Notification Types

#### 1. Point Received (`point_received`)

**Trigger:** When an order is marked as done (completed)

**Push Notification:**

-   Title: "Poin Rewards Berhasil Masuk"
-   Message: "Selamat! Anda mendapatkan {point_amount} poin dari pembelian {store_name}: {product_name}"

**Email:** Sent with order details, point amount, and current balance

**Data Structure:**

```json
{
    "point_amount": 50,
    "order_number": "INV-123",
    "product_name": "TV",
    "store_name": "Store Name",
    "transaction_date": "25 Dec 2025 19:30",
    "current_balance": 150
}
```

#### 2. Redemption Status (`redemption_status`)

**Trigger:** When redemption status changes

**Sub-types:**

**a. Initial Redemption (Pending)**

-   Push Title: "Permintaan Penukaran Hadiah Berhasil Dikirim"
-   Push Message: "Permintaan penukaran Hadiah {namaHadiah} Anda berhasil dan akan segera diproses"
-   Email: Sent with redemption details

**b. Processing Status**

-   Push Title: "Hadiah Dalam Proses Pengiriman"
-   Push Message: "Permintaan penukaran Hadiah {namaHadiah} Anda telah diproses dan sedang dalam proses pengiriman"
-   Email: Sent with processing status and waybill number (if available)

**c. Done Status**

-   Push Title: "Penukaran Hadiah Selesai"
-   Push Message: "Permintaan penukaran Hadiah {namaHadiah} Anda telah diselesaikan"
-   Email: Sent with completion details, redemption code, dates, and points spent

**Data Structure:**

```json
{
    "redemption_id": 1,
    "redemption_code": "RED-ABC12345",
    "gift_name": "TV",
    "status": "pending|processing|done"
}
```

#### 3. Point Expired (`point_expired`)

**Trigger:** When points expire (via scheduled command `points:check-expiry`)

**Push Notification:**

-   Title: "Poin Rewards Anda Telah Kedaluwarsa"
-   Message (if remaining > 0): "Sebagian poin rewards Anda ({expiredPoints} poin) telah kedaluwarsa pada {tanggal}. Sisa poin Anda: {remainingPoints} poin."
-   Message (if remaining == 0): "Semua poin rewards Anda ({expiredPoints} poin) telah kedaluwarsa pada {tanggal}."

**Email:** Sent with expiry details, expired points, and remaining points (if any)

**Data Structure:**

```json
{
    "expired_points": 500,
    "expiry_date": "31 Des 2025",
    "remaining_points": 1000
}
```

### FCM Token Management

-   FCM token should be sent during login via `fcm_token` field
-   Token is stored in `customers.fcm_token` column
-   Token is optional but recommended for push notifications
-   If token is not provided, push notifications will be skipped (email and in-app notifications still work)

### Queue Configuration

**Development:**

-   `QUEUE_CONNECTION=sync` - Notifications are sent synchronously (no need to run `php artisan queue:work`)

**Production:**

-   `QUEUE_CONNECTION=database` or `redis` - Notifications are queued and processed asynchronously
-   Run `php artisan queue:work` to process queued notifications

---

## 💰 Point System Details

### Point Earning

**When:** Points are awarded when an order is marked as **done** (completed), not during checkout.

**Calculation:**

-   Points are calculated based on cart totals
-   Point conversion settings are stored in `point_conversions` table
-   Points are separated into:
    -   `point_default`: Points from non-promotional items
    -   `point_promo`: Points from promotional items
    -   `total_point`: Sum of both

**Formula:**

-   `point_default = floor((grand_total / point_conversions.nominal) * point_conversions.point)`
-   `point_promo = floor((grand_total_barang_promo / point_conversions.nominal) * point_conversions.point)`
-   `total_point = point_default + point_promo`

**Point History:**

-   Type: `TYPE_IN` (incoming)
-   Reference: Order model
-   `valid_until`: Set to `now + 1 year`
-   `consecutive_expiry_count`: Reset to `0` (important for expiry logic)

### Point Deduction

**When:** Points are deducted when:

1. Redemption status changes to `done` (via admin panel or customer marking as received)
2. Points expire (via scheduled command)

**Redemption Deduction:**

-   Type: `TYPE_OUT` (outgoing)
-   Reference: Redemption model
-   `consecutive_expiry_count`: Reset to `0` (important for expiry logic)
-   Deduction happens only once (duplicate check implemented)

**Expiry Deduction:**

-   Type: `TYPE_OUT` (outgoing)
-   Reference: `SYSTEM_SCHEDULER`
-   Logic based on `consecutive_expiry_count`:
    -   **First expiry (`consecutive_expiry_count == 0`)**: Deduct 50% of balance, set `consecutive_expiry_count = 1`
    -   **Second expiry (`consecutive_expiry_count >= 1`)**: Wipe out balance to 0, set `consecutive_expiry_count = 0`
-   `valid_until`: Set to `now + 1 year` for new record

### Point Expiry System

**Scheduled Command:** `points:check-expiry` (runs daily at 3:00 AM)

**Logic:**

1. Process users in chunks
2. Get latest `PointHistory` record for each user
3. Check if `valid_until < now()`
4. If expired:
    - **First expiry:** Deduct 50% of balance
    - **Second expiry:** Wipe out balance to 0
5. Create new `PointHistory` record with deduction
6. Send notification (push + email)

**Important Notes:**

-   Any `PURCHASE` or `REDEEM` transaction **MUST** reset `consecutive_expiry_count` to `0`
-   This ensures new point earnings give a fresh start to the expiry counter
-   Expiry notifications use conditional wording based on remaining points

### Point History Format

**Incoming Points (from purchases):**

-   Title: "Point Masuk"
-   Point: `+{amount}`
-   Code: Order number (e.g., "INV-20251225-001")

**Outgoing Points (from redemptions):**

-   Title: "Redeem Reward"
-   Point: `-{amount}`
-   Code: Redemption code (e.g., "RED-ABC12345")

**Outgoing Points (from expiry):**

-   Title: "Redeem Reward" (or custom title)
-   Point: `-{amount}`
-   Code: `null` (system-generated)

---

## 🎁 Redemption System Details

### Redemption Flow

1. **Customer creates redemption** (`POST /api/mobile/redemption`)

    - Points are **NOT** deducted yet
    - `store_id` is required in request body and will be saved to redemption
    - System uses customer's default address automatically
    - Push notification and email sent immediately
    - Status: `pending`

2. **Admin processes redemption** (Admin Panel)

    - Status changes to `processing`
    - Waybill number can be added
    - Push notification and email sent
    - Points are **NOT** deducted yet

3. **Redemption completed** (Admin Panel or Customer)
    - Status changes to `done`
    - Points are deducted (only once, with duplicate check)
    - Push notification and email sent
    - Customer can mark as received

### Redemption Statuses

-   **`pending`**: Initial status when redemption is created
-   **`processing`**: Admin has processed the redemption and is shipping
-   **`done`**: Redemption is completed (points deducted at this stage)

### Address Handling

-   Redemption uses `customer_address_id` (not `shipping_address` string)
-   System automatically uses customer's default address (`is_default = true`)
-   If no default address exists, `customer_address_id` will be `null`
-   `shipping_address` in API response is a formatted object from `customer_addresses` table

### Store Association

-   Redemption is associated with a store via `store_id` field
-   `store_id` is required when creating redemption (`POST /api/mobile/redemption`)
-   `store_id` must exist in `stores` table
-   `store_id` is nullable in database but required in API request for redemption creation

### Point Deduction Logic

**Important:** Points are deducted only when redemption status becomes `done`, not when redemption is created.

**Deduction happens in:**

1. `RedemptionController::markAsReceived()` (Mobile API) - when customer marks as received
2. `RedemtionController::update()` (Admin Panel) - when admin changes status to `done`

**Duplicate Prevention:**

-   System checks if point deduction already exists for this redemption
-   Uses `PointHistory` with `reference_type = Redemption::class` and `reference_id = redemption->id`
-   If already deducted, skip deduction (prevents duplicate deductions)

### Redemption Notifications

**Initial Redemption (Pending):**

-   Sent immediately after redemption is created
-   Title: "Permintaan Penukaran Hadiah Berhasil Dikirim"
-   Includes redemption code and gift name

**Processing Status:**

-   Sent when admin changes status to `processing`
-   Title: "Hadiah Dalam Proses Pengiriman"
-   Includes waybill number (if available)

**Done Status:**

-   Sent when admin changes status to `done` or customer marks as received
-   Title: "Penukaran Hadiah Selesai"
-   Includes completion dates and points spent

---

## 🔑 Key Points for Implementation

### 1. Promotion System

-   Products can be in active promotions
-   Check `is_promotion` field to determine if product is in promotion
-   `terms_n_condition` contains promotion terms (null if not in promotion)
-   Only one active promotion can exist at a time

### 2. Point System

-   Points are calculated based on cart totals
-   Point conversion settings are stored in `point_conversions` table
-   Points are separated into `point_default` and `point_promo`
-   Total points = default + promo points
-   Points are awarded when order is **done**, not during checkout
-   Points expire after 1 year (with 50% deduction on first expiry, full wipe out on second)

### 3. Redemption System

-   Users can redeem points for gifts
-   Redemption statuses: `pending`, `processing`, `done`
-   Points are deducted when status becomes `done` (not when redemption is created)
-   System automatically uses customer's default address
-   Users can mark redemption as received when status is `processing` or `done`
-   Redemption list includes `created_at` field (no longer grouped by date)

### 4. FCM Token

-   FCM token should be sent during login
-   Token is optional but recommended for push notifications
-   Token is stored in customer profile
-   If token is not provided, push notifications will be skipped

### 5. Notification System

-   All notifications are sent via queues (asynchronous)
-   Three notification types: `point_received`, `redemption_status`, `point_expired`
-   Notifications are stored in database for in-app display
-   Push notifications require FCM token
-   Email notifications are sent to customer's email
-   In development (`QUEUE_CONNECTION=sync`), notifications are sent synchronously

### 6. Point Expiry

-   Points expire after 1 year from earning date
-   First expiry: 50% deduction
-   Second consecutive expiry: Full wipe out (balance = 0)
-   Expiry counter resets when new points are earned
-   Expiry notifications use conditional wording based on remaining points

---

**Last Updated:** January 10, 2026
