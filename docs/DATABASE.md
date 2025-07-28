# Database Schema Documentation

This document describes the database schema, entity relationships, and data model for the Car Rental System.

## 🗄️ Database Overview

- **Database Engine**: MySQL 8.0+
- **ORM**: Hibernate/JPA
- **Schema Management**: Hibernate DDL Auto-update
- **Character Set**: UTF8MB4

## 📊 Entity Relationship Diagram

```
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│   Customer  │────│ Reservation │────│   Rental    │
│             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
       │                   │                   │
       │                   │                   │
┌─────────────┐    ┌─────────────┐    ┌─────────────┐
│Notification │    │   Vehicle   │    │   Payment   │
│             │    │             │    │             │
└─────────────┘    └─────────────┘    └─────────────┘
                           │                   │
                           │                   │
                   ┌─────────────┐    ┌─────────────┐
                   │ParkingStall │    │   Invoice   │
                   │             │    │             │
                   └─────────────┘    └─────────────┘
```

## 📋 Table Definitions

### Core Business Tables

#### `customer`
Primary entity for system users.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| customer_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique customer identifier |
| customer_name | VARCHAR(255) | | Customer full name |
| customer_address | VARCHAR(255) | | Customer address |
| phone_number | VARCHAR(255) | | Contact phone number |
| driver_license | VARCHAR(255) | | Driver license number |
| email | VARCHAR(255) | UNIQUE, NOT NULL | Login email |
| password | VARCHAR(255) | NOT NULL | Encrypted password |
| role_id | INT | FOREIGN KEY | Reference to roles table |
| created_at | DATETIME(6) | | Account creation timestamp |
| updated_at | DATETIME(6) | | Last update timestamp |

#### `vehicle`
Base table for all vehicle types using Single Table Inheritance.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| vehicle_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique vehicle identifier |
| dtype | VARCHAR(31) | NOT NULL | Discriminator (CAR, BUS, TRUCK, VAN, MOTOR) |
| vehicle_name | VARCHAR(255) | | Vehicle display name |
| manufacturer | VARCHAR(255) | | Vehicle manufacturer |
| model | VARCHAR(255) | | Vehicle model |
| plate_number | VARCHAR(255) | | License plate number |
| engine_capacity | DOUBLE | NOT NULL | Engine size in liters |
| fuel_type | VARCHAR(255) | | Fuel type (Petrol, Diesel, Electric) |
| has_air_conditioning | BIT(1) | NOT NULL | AC availability |
| vehicle_status | VARCHAR(255) | | Current status (Available, Rented, Maintenance) |
| latitude | DOUBLE | NOT NULL | GPS latitude |
| longitude | DOUBLE | NOT NULL | GPS longitude |
| rental_id | INT | FOREIGN KEY | Current rental (if any) |
| reservation_id | INT | FOREIGN KEY | Current reservation (if any) |

**Vehicle Type Specific Columns:**
- **Car**: `num_of_doors` (INT)
- **Bus**: `num_of_seats` (INT), `has_wi_fi` (BIT)
- **Truck**: `cargo_capacity` (INT), `num_of_axles` (INT), `trailer_type` (VARCHAR)
- **Van**: `cargo_capacity` (INT)
- **Motor**: `has_side_car` (BIT), `motor_type` (VARCHAR)

#### `reservation`
Customer vehicle reservations.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| reservation_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique reservation identifier |
| customer_id | INT | FOREIGN KEY, NOT NULL | Customer making reservation |
| reservation_start_date | DATE | | Reservation start date |
| reservation_end_date | DATE | | Reservation end date |
| additional_services | VARCHAR(255) | | Additional services requested |
| status | VARCHAR(255) | | Reservation status (PENDING, CONFIRMED, CANCELLED) |

#### `rental`
Active vehicle rentals.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| rental_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique rental identifier |
| driver_license | VARCHAR(255) | | Driver license for rental |
| rental_start_date | DATE | | Actual rental start date |
| rental_end_date | DATE | | Actual rental end date |
| reservation_id | INT | FOREIGN KEY, UNIQUE | Associated reservation |
| payment_id | INT | FOREIGN KEY, UNIQUE | Associated payment |
| rental_prices_id | INT | FOREIGN KEY, UNIQUE | Pricing information |

#### `payment`
Payment transactions.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| payment_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique payment identifier |
| total_amount | DOUBLE | NOT NULL | Payment amount |
| payment_method | VARCHAR(255) | | Payment method (card, onArrival) |
| card_type | VARCHAR(255) | | Card type (Stripe, PayPal) |
| status | ENUM | | Payment status (PENDING, COMPLETED, FAILED) |
| payment_date | DATETIME(6) | | Payment processing date |

#### `rental_prices`
Pricing structure for rentals.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| rental_id | INT | PRIMARY KEY | Links to rental table |
| price_per_day | DOUBLE | NOT NULL | Daily rental rate |
| price_per_week | DOUBLE | NOT NULL | Weekly rental rate |
| price_per_month | DOUBLE | NOT NULL | Monthly rental rate |
| price_per_year | DOUBLE | NOT NULL | Yearly rental rate |

### Organizational Tables

#### `branch`
Rental company branches.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| branch_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique branch identifier |
| branch_name | VARCHAR(255) | | Branch name |
| branch_location | VARCHAR(255) | | Branch address |
| contact_details | VARCHAR(255) | | Branch contact information |
| payment_id | INT | FOREIGN KEY, NOT NULL | Associated payment |
| reservation_id | INT | FOREIGN KEY, NOT NULL | Associated reservation |

