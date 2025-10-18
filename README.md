# OAuth2 Resource Server

A Spring Boot-based OAuth2 Resource Server that protects APIs with role-based access control. It validates JWT tokens from the authorization server and provides secure endpoints for managing products.

## Project Overview

This project implements an OAuth2 Resource Server using Spring Security. It protects the Products API with OAuth2 token validation and enforces role-based access control. Users with the USER role can only view products, while ADMIN users can create, update, and delete products.

## Features

- **OAuth2 Token Validation**: Validates JWT tokens from the authorization server
- **Role-Based Access Control**: Different permissions for USER and ADMIN roles
- **Protected API Endpoints**: All product endpoints require valid OAuth2 tokens
- **JWT Token Verification**: Automatic verification of token signatures using JWKS endpoint
- **In-Memory Database**: H2 database for development and testing
- **CORS Support**: Allows cross-origin requests from client applications

## Tech Stack

- **Framework**: Spring Boot 3.x
- **Security**: Spring Security 6.x, OAuth2 Resource Server
- **Database**: H2 (In-Memory Database)
- **ORM**: Spring Data JPA
- **Authentication**: JWT (JSON Web Tokens)
- **Build Tool**: Maven
- **Language**: Java 17+
- **Utilities**: Lombok

## Database Schema

The application uses a single `products` table to store product information:

```sql
CREATE TABLE products (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    price DOUBLE NOT NULL
);
```

### Table Columns

| Column | Type | Description |
|--------|------|-------------|
| `id` | BIGINT | Primary key, auto-incremented |
| `name` | VARCHAR(255) | Product name |
| `price` | DOUBLE | Product price |

## Installation

### Prerequisites

- Java 17 or higher
- Maven 3.8+
- Git
- Auth Server running on http://127.0.0.1:8081

### Step 1: Clone the Repository

```bash
git clone <repository-url>
cd resource-server
```

### Step 2: Configure application.yml

The `application.yml` is already configured. Verify the following settings:

```yaml
server:
  port: 8082

spring:
  security:
    oauth2:
      resourceserver:
        jwt:
          issuer-uri: http://127.0.0.1:8081

  datasource:
    url: jdbc:h2:mem:testdb
    driver-class-name: org.h2.Driver
    username: sa
    password:

  jpa:
    hibernate:
      ddl-auto: update
    show-sql: true

  h2:
    console:
      enabled: true
      path: /h2-console
```

### Step 3: Build the Project

```bash
mvn clean install
```

### Step 4: Run the Application

```bash
mvn spring-boot:run
```

Or:

```bash
java -jar target/resource-server-0.0.1-SNAPSHOT.jar
```

The server will start on `http://localhost:8082`

### Step 5: Verify Installation

1. **Check H2 Console**: http://localhost:8082/h2-console
    - JDBC URL: `jdbc:h2:mem:testdb`
    - Username: `sa`
    - Password: (leave empty)

2. **Test API Endpoint** (requires valid token):
   ```bash
   curl -H "Authorization: Bearer <access_token>" \
        http://localhost:8082/api/products
   ```

## Project Structure

```
src/main/java/com/example/resourceserver/
├── ResourceServerApplication.java
├── config/
│   └── SecurityConfig.java
├── controller/
│   └── ProductController.java
├── service/
│   └── ProductService.java
├── repository/
│   └── ProductRepository.java
├── entity/
│   └── Product.java
└── dto/
    ├── ProductRequest.java
    └── ProductResponse.java

src/main/resources/
├── application.yml
└── data.sql (for sample data)
```

## API Endpoints

All endpoints require a valid OAuth2 access token in the `Authorization` header.

### Products Endpoints

#### Get All Products

```
GET /api/products
```

**Required Role**: USER or ADMIN

**Response**:
```json
[
  {
    "id": 1,
    "name": "Product 1",
    "price": 29.99
  }
]
```

#### Get Product by ID

```
GET /api/products/{id}
```

**Required Role**: USER or ADMIN

**Response**:
```json
{
  "id": 1,
  "name": "Product 1",
  "price": 29.99
}
```

#### Create Product

```
POST /api/products
```

**Required Role**: ADMIN only

**Request Body**:
```json
{
  "name": "New Product",
  "price": 49.99
}
```

**Response**:
```json
{
  "id": 2,
  "name": "New Product",
  "price": 49.99
}
```

#### Update Product

```
PUT /api/products/{id}
```

**Required Role**: ADMIN only

**Request Body**:
```json
{
  "name": "Updated Product",
  "price": 59.99
}
```

**Response**:
```json
{
  "id": 1,
  "name": "Updated Product",
  "price": 59.99
}
```

#### Delete Product

```
DELETE /api/products/{id}
```

**Required Role**: ADMIN only

**Response**: 204 No Content

## Role-Based Access Control

The resource server enforces role-based authorization:

| Endpoint | USER | ADMIN |
|----------|------|-------|
| GET /api/products | ✅ | ✅ |
| GET /api/products/{id} | ✅ | ✅ |
| POST /api/products | ❌ | ✅ |
| PUT /api/products/{id} | ❌ | ✅ |
| DELETE /api/products/{id} | ❌ | ✅ |

## Security Configuration

Security is configured in `SecurityConfig.java`:

- **JWT Validation**: All tokens are validated against the authorization server's JWKS endpoint
- **Role Extraction**: User roles are extracted from the JWT token claims
- **Method-Level Security**: `@PreAuthorize` annotations enforce role-based access on controller methods
- **CORS**: Enabled for client applications at http://localhost:8083




## Authentication Flow

1. Client app sends HTTP request with Bearer token in Authorization header
2. Resource server validates JWT signature using auth server's public keys
3. Resource server extracts user roles from token claims
4. Controller method checks user role with `@PreAuthorize` annotation
5. If authorized, request proceeds; if not, returns 403 Forbidden

## Error Responses

### 401 Unauthorized
```json
{
  "error": "unauthorized",
  "error_description": "The access token is invalid"
}
```

### 403 Forbidden
```json
{
  "error": "access_denied",
  "error_description": "User does not have the required role"
}
```

### 404 Not Found
```json
{
  "error": "not_found",
  "error_description": "Product not found"
}
```

## Configuration

### JWT Validation

Token validation is automatic. The resource server fetches public keys from:

```
http://127.0.0.1:8081/.well-known/jwks.json
```

### Role Mapping

Roles are extracted from the JWT `roles` claim and mapped with `ROLE_` prefix:
- JWT claim `"roles": ["USER"]` → Spring authority `ROLE_USER`
- JWT claim `"roles": ["ADMIN"]` → Spring authority `ROLE_ADMIN`



## Related Projects

- **Authorization Server** (Port 8081): Issues OAuth2 tokens
- **Client Application** (Port 8083): OAuth2 client web app

