# DATAPOLIS API DOCUMENTATION

## Version 4.0

This document provides a detailed reference for the DATAPOLIS API, including all endpoints, request/response formats, and authentication methods.

---

### Authentication

All API requests must be authenticated using a JWT token. The token should be included in the `Authorization` header as a Bearer token.

`Authorization: Bearer <your_jwt_token>`

---

### PAE M11-DP (DATAPOLIS) Endpoints

#### `/api/pae/dp/analyze`

- **Method:** `POST`
- **Description:** Analyzes the precession graph for a given set of properties.
- **Request Body:**
```json
{
  "properties": [
    {
      "id": "prop1",
      "relations": [
        {"target_id": "prop2", "weight": 0.5}
      ]
    }
  ]
}
```
- **Response:**
```json
{
  "precession_scores": {
    "prop1": {
      "prop1": 0,
      "prop2": 0.5
    }
  }
}
```

---

### PAE M11-AG (ÁGORA) Endpoints

#### `/api/pae/ag/analyze`

- **Method:** `POST`
- **Description:** Analyzes the territorial graph to calculate precession indexes.
- **Request Body:**
```json
{
  "nodes": [
    {"id": "zone1", "attributes": {"type": "residential"}}
  ],
  "edges": [
    {"source": "zone1", "target": "zone2", "weight": 0.8}
  ]
}
```
- **Response:**
```json
{
  "precession_indexes": {
    "zone1": 0.65
  }
}
```
