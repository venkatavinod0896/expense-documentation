# Expense Backend v3 — API Documentation

## Overview

**Base URL:** `http://<host>:8080`
**Default Port:** `8080` (override with `APP_PORT` environment variable)
**Content-Type:** `application/json`

The Expense Backend v3 is a Node.js/Express REST API that manages financial transactions stored in a MySQL database. It supports creating, retrieving, and deleting expense transactions.

---

## Database Schema

```sql
CREATE TABLE transactions (
    id          INT AUTO_INCREMENT PRIMARY KEY,
    amount      INT NOT NULL,
    description VARCHAR(255) NOT NULL,
    category    VARCHAR(50) NOT NULL
);
```

### Transaction Object

| Field       | Type    | Description                          |
|-------------|---------|--------------------------------------|
| id          | integer | Auto-generated unique identifier     |
| amount      | integer | Transaction amount (positive integer, max 1,000,000) |
| description | string  | Short description (max 255 characters) |
| category    | string  | One of the valid categories (see below) |

### Valid Categories

`Food` | `Travel` | `Entertainment` | `Shopping` | `Health` | `Utilities` | `Other`

---

## Environment Variables

| Variable      | Default        | Description               |
|---------------|----------------|---------------------------|
| `APP_PORT`    | `8080`         | Port the server listens on |
| `DB_HOST`     | *(empty)*      | MySQL host address         |
| `DB_USER`     | `expense`      | MySQL username             |
| `DB_PWD`      | `ExpenseApp@1` | MySQL password             |
| `DB_DATABASE` | `transactions` | MySQL database name        |

---

## Endpoints

### Health Check

#### `GET /health`

Returns the health status of the service.

**Response `200 OK`**
```json
{ "status": "ok" }
```

---

### Transactions

#### `GET /transaction`

Retrieves all transactions, ordered by most recent first.

**Response `200 OK`**
```json
{
  "result": [
    {
      "id": 3,
      "amount": 500,
      "description": "Groceries",
      "category": "Food"
    },
    {
      "id": 2,
      "amount": 1200,
      "description": "Flight ticket",
      "category": "Travel"
    }
  ]
}
```

**Response `500 Internal Server Error`**
```json
{
  "message": "could not retrieve transactions",
  "error": "<db error message>"
}
```

---

#### `GET /transaction/:id`

Retrieves a single transaction by its ID.

**Path Parameters**

| Parameter | Type    | Description           |
|-----------|---------|-----------------------|
| `id`      | integer | Positive integer ID   |

**Response `200 OK`**
```json
{
  "id": 3,
  "amount": 500,
  "description": "Groceries",
  "category": "Food"
}
```

**Response `400 Bad Request`** — invalid id format
```json
{ "message": "invalid id" }
```

**Response `404 Not Found`** — id does not exist
```json
{ "message": "transaction <id> not found" }
```

**Response `500 Internal Server Error`**
```json
{
  "message": "could not retrieve transaction",
  "error": "<db error message>"
}
```

---

#### `POST /transaction`

Creates a new transaction.

**Request Body**

```json
{
  "amount": 750,
  "description": "Monthly electricity bill",
  "category": "Utilities"
}
```

| Field         | Type    | Required | Constraints                                       |
|---------------|---------|----------|---------------------------------------------------|
| `amount`      | integer | Yes      | Positive integer, max 1,000,000                  |
| `description` | string  | Yes      | Non-empty string, max 255 characters             |
| `category`    | string  | Yes      | One of the valid categories                       |

**Response `201 Created`**
```json
{ "message": "transaction added successfully" }
```

**Response `400 Bad Request`** — validation failure
```json
{
  "message": "validation failed",
  "errors": [
    "amount must be a positive integer",
    "category must be one of: Food, Travel, Entertainment, Shopping, Health, Utilities, Other"
  ]
}
```

**Response `500 Internal Server Error`**
```json
{
  "message": "could not add transaction",
  "error": "<db error message>"
}
```

---

#### `DELETE /transaction`

Deletes **all** transactions from the database.

**Response `200 OK`**
```json
{ "message": "all transactions deleted" }
```

**Response `500 Internal Server Error`**
```json
{
  "message": "could not delete transactions",
  "error": "<db error message>"
}
```

---

#### `DELETE /transaction/:id`

Deletes a single transaction by its ID.

**Path Parameters**

| Parameter | Type    | Description          |
|-----------|---------|----------------------|
| `id`      | integer | Positive integer ID  |

**Response `200 OK`**
```json
{ "message": "transaction <id> deleted" }
```

**Response `400 Bad Request`** — invalid id format
```json
{ "message": "invalid id" }
```

**Response `500 Internal Server Error`**
```json
{
  "message": "could not delete transaction",
  "error": "<db error message>"
}
```

---

## Validation Rules

| Field         | Rule                                                                 |
|---------------|----------------------------------------------------------------------|
| `amount`      | Required; must be a positive integer; cannot exceed 1,000,000       |
| `description` | Required; cannot be blank; max 255 characters                        |
| `category`    | Required; must exactly match one of the seven valid category values  |

---

## Quick Reference

| Method   | Path              | Description                    |
|----------|-------------------|--------------------------------|
| GET      | `/health`         | Service health check           |
| GET      | `/transaction`    | List all transactions          |
| POST     | `/transaction`    | Create a new transaction       |
| DELETE   | `/transaction`    | Delete all transactions        |
| GET      | `/transaction/:id`| Get a transaction by ID        |
| DELETE   | `/transaction/:id`| Delete a transaction by ID     |

---

## Dependencies

| Package       | Version   | Purpose                        |
|---------------|-----------|--------------------------------|
| express       | ^4.18.2   | HTTP server framework          |
| body-parser   | ^1.20.2   | JSON request body parsing      |
| cors          | ^2.8.5    | Cross-origin resource sharing  |
| mysql2        | ^3.6.0    | MySQL database driver (pooled) |
