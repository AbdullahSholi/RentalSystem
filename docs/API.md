# API Documentation

This document provides comprehensive information about all REST API endpoints available in the Car Rental System.

## Base URL
```
http://localhost:9090
```

## Authentication

Most endpoints require JWT authentication. Include the token in the Authorization header:
```
Authorization: Bearer <your-jwt-token>
```

## API Endpoints Overview

### Authentication Endpoints
- `POST /auth/signup` - Register new user
- `POST /auth/login` - User login

### Vehicle Management
- `GET /api/car-rental-system/vehicles` - Get all vehicles
- `GET /api/car-rental-system/vehicle` - Get vehicle by ID
- `POST /api/car-rental-system/vehicle` - Create new vehicle
- `PUT /api/car-rental-system/vehicle` - Update vehicle
- `DELETE /api/car-rental-system/vehicle` - Delete vehicle

### Payment Processing
- `GET /api/v1/car-rental-system/payments` - Get all payments
- `POST /api/v1/car-rental-system/process` - Process payment

### Reservation Management
- `GET /api/car-rental-system/reservations` - Get all reservations
- `GET /api/car-rental-system/reservation` - Get reservation by ID
- `POST /api/car-rental-system/reservation` - Create reservation
- `PUT /api/car-rental-system/reservation` - Update reservation
- `DELETE /api/car-rental-system/reservation` - Delete reservation

### Customer Management
- `GET /api/car-rental-system/customers` - Get all customers
- `GET /api/car-rental-system/customer` - Get customer by ID
- `POST /api/car-rental-system/customer` - Create customer
- `PUT /api/car-rental-system/customer` - Update customer
- `DELETE /api/car-rental-system/customer` - Delete customer

---

## Detailed API Reference

## Authentication API

### Register User
```http
POST /auth/signup
Content-Type: application/json

{
  "fullName": "John Doe",
  "email": "john.doe@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "customerId": 1,
  "customerName": "John Doe",
  "email": "john.doe@example.com",
  "role": {
    "id": 1,
    "name": "USER"
  }
}
```

### Login User
```http
POST /auth/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "securePassword123"
}
```

**Response:**
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600000
}
```

## Vehicle Management API

### Get All Vehicles
```http
GET /api/car-rental-system/vehicles
Authorization: Bearer <token>
```

**Required Role:** ADMIN, SUPER_ADMIN

**Response:**
```json
[
  {
    "vehicleId": 1,
    "vehicleName": "Toyota Camry",
    "manufacturer": "Toyota",
    "model": "Camry 2024",
    "dtype": "CAR",
    "engineCapacity": 2.5,
    "fuelType": "Petrol",
    "hasAirConditioning": true,
    "vehicleStatus": "Available",
    "latitude": 30.0,
    "longitude": 0.0
  }
]
```

### Get Vehicle by ID
```http
GET /api/car-rental-system/vehicle?vehicleId=1
Authorization: Bearer <token>
```

**Required Role:** Authenticated user

### Create Vehicle
```http
POST /api/car-rental-system/vehicle
Authorization: Bearer <token>
Content-Type: application/json

{
  "vehicleName": "Honda Civic",
  "manufacturer": "Honda",
  "model": "Civic 2024",
  "dtype": "CAR",
  "engineCapacity": 2.0,
  "fuelType": "Petrol",
  "hasAirConditioning": true,
  "vehicleStatus": "Available",
  "latitude": 31.5,
  "longitude": 35.2
}
```

**Required Role:** ADMIN, SUPER_ADMIN

## Payment Processing API

### Get All Payments
```http
GET /api/v1/car-rental-system/payments
Authorization: Bearer <token>
```

**Response:**
```json
[
  {
    "paymentId": 1,
    "totalAmount": 3000.0,
    "paymentMethod": "onArrival",
    "status": "COMPLETED",
    "paymentDate": "2024-10-09T12:29:31",
    "cardType": null
  }
]
```

### Process Payment
```http
POST /api/v1/car-rental-system/process
Authorization: Bearer <token>
Content-Type: application/json

