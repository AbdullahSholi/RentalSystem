# Car Rental System

A comprehensive Spring Boot-based car rental management system with advanced payment processing, role-based security, and multi-vehicle type support.

## 🚗 Project Overview

This rental system provides a complete solution for managing vehicle rentals with features including:

- **Multi-vehicle Support**: Cars, Buses, Trucks, Vans, and Motorcycles
- **Advanced Payment Processing**: Stripe and PayPal integration with multiple payment strategies
- **Role-based Security**: JWT authentication with USER, ADMIN, and SUPER_ADMIN roles
- **Comprehensive Management**: Reservations, rentals, invoicing, and notifications
- **Branch Management**: Multi-location support with parking stall management
- **Employee & Department Management**: Complete organizational structure

## 🏗️ Technology Stack

- **Backend**: Spring Boot 3.1.0, Java 17
- **Database**: MySQL with JPA/Hibernate
- **Security**: Spring Security with JWT
- **Payment**: Stripe SDK, PayPal REST API
- **Documentation**: OpenAPI 3 (Swagger)
- **Testing**: JUnit 5, Mockito, TestContainers
- **Build Tool**: Maven

## 📋 Prerequisites

- Java 17 or higher
- MySQL 8.0+
- Maven 3.6+
- Stripe Account (for payment processing)
- PayPal Developer Account (for payment processing)

## 🚀 Quick Start

### 1. Clone the Repository

```bash
git clone <repository-url>
cd RentalSystem
```

### 2. Database Setup

```bash
# Create MySQL database
mysql -u root -p
CREATE DATABASE rental;
```

### 3. Configure Application

Update `src/main/resources/application.properties`:

```properties
# Database Configuration
spring.datasource.url=jdbc:mysql://localhost:3306/rental
spring.datasource.username=your_username
spring.datasource.password=your_password

# Payment Configuration
stripe.api.secret-key=your_stripe_secret_key
stripe.api.publishable-key=your_stripe_publishable_key
paypal.client.id=your_paypal_client_id
paypal.client.secret=your_paypal_client_secret

# JWT Configuration
security.jwt.secret-key=your_jwt_secret_key
```

### 4. Run the Application

```bash
# Using Maven
./mvnw spring-boot:run

# Or using Maven Wrapper on Windows
mvnw.cmd spring-boot:run
```

The application will start on `http://localhost:9090`

### 5. Access API Documentation

Visit `http://localhost:9090/swagger-ui.html` for interactive API documentation.

## 📚 Documentation

This project includes comprehensive documentation organized into multiple files:

### Core Documentation

- **[API Documentation](docs/API.md)** - Complete REST API reference
- **[Architecture Guide](docs/ARCHITECTURE.md)** - System design and patterns
- **[Database Schema](docs/DATABASE.md)** - Data model and relationships

### Feature Documentation

- **[Security Guide](docs/SECURITY.md)** - Authentication and authorization
- **[Payment System](docs/PAYMENT.md)** - Payment processing and integrations

## 🔑 Default Users

The system comes with pre-configured roles:


| Role        | Username              | Password     | Description        |
| ----------- | --------------------- | ------------ | ------------------ |
| SUPER_ADMIN | super.admin@email.com | (configured) | Full system access |
| USER        | (register new)        | -            | Customer access    |

## 🏃‍♂️ Running Tests

```bash
# Run all tests
./mvnw test

# Run specific test suite
./mvnw test -Dtest=PaymentControllerUnitTests
```

## 🌟 Key Features

### Vehicle Management

- Support for 5 vehicle types (Car, Bus, Truck, Van, Motor)
- Real-time availability tracking
- Location-based services with GPS coordinates
- Vehicle status management

### Payment Processing

- Multiple payment methods (Card, On Arrival)
- Stripe and PayPal integration
- Strategy pattern implementation
- Secure payment processing

### Reservation System

- Date-based availability checking
- Additional services integration
- Status tracking (PENDING, CONFIRMED, CANCELLED)
- Customer-specific reservations

### Security Features

- JWT-based authentication
- Role-based access control
- Password encryption with BCrypt
- CORS configuration

### Notification System

- Customer notifications
- Read/unread status tracking
- Timestamp management

## 🔧 Development

### Project Structure

```
src/
├── main/java/com/rental/rental/
│   ├── controller/          # REST Controllers
│   ├── service/            # Business Logic
│   ├── model/              # JPA Entities
│   ├── dto/                # Data Transfer Objects
│   ├── repository/         # Data Access Layer
│   ├── configs/            # Configuration Classes
│   └── services/           # Authentication Services
└── test/                   # Test Classes
```

### Design Patterns Used

- **Strategy Pattern**: Payment processing
- **Factory Pattern**: Payment method selection
- **Template Method**: Card payment processing
- **Repository Pattern**: Data access
- **DTO Pattern**: Data transfer

## 🤝 Contributing

1. Fork the repository
2. Create a feature branch
3. Commit your changes
4. Push to the branch
5. Create a Pull Request

## 📄 License

This project is licensed under the MIT License - see the LICENSE file for details.

## 📞 Support

For support and questions:

- Check the [documentation](docs/)
- Review the API documentation at `/swagger-ui.html`
- Create an issue in the repository

---

**Note**: This documentation reflects the current state of the project. For the most up-to-date information, refer to the individual documentation files in the `docs/` directory.
