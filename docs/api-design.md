# ShiftPay Backend API Design

## Table of Contents

- [Overview](#overview)
- [Authentication](#authentication)
- [Data Models](#data-models)
- [API Endpoints](#api-endpoints)
  - [Shifts](#shifts-apishifts)
  - [Shift Templates](#shift-templates-apishifttemplates)
  - [Work Infos](#work-infos-apiworkinfos)
- [Error Handling](#error-handling)

---

## Overview

**Base URL:** `https://<host>`

ShiftPay Backend is a RESTful API for managing work shifts, shift templates, and workplace information. All data is scoped to the authenticated user and stored in Azure Cosmos DB.

### Key Characteristics

| Aspect | Details |
|--------|---------|
| Protocol | HTTPS |
| Data Format | JSON |
| Date/Time Format | ISO 8601 (e.g., `2024-01-15T09:00:00Z`) |
| ID Format | UUID/GUID |
| Authentication | JWT Bearer Token (Azure AD) |

---

## Authentication

All endpoints require authentication via the `[Authorize]` attribute.

- **Production**: Azure AD with JWT bearer tokens (managed by MSAL on the frontend).
- **Development**: Set `Authentication:UseFake=true` to bypass authentication with a test user (`test-user-id`).

The authenticated user's `NameIdentifier` claim is used as the Cosmos DB partition key for user-scoped data.

---

## Data Models

### ShiftDTO

Represents a work shift record.

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "workplace": "Coffee Shop",
  "payRate": 25.50,
  "startTime": "2024-01-15T09:00:00",
  "endTime": "2024-01-15T17:00:00",
  "unpaidBreaks": ["00:30:00", "00:15:00"]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | `guid?` | No | Auto-generated on create |
| `workplace` | `string` | Yes | Name of the workplace (trimmed) |
| `payRate` | `decimal` | Yes | Hourly pay rate (≥ 0) |
| `startTime` | `datetime` | Yes | Shift start time |
| `endTime` | `datetime` | Yes | Shift end time (must be after `startTime`) |
| `unpaidBreaks` | `TimeSpan[]` | Yes | List of unpaid break durations |

Shifts and templates also accept optional `workProfileId`, `payRateId`, `position`, `location`, and `payRateName` fields. These are saved snapshots; the server does not resolve a profile to recalculate `payRate`. Old records without these fields remain valid. Profile and rate IDs identify the chosen options even when their amounts match.

**Validation Rules:**
- `startTime` must be before `endTime`
- `payRate` cannot be negative
- Total `unpaidBreaks` cannot exceed shift duration

---

### ShiftTemplateDTO

Represents a reusable shift template.

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "templateName": "Morning Shift",
  "workplace": "Coffee Shop",
  "payRate": 25.50,
  "startTime": "2024-01-01T09:00:00",
  "endTime": "2024-01-01T17:00:00",
  "unpaidBreaks": ["00:30:00"]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | `guid?` | No | Auto-generated on create |
| `templateName` | `string` | Yes | Unique template name per user (trimmed) |
| `workplace` | `string` | Yes | Name of the workplace (trimmed) |
| `payRate` | `decimal` | Yes | Hourly pay rate (≥ 0) |
| `startTime` | `datetime` | Yes | Template start time |
| `endTime` | `datetime` | Yes | Template end time |
| `unpaidBreaks` | `TimeSpan[]` | Yes | List of unpaid break durations |

---

### WorkInfoDTO

Represents one workplace per user, containing separate position/location profiles and named hourly rates.

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "workplace": "Coffee Shop",
  "payRates": [25.50, 30.00],
  "profiles": [
    {
      "id": "52ec312c-8671-4e7b-bf60-ac62ec18c9e9",
      "position": "Barista",
      "location": "City branch",
      "payRates": [
        { "id": "8c52ae31-14a9-42c8-8d6b-9c366bfa5b06", "name": "Standard", "hourlyRate": 25.50 },
        { "id": "2d02f26a-d980-40e3-8bdb-c299804b1e33", "name": "Weekend", "hourlyRate": 30.00 }
      ]
    }
  ]
}
```

| Field | Type | Required | Description |
|-------|------|----------|-------------|
| `id` | `guid?` | No | Auto-generated on create |
| `workplace` | `string` | Yes | Unique workplace name per user (trimmed); immutable after creation |
| `payRates` | `decimal[]` | No | Legacy numeric projection of all profile rates; amounts are non-negative |
| `profiles` | `WorkProfile[]` | On PUT | Complete profiles; an explicit empty list removes all profiles |

Each profile has a stable string `id`, optional `position` and `location` (up to 200 characters each), and a `payRates` list. Each rate has a stable string `id`, a `name` (up to 100 characters), and a non-negative `hourlyRate`. IDs must be unique within their containing workplace/profile, non-empty, and at most 100 characters. New UI entries use UUIDs. Duplicate amounts with different IDs and names are valid. Profiles may have no rates.

Older documents and backups containing only numbers expose a generic profile with ID `legacy`, and rates with stable IDs `legacy-{amount}` and empty names. The UI shows the amount when no name exists. IDs stay stable when an amount is edited. Custom remembered amounts use the same rate ID format within their selected profile; if an edited rate already occupies that ID, the new amount receives a UUID and the name `Custom`. The application does not guess whether an old amount means Standard, Weekend, or Holiday.

When structured profiles are supplied, they are authoritative over the numeric projection. Old Cosmos documents stay readable, and explicitly removed profiles/rates do not reappear from a stale numeric list. All profiles remain embedded in the existing workplace document, preserving the Cosmos `/UserId` partition and unique `/Workplace` key without a container migration.

---

## API Endpoints

### Shifts (`/api/Shifts`)

Manage work shift records.

---

#### List Shifts

```http
GET /api/Shifts
```

Retrieves shifts with optional filtering.

**Query Parameters:**

| Parameter | Type | Description |
|-----------|------|-------------|
| `year` | `int?` | Filter by year |
| `month` | `int?` | Filter by month (1-12) |
| `day` | `int?` | Filter by day (1-31) |
| `startTime` | `datetime?` | Filter shifts starting on or after this time |
| `endTime` | `datetime?` | Filter shifts ending on or before this time |
| `id` | `guid[]?` | Filter by specific shift IDs (repeatable) |

**Example:**
```http
GET /api/Shifts?year=2024&month=1&day=15
GET /api/Shifts?startTime=2024-01-01T00:00:00&endTime=2024-01-31T23:59:59
GET /api/Shifts?id=abc123&id=def456
```

**Response:**
```http
200 OK
Content-Type: application/json

[
  { "id": "...", "workplace": "...", ... },
  { "id": "...", "workplace": "...", ... }
]
```

---

#### Get Shift

```http
GET /api/Shifts/{id}
```

Retrieves a single shift by ID.

**Response:**

| Status | Description |
|--------|-------------|
| `200 OK` | Returns `ShiftDTO` |
| `404 Not Found` | Shift not found |

---

#### Create Shift

```http
POST /api/Shifts
Content-Type: application/json

{
  "workplace": "Coffee Shop",
  "payRate": 25.50,
  "startTime": "2024-01-15T09:00:00",
  "endTime": "2024-01-15T17:00:00",
  "unpaidBreaks": ["00:30:00"]
}
```

**Response:**

| Status | Description |
|--------|-------------|
| `201 Created` | Returns created `ShiftDTO` with `Location` header |
| `400 Bad Request` | Validation error |
| `409 Conflict` | Database conflict |

---

#### Create Shifts (Batch)

```http
POST /api/Shifts/batch
Content-Type: application/json

[
  { "workplace": "Coffee Shop", "payRate": 25.50, ... },
  { "workplace": "Coffee Shop", "payRate": 25.50, ... }
]
```

Creates multiple shifts in a single request.

**Response:**

| Status | Description |
|--------|-------------|
| `201 Created` | Returns array of created `ShiftDTO[]` |
| `400 Bad Request` | Validation error in any shift |
| `409 Conflict` | Database conflict |

---

#### Update Shift

```http
PUT /api/Shifts/{id}
Content-Type: application/json

{
  "workplace": "Updated Workplace",
  "payRate": 30.00,
  "startTime": "2024-01-15T10:00:00",
  "endTime": "2024-01-15T18:00:00",
  "unpaidBreaks": ["00:45:00"]
}
```

**Notes:**
- If `id` is provided in the body, it must match the route `id`.
- Partition key changes (date changes) are handled internally via delete + insert.

**Response:**

| Status | Description |
|--------|-------------|
| `200 OK` | Returns updated `ShiftDTO` |
| `400 Bad Request` | Validation error or ID mismatch |
| `404 Not Found` | Shift not found |
| `409 Conflict` | Concurrency or database conflict |

---

#### Delete Shift

```http
DELETE /api/Shifts/{id}
```

Deletes a single shift by ID.

**Response:**

| Status | Description |
|--------|-------------|
| `204 No Content` | Successfully deleted |
| `404 Not Found` | Shift not found |

---

#### Delete Shifts (Bulk)

```http
DELETE /api/Shifts
```

Deletes multiple shifts matching the filter criteria.

**Query Parameters:** Same as [List Shifts](#list-shifts).

**Example:**
```http
DELETE /api/Shifts?year=2024&month=1
DELETE /api/Shifts?id=abc123&id=def456
```

**Response:**

| Status | Description |
|--------|-------------|
| `204 No Content` | Successfully deleted |
| `404 Not Found` | No matching shifts found |

---

### Shift Templates (`/api/ShiftTemplates`)

Manage reusable shift templates.

---

#### List Shift Templates

```http
GET /api/ShiftTemplates
```

Retrieves all shift templates for the authenticated user.

**Response:**
```http
200 OK
Content-Type: application/json

[
  { "id": "...", "templateName": "Morning Shift", ... },
  { "id": "...", "templateName": "Evening Shift", ... }
]
```

---

#### Get Shift Template

```http
GET /api/ShiftTemplates/{id}
```

**Response:**

| Status | Description |
|--------|-------------|
| `200 OK` | Returns `ShiftTemplateDTO` |
| `400 Bad Request` | ID is empty |
| `404 Not Found` | Template not found |

---

#### Create or Update Shift Template

```http
POST /api/ShiftTemplates
Content-Type: application/json

{
  "templateName": "Morning Shift",
  "workplace": "Coffee Shop",
  "payRate": 25.50,
  "startTime": "2024-01-01T09:00:00",
  "endTime": "2024-01-01T17:00:00",
  "unpaidBreaks": ["00:30:00"]
}
```

**Behavior:**
- If a template with the same `templateName` exists → **updates** the existing template.
- Otherwise → **creates** a new template.

**Response:**

| Status | Description |
|--------|-------------|
| `201 Created` | New template created |
| `200 OK` | Existing template updated |
| `400 Bad Request` | Validation error |
| `409 Conflict` | Database conflict |

---

#### Delete Shift Template

```http
DELETE /api/ShiftTemplates/{id}
```

**Response:**

| Status | Description |
|--------|-------------|
| `204 No Content` | Successfully deleted |
| `400 Bad Request` | ID is empty |
| `404 Not Found` | Template not found |

---

### Work Infos (`/api/WorkInfos`)

Manage workplace profiles and named rates. These endpoints remain scoped to the authenticated user.

---

#### List Work Infos

```http
GET /api/WorkInfos
```

Retrieves all work info records for the authenticated user.

**Response:**
```http
200 OK
Content-Type: application/json

[
  { "id": "...", "workplace": "Coffee Shop", "payRates": [20.50, 25.00], "profiles": [{ "id": "legacy", "payRates": [{ "id": "legacy-20.5", "name": "", "hourlyRate": 20.50 }, { "id": "legacy-25", "name": "", "hourlyRate": 25.00 }] }] },
  { "id": "...", "workplace": "Restaurant", "payRates": [], "profiles": [{ "id": "d3feb020-bdbf-436f-afd8-ee2e17f4343f", "position": "Supervisor", "location": "North", "payRates": [] }] }
]
```

---

#### Get Work Info

```http
GET /api/WorkInfos/{id}
```

**Response:**

| Status | Description |
|--------|-------------|
| `200 OK` | Returns `WorkInfoDTO` |
| `400 Bad Request` | ID is empty |
| `404 Not Found` | Work info not found |

---

#### Create or Merge Work Info

```http
POST /api/WorkInfos
Content-Type: application/json

{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "workplace": "Coffee Shop",
  "payRates": [25.50, 28.00]
}
```

**Behavior:**
- Finds an existing record by ID in the user's partition, then by workplace to support local backups whose generated ID differs from the account record.
- Creates a record when none exists. Otherwise merges profiles and rates by stable ID; existing metadata and amounts win on matching IDs. Use PUT for deliberate edits or removals.
- Legacy requests containing numeric `payRates` remain additive. Complete profile payloads use the `WorkInfoDTO` shape above.

**Response:**

| Status | Description |
|--------|-------------|
| `201 Created` | New work info created |
| `200 OK` | Existing work info updated |
| `400 Bad Request` | Validation error |
| `409 Conflict` | Database conflict |

---

#### Replace Work Info Profiles

```http
PUT /api/WorkInfos/{id}
Content-Type: application/json
```

Send the complete `WorkInfoDTO`, including `profiles`. Replaces the profile/rate lists, allowing edits and removals. Workplace names cannot change; create a new workplace instead. An optional body ID must match the route ID. Returns `200 OK`, `400 Bad Request` for invalid input, `404 Not Found` for an unavailable record, or `409 Conflict` for a database conflict.

Deploy the backend before the frontend. Existing numeric-rate clients remain compatible; new clients need the structured GET/POST and PUT support to preserve profile details. Work profile changes affect future selection only. Templates keep their saved amount until explicitly edited or replaced.

---

#### Delete Work Info or Pay Rate

```http
DELETE /api/WorkInfos/{id}
DELETE /api/WorkInfos/{id}?payRate=25.50
```

**Behavior:**
- Without `payRate` → Deletes the entire work info record.
- With `payRate` → Removes matching unnamed legacy/custom rate entries. Named rates with the same amount remain intact. Use PUT with rate IDs to edit or remove named choices.

**Response:**

| Status | Description |
|--------|-------------|
| `204 No Content` | Successfully deleted |
| `400 Bad Request` | ID is empty |
| `404 Not Found` | Work info not found (when removing pay rate) |

---

## Error Handling

All errors follow standard HTTP status codes with JSON error details.

### Common Error Responses

| Status Code | Description |
|-------------|-------------|
| `400 Bad Request` | Invalid input, validation failure, or malformed request |
| `401 Unauthorized` | Missing or invalid authentication token |
| `404 Not Found` | Requested resource does not exist |
| `409 Conflict` | Database conflict or concurrency error |
