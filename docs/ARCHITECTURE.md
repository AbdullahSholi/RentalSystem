# System Architecture

This document describes the architecture, design patterns, and structural organization of the Car Rental System.

## 🏗️ Overall Architecture

The system follows a **layered architecture** pattern with clear separation of concerns:

```
┌─────────────────────────────────────────┐
│              Presentation Layer          │
│         (REST Controllers)              │
├─────────────────────────────────────────┤
│              Business Layer             │
│            (Services)                   │
├─────────────────────────────────────────┤
│              Data Access Layer          │
│         (Repositories)                  │
├─────────────────────────────────────────┤
│              Data Layer                 │
│            (MySQL Database)             │
└─────────────────────────────────────────┘
```

## 📁 Project Structure

```
src/main/java/com/rental/rental/
├── controller/              # REST API Controllers
│   ├── AuthenticationController.java
│   ├── PaymentController.java
│   ├── VehicleController.java
│   ├── ReservationController.java
│   ├── CustomerController.java
│   ├── AdminController.java
│   ├── EmployeeController.java
│   ├── BranchController.java
│   ├── DepartmentController.java
│   ├── InvoiceController.java
│   ├── NotificationController.java
│   └── ServiceController.java
├── service/                 # Business Logic Layer
│   ├── PaymentService/      # Payment processing services
│   │   ├── PaymentService.java
│   │   ├── PaymentFactoryImplementation/
│   │   └── PaymentStrategyImplementation/
│   ├── VehicleService.java
│   ├── ReservationService.java
│   ├── CustomerService.java
│   ├── AdminService.java
│   ├── EmployeeService.java
│   ├── BranchService.java
│   ├── DepartmentService.java
│   ├── InvoiceService.java
│   ├── NotificationService.java
│   └── ServiceService.java
├── model/                   # JPA Entities
│   ├── Customer.java
│   ├── Vehicle.java         # Base class
│   ├── Car.java            # Vehicle subclass
│   ├── Bus.java            # Vehicle subclass
│   ├── Truck.java          # Vehicle subclass
│   ├── Van.java            # Vehicle subclass
│   ├── Motor.java          # Vehicle subclass
│   ├── Reservation.java
│   ├── Payment.java
│   ├── Rental.java
│   ├── RentalPrices.java
│   ├── Branch.java
│   ├── ParkingStall.java
│   ├── Employee.java
│   ├── Department.java
│   ├── Admin.java
│   ├── Invoice.java
│   ├── Notification.java
│   ├── _Service.java
│   └── Role.java
├── dto/                     # Data Transfer Objects
│   ├── CustomerDTO.java
│   ├── VehicleDTO.java
│   ├── ReservationDTO.java
│   ├── PaymentDTO.java
│   ├── RentalDTO.java
│   ├── LoginUserDto.java
│   ├── RegisterUserDto.java
│   └── LoginResponse.java
├── repository/              # Data Access Layer
│   ├── CustomerRepository.java
│   ├── VehicleRepository.java
│   ├── ReservationRepository.java
│   ├── PaymentRepository.java
│   ├── RentalRepository.java
│   ├── BranchRepository.java
│   ├── EmployeeRepository.java
│   ├── DepartmentRepository.java
│   ├── AdminRepository.java
│   ├── InvoiceRepository.java
│   ├── NotificationRepository.java
│   ├── ServiceRepository.java
│   ├── UserRepository.java
│   └── RoleRepository.java
├── configs/                 # Configuration Classes
│   ├── SecurityConfiguration.java
│   ├── ApplicationConfiguration.java
│   ├── JwtAuthenticationFilter.java
│   └── config/
│       ├── PayPalConfig.java
│       └── StripeConfig.java
└── services/               # Authentication Services
    ├── AuthenticationService.java
    └── JwtService.java
```

## 🎨 Design Patterns

### 1. Strategy Pattern
**Location**: `service/PaymentService/PaymentStrategyImplementation/`

Used for payment processing to allow different payment methods:

```java
public interface PaymentStrategy {
    boolean pay(PaymentDTO paymentDTO);
}

// Implementations:
- CardPaymentContext
- OnArrivalPaymentContext
- StripePayment
- PayPalPayment
```

**Benefits**:
- Easy to add new payment methods
- Encapsulates payment algorithms
- Runtime strategy selection

### 2. Factory Pattern
**Location**: `service/PaymentService/PaymentFactoryImplementation/`

```java
@Service
public class PaymentFactoryService {
    public PaymentStrategy getPaymentService(PaymentDTO paymentDTO) {
        if("onArrival".equalsIgnoreCase(paymentDTO.getPaymentMethod())) {
            return onArrivalPaymentContext;
        } else if("Card".equalsIgnoreCase(paymentDTO.getPaymentMethod())) {
            return cardPaymentContext;
        }
        // ...
    }
}
```

**Benefits**:
- Centralized object creation
- Loose coupling
- Easy to extend with new payment types

### 3. Template Method Pattern
**Location**: `service/PaymentService/PaymentStrategyImplementation/CardPaymentTemplate.java`

```java
public abstract class CardPaymentTemplate implements PaymentStrategy {
    public final boolean pay(PaymentDTO paymentDTO) {
        validateCard(paymentDTO);
        boolean isProcessed = processPaymentWithProvider(paymentDTO);
        confirmPayment(paymentDTO);
        return isProcessed;
    }
    
    protected abstract boolean processPaymentWithProvider(PaymentDTO paymentDTO);
}
```