#### `parking_stall`
Parking spaces within branches.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| stall_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique stall identifier |
| branch_id | INT | FOREIGN KEY, NOT NULL | Branch containing stall |
| vehicle_id | INT | FOREIGN KEY, UNIQUE | Vehicle parked (if any) |

#### `department`
Company departments.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| department_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique department identifier |
| department_name | VARCHAR(255) | | Department name |
| department_position | VARCHAR(255) | | Department position/role |

#### `employee`
Company employees.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| employee_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique employee identifier |
| employee_name | VARCHAR(255) | | Employee full name |
| phone_number | VARCHAR(255) | | Employee contact number |
| employment_type | VARCHAR(255) | | Employment type (Full-time, Part-time) |
| department_id | INT | FOREIGN KEY, NOT NULL | Employee department |

#### `admin`
System administrators.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| admin_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique admin identifier |
| admin_name | VARCHAR(255) | | Administrator name |
| phone_number | VARCHAR(255) | | Admin contact number |

### Support Tables

#### `invoice`
Generated invoices.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| invoice_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique invoice identifier |
| total_amount | DOUBLE | NOT NULL | Invoice total amount |
| print_date | DATETIME(6) | | Invoice generation date |
| employee_id | INT | FOREIGN KEY, NOT NULL | Employee who generated invoice |
| payment_id | INT | FOREIGN KEY, UNIQUE | Associated payment |

#### `notification`
Customer notifications.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| notification_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique notification identifier |
| customer_id | INT | FOREIGN KEY, NOT NULL | Target customer |
| message | VARCHAR(255) | | Notification message |
| is_read | BIT(1) | NOT NULL | Read status |
| timestamp | TIME(6) | | Notification time |

#### `_service`
Additional services offered.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| service_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique service identifier |
| service_name | VARCHAR(255) | | Service name |
| service_cost | DOUBLE | NOT NULL | Service cost |

#### `reservation_service`
Many-to-many relationship between reservations and services.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| reservation_id | INT | FOREIGN KEY, PRIMARY KEY | Reservation identifier |
| service_id | INT | FOREIGN KEY, PRIMARY KEY | Service identifier |

#### `vehicle_check`
Vehicle maintenance and inspection records.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| check_id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique check identifier |
| vehicle_id | INT | FOREIGN KEY, NOT NULL | Vehicle being checked |
| check_date | DATETIME(6) | | Check date |
| status | VARCHAR(255) | | Check status/result |

### Security Tables

#### `roles`
System roles for authorization.

| Column | Type | Constraints | Description |
|--------|------|-------------|-------------|
| id | INT | PRIMARY KEY, AUTO_INCREMENT | Unique role identifier |
| name | ENUM | UNIQUE, NOT NULL | Role name (USER, ADMIN, SUPER_ADMIN) |
| description | VARCHAR(255) | | Role description |
| created_at | DATETIME(6) | | Role creation date |
| updated_at | DATETIME(6) | | Last update date |

## 🔗 Key Relationships

### One-to-One Relationships
- `rental` ↔ `reservation` (One rental per reservation)
- `rental` ↔ `payment` (One payment per rental)
- `rental` ↔ `rental_prices` (One pricing structure per rental)
- `invoice` ↔ `payment` (One invoice per payment)

### One-to-Many Relationships
- `customer` → `reservation` (Customer can have multiple reservations)
- `customer` → `notification` (Customer can receive multiple notifications)
- `branch` → `parking_stall` (Branch has multiple parking stalls)
- `department` → `employee` (Department has multiple employees)
- `employee` → `invoice` (Employee can generate multiple invoices)
- `rental` → `vehicle` (Rental can include multiple vehicles)

### Many-to-Many Relationships
- `reservation` ↔ `_service` (via `reservation_service`)

## 📝 Data Constraints

### Enum Values
- **Payment Status**: `PENDING`, `COMPLETED`, `FAILED`
- **Role Names**: `USER`, `ADMIN`, `SUPER_ADMIN`
- **Vehicle Types**: `CAR`, `BUS`, `TRUCK`, `VAN`, `MOTOR`

### Unique Constraints
- `customer.email` - Ensures unique login credentials
- `roles.name` - Prevents duplicate role names
- `parking_stall.vehicle_id` - One vehicle per parking stall
- Various one-to-one relationship constraints

### Foreign Key Constraints
All foreign key relationships are enforced at the database level with appropriate cascade rules.

## 🔧 Database Configuration

### Connection Settings
```properties
spring.datasource.url=jdbc:mysql://localhost:3306/rental
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
spring.jpa.hibernate.ddl-auto=update
spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQLDialect
```

### Performance Optimizations
- **Indexes**: Automatically created for primary keys, foreign keys, and unique constraints
- **Connection Pooling**: HikariCP (Spring Boot default)
- **Query Optimization**: JPA query optimization enabled

## 📊 Sample Data

The system includes sample data for:
- **100 Admin records** (admin_id 1-100)
- **2 Customer records** (including super admin)
- **1 Branch record** (Asira Branch in Nablus)
- **6 Vehicle records** (Cars, Bus, Truck)
- **1 Reservation record**
- **6 Payment records**
- **3 Role records** (USER, ADMIN, SUPER_ADMIN)

## 🔄 Migration Strategy

### Schema Updates
- **Development**: `hibernate.ddl-auto=update`
- **Production**: Use Flyway or Liquibase for controlled migrations

### Data Migration
- Export/Import scripts for data transfer
- Backup strategies for production data
- Version control for schema changes

This database schema provides a robust foundation for the car rental system with proper normalization, referential integrity, and scalability considerations.