{
  "totalAmount": 2000.0,
  "paymentMethod": "card",
  "cardType": "Stripe",
  "status": "PENDING"
}
```

**Required Role:** ADMIN, CUSTOMER

**Response:**
```json
{
  "message": "Payment processed successfully"
}
```

**Supported Payment Methods:**
- `onArrival` - Payment on arrival
- `card` - Card payment (requires cardType)

**Supported Card Types:**
- `Stripe` - Stripe payment processing
- `PayPal` - PayPal payment processing

## Reservation Management API

### Get All Reservations
```http
GET /api/car-rental-system/reservations
Authorization: Bearer <token>
```

**Required Role:** ADMIN, SUPER_ADMIN

### Create Reservation
```http
POST /api/car-rental-system/reservation
Authorization: Bearer <token>
Content-Type: application/json

{
  "reservationStartDate": "2024-12-01",
  "reservationEndDate": "2024-12-07",
  "additionalServices": "GPS Navigation",
  "status": "PENDING",
  "customerId": 1
}
```

**Required Role:** Authenticated user

## Customer Management API

### Get All Customers
```http
GET /api/car-rental-system/customers
Authorization: Bearer <token>
```

**Required Role:** ADMIN, SUPER_ADMIN

### Get Customer by ID
```http
GET /api/car-rental-system/customer?customerId=1
Authorization: Bearer <token>
```

**Required Role:** Authenticated user

## Branch Management API

### Get All Branches
```http
GET /api/v1/car-rental-system/branches
Authorization: Bearer <token>
```

**Required Role:** ADMIN, SUPER_ADMIN

### Get Branch by ID
```http
GET /api/v1/car-rental-system/branch/1
Authorization: Bearer <token>
```

**Required Role:** Authenticated user

## Employee Management API

### Get All Employees
```http
GET /api/v1/car-rental-system/employees
Authorization: Bearer <token>
```

### Get Employee by ID
```http
GET /api/v1/car-rental-system/employee/1
Authorization: Bearer <token>
```

## Department Management API

### Get All Departments
```http
GET /api/v1/car-rental-system/departments
Authorization: Bearer <token>
```

## Invoice Management API

### Get All Invoices
```http
GET /api/v1/car-rental-system/invoices
Authorization: Bearer <token>
```

### Get Invoice by ID
```http
GET /api/v1/car-rental-system/invoice/1
Authorization: Bearer <token>
```

## Notification Management API

### Get All Notifications
```http
GET /api/car-rental-system/notifications
Authorization: Bearer <token>
```

### Get Notifications by Customer ID
```http
GET /api/car-rental-system/notifications/1
Authorization: Bearer <token>
```

## Service Management API

### Get All Services
```http
GET /api/car-rental-system/services
Authorization: Bearer <token>
```

### Create Service
```http
POST /api/car-rental-system/service
Authorization: Bearer <token>
Content-Type: application/json

{
  "serviceName": "GPS Navigation",
  "serviceCost": 50.0
}
```

## Admin Management API

### Get All Admins
```http
GET /api/v1/car-rental-system/admins
Authorization: Bearer <token>
```

**Required Role:** SUPER_ADMIN

## Error Responses

### 400 Bad Request
```json
{
  "message": "Invalid request data"
}
```

### 401 Unauthorized
```json
{
  "message": "Authentication required"
}
```

### 403 Forbidden
```json
{
  "message": "Insufficient permissions"
}
```

### 404 Not Found
```json
{
  "message": "Resource not found"
}
```

## Rate Limiting

Currently, no rate limiting is implemented. Consider implementing rate limiting for production use.

## Swagger Documentation

Interactive API documentation is available at:
```
http://localhost:9090/swagger-ui.html
```

This provides a complete interface for testing all endpoints with proper authentication.