**Benefits**:
- Defines algorithm skeleton
- Allows customization of specific steps
- Code reuse for common operations

### 4. Repository Pattern
**Location**: `repository/` package

```java
public interface CustomerRepository extends JpaRepository<Customer, Integer> {
    // Custom query methods
}
```

**Benefits**:
- Abstracts data access
- Testable business logic
- Consistent data access interface

### 5. DTO Pattern
**Location**: `dto/` package

Separates internal models from API contracts:

```java
@Data
public class CustomerDTO {
    private int customerId;
    private String customerName;
    // ... other fields
}
```

**Benefits**:
- API stability
- Data validation
- Security (field filtering)

## 🔧 Architectural Components

### 1. Controllers Layer
- **Responsibility**: Handle HTTP requests/responses
- **Features**: 
  - Request validation
  - Response formatting
  - Security annotations
  - API documentation

### 2. Service Layer
- **Responsibility**: Business logic implementation
- **Features**:
  - Transaction management
  - Business rule enforcement
  - DTO/Entity conversion
  - Cross-cutting concerns

### 3. Repository Layer
- **Responsibility**: Data persistence
- **Features**:
  - CRUD operations
  - Custom queries
  - Transaction support
  - Entity management

### 4. Configuration Layer
- **Responsibility**: Application configuration
- **Components**:
  - Security configuration
  - JWT configuration
  - Payment provider configuration
  - Database configuration

## 🚗 Domain Model

### Core Entities

#### Vehicle Hierarchy
```
Vehicle (Base Class)
├── Car
├── Bus
├── Truck
├── Van
└── Motor
```

**Inheritance Strategy**: Single Table Inheritance with discriminator column `dtype`

#### Key Relationships
- **Customer** ↔ **Reservation** (One-to-Many)
- **Customer** ↔ **Notification** (One-to-Many)
- **Reservation** ↔ **Rental** (One-to-One)
- **Payment** ↔ **Rental** (One-to-One)
- **Vehicle** ↔ **Rental** (Many-to-One)
- **Branch** ↔ **ParkingStall** (One-to-Many)
- **Employee** ↔ **Department** (Many-to-One)

## 🔐 Security Architecture

### Authentication Flow
```
Client Request → JWT Filter → Authentication Provider → UserDetailsService → Database
```

### Authorization Levels
- **PUBLIC**: `/auth/**` endpoints
- **USER**: Basic customer operations
- **ADMIN**: Management operations
- **SUPER_ADMIN**: Full system access

## 💳 Payment Architecture

### Payment Processing Flow
```
Payment Request → PaymentFactoryService → PaymentStrategy → External Provider → Response
```

### Supported Providers
- **Stripe**: Credit card processing
- **PayPal**: Alternative payment method
- **On Arrival**: Deferred payment

## 📊 Data Flow

### Typical Request Flow
```
1. HTTP Request → Controller
2. Controller → Service (Business Logic)
3. Service → Repository (Data Access)
4. Repository → Database
5. Database → Repository → Service → Controller
6. Controller → HTTP Response
```

### Cross-Cutting Concerns
- **Logging**: Implemented via Spring Boot defaults
- **Validation**: Bean Validation (JSR-303)
- **Exception Handling**: Global exception handlers
- **Security**: Method-level security annotations

## 🧪 Testing Architecture

### Test Structure
```
src/test/java/com/rental/rental/
├── controller/          # Controller unit tests
├── service/            # Service unit tests
├── repository/         # Repository unit tests
└── RentalApplicationTests.java  # Test suite
```

### Testing Patterns
- **Unit Tests**: Mockito for dependencies
- **Integration Tests**: TestContainers for database
- **Test Suites**: JUnit 5 test suites

## 🔄 Extension Points

### Adding New Vehicle Types
1. Create new entity extending `Vehicle`
2. Add discriminator value
3. Update `VehicleService` factory logic
4. Add specific fields in DTO

### Adding New Payment Methods
1. Implement `PaymentStrategy` interface
2. Update `PaymentFactoryService`
3. Add configuration if needed
4. Update API documentation

### Adding New Roles
1. Update `RoleEnum`
2. Add database migration
3. Update security configuration
4. Add role-specific endpoints

## 📈 Performance Considerations

### Database Optimization
- **Indexing**: Primary keys, foreign keys, unique constraints
- **Connection Pooling**: HikariCP (Spring Boot default)
- **Lazy Loading**: JPA lazy loading for collections

### Caching Strategy
- **Entity Caching**: Can be implemented with Spring Cache
- **Query Caching**: Hibernate second-level cache (configurable)

### Scalability
- **Stateless Design**: JWT tokens, no server-side sessions
- **Microservice Ready**: Clear service boundaries
- **Database Scaling**: Read replicas, connection pooling

## 🔍 Monitoring and Observability

### Available Endpoints
- **Health Check**: `/actuator/health`
- **Metrics**: `/actuator/metrics`
- **Info**: `/actuator/info`

### Logging
- **Framework**: Logback (Spring Boot default)
- **Levels**: Configurable via `application.properties`
- **Format**: JSON format recommended for production

This architecture provides a solid foundation for a scalable, maintainable car rental system with clear separation of concerns and extensible design patterns.
