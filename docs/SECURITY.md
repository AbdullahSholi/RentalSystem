# Security Documentation

This document describes the security architecture, authentication, authorization, and security configurations of the Car Rental System.

## 🔐 Security Overview

The system implements a comprehensive security model with:
- **JWT-based Authentication**
- **Role-based Access Control (RBAC)**
- **Password Encryption**
- **Stateless Session Management**
- **CORS Configuration**
- **Method-level Security**

## 🔑 Authentication System

### JWT (JSON Web Token) Authentication

The system uses JWT tokens for stateless authentication:

```java
@Service
public class JwtService {
    @Value("${security.jwt.secret-key}")
    private String secretKey;
    
    @Value("${security.jwt.expiration-time}")
    private long jwtExpiration; // 3600000ms (1 hour)
}
```

#### Token Structure
```
Header.Payload.Signature
```

#### Token Claims
- **Subject**: User email
- **Issued At**: Token creation time
- **Expiration**: Token expiry time
- **Custom Claims**: Additional user information

### Authentication Flow

```
1. User Login → AuthenticationController
2. Credentials Validation → AuthenticationService
3. JWT Token Generation → JwtService
4. Token Response → Client
5. Subsequent Requests → JWT Filter → Validation
```

### Login Process

#### Registration Endpoint
```http
POST /auth/signup
Content-Type: application/json

{
  "fullName": "John Doe",
  "email": "john.doe@example.com",
  "password": "securePassword123"
}
```

#### Login Endpoint
```http
POST /auth/login
Content-Type: application/json

{
  "email": "john.doe@example.com",
  "password": "securePassword123"
}
```

#### Response
```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "expiresIn": 3600000
}
```

## 👥 Role-Based Access Control

### Role Hierarchy

```
SUPER_ADMIN (Highest Privilege)
    ↓
ADMIN (Management Privilege)
    ↓
USER (Customer Privilege)
```

### Role Definitions

#### USER Role
- **Description**: Default customer role
- **Permissions**:
  - View own profile
  - Create/view own reservations
  - View available vehicles
  - Process payments
  - View own notifications

#### ADMIN Role
- **Description**: System administrator
- **Permissions**:
  - All USER permissions
  - Manage customers
  - Manage vehicles
  - View all reservations
  - Manage branches
  - Process payments for customers

#### SUPER_ADMIN Role
- **Description**: System super administrator
- **Permissions**:
  - All ADMIN permissions
  - Manage admin accounts
  - System configuration
  - Full database access

### Role Implementation

#### Database Schema
```sql
CREATE TABLE `roles` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` enum('ADMIN','SUPER_ADMIN','USER') NOT NULL,
  `description` varchar(255) DEFAULT NULL,
  `created_at` datetime(6) DEFAULT NULL,
  `updated_at` datetime(6) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `UK_ofx66keruapi6vyqpv6f2or37` (`name`)
);
```

#### Customer-Role Relationship
```java
@Entity
public class Customer implements UserDetails {
    @ManyToOne
    @JoinColumn(name = "role_id")
    private Role role;
    
    @Override
    public Collection<? extends GrantedAuthority> getAuthorities() {
        return List.of(new SimpleGrantedAuthority("ROLE_" + role.getName().name()));
    }
}
```

## 🛡️ Security Configuration

### Spring Security Configuration

```java
@Configuration
@EnableWebSecurity
@EnableMethodSecurity
public class SecurityConfiguration {
    
    @Bean
    public SecurityFilterChain securityFilterChain(HttpSecurity http) throws Exception {
        http
            .csrf(csrf -> csrf.disable())
            .authorizeHttpRequests(auth -> auth
                .requestMatchers("/auth/**").permitAll()
                .anyRequest().authenticated()
            )
            .sessionManagement(session -> session
                .sessionCreationPolicy(SessionCreationPolicy.STATELESS)
            )
            .authenticationProvider(authenticationProvider)
            .addFilterBefore(jwtAuthenticationFilter, UsernamePasswordAuthenticationFilter.class);
        
        return http.build();
    }
}
```

### JWT Authentication Filter

```java
@Component
public class JwtAuthenticationFilter extends OncePerRequestFilter {
    
    @Override
    protected void doFilterInternal(
            HttpServletRequest request,
            HttpServletResponse response,
            FilterChain filterChain) throws ServletException, IOException {
        
        final String authHeader = request.getHeader("Authorization");
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            filterChain.doFilter(request, response);
            return;
        }
        
        final String jwt = authHeader.substring(7);
        final String userEmail = jwtService.extractUsername(jwt);
        
        if (userEmail != null && SecurityContextHolder.getContext().getAuthentication() == null) {
            UserDetails userDetails = this.userDetailsService.loadUserByUsername(userEmail);
            
            if (jwtService.isTokenValid(jwt, userDetails)) {
                UsernamePasswordAuthenticationToken authToken = new UsernamePasswordAuthenticationToken(
                        userDetails, null, userDetails.getAuthorities());
                authToken.setDetails(new WebAuthenticationDetailsSource().buildDetails(request));
                SecurityContextHolder.getContext().setAuthentication(authToken);
            }
        }
        
        filterChain.doFilter(request, response);
    }
}
```

## 🔒 Method-Level Security

### Security Annotations

#### @PreAuthorize
Used to check permissions before method execution:

```java
@GetMapping("/customers")
@PreAuthorize("hasAnyRole('ADMIN', 'SUPER_ADMIN')")
public ResponseEntity<List<CustomerDTO>> getAllCustomers() {
    // Only ADMIN and SUPER_ADMIN can access
}

@GetMapping("/customer")
@PreAuthorize("isAuthenticated()")
public ResponseEntity<CustomerDTO> getCustomer(@RequestParam int customerId) {
    // Any authenticated user can access
}

@GetMapping("/admins")
@PreAuthorize("hasRole('SUPER_ADMIN')")
public ResponseEntity<List<AdminDTO>> getAdmins() {
    // Only SUPER_ADMIN can access
}
```

### Permission Matrix

| Endpoint | USER | ADMIN | SUPER_ADMIN |
|----------|------|-------|-------------|
| `/auth/**` | ✅ | ✅ | ✅ |
| `GET /customers` | ❌ | ✅ | ✅ |
| `GET /customer` | ✅ | ✅ | ✅ |
| `GET /vehicles` | ❌ | ✅ | ✅ |
| `GET /vehicle` | ✅ | ✅ | ✅ |
| `POST /vehicle` | ❌ | ✅ | ✅ |
| `GET /reservations` | ❌ | ✅ | ✅ |
| `POST /reservation` | ✅ | ✅ | ✅ |
| `POST /process` (payment) | ✅ | ✅ | ✅ |
| `GET /admins` | ❌ | ❌ | ✅ |
| `GET /branches` | ❌ | ✅ | ✅ |

## 🔐 Password Security

### Password Encryption
```java
@Bean
BCryptPasswordEncoder passwordEncoder() {
    return new BCryptPasswordEncoder();
}
```

### Password Hashing Process
1. **Registration**: Plain password → BCrypt hash → Database storage
2. **Login**: Plain password → BCrypt verification → Authentication

### Password Requirements
- Minimum length: Not enforced (should be implemented)
- Complexity: Not enforced (should be implemented)
- Storage: BCrypt hashed with salt

## 🌐 CORS Configuration

```java
@Bean
CorsConfigurationSource corsConfigurationSource() {
    CorsConfiguration configuration = new CorsConfiguration();
    configuration.setAllowedOrigins(List.of("http://localhost:8005"));
    configuration.setAllowedMethods(List.of("GET", "POST"));
    configuration.setAllowedHeaders(List.of("Authorization", "Content-Type"));
    
    UrlBasedCorsConfigurationSource source = new UrlBasedCorsConfigurationSource();
    source.registerCorsConfiguration("/**", configuration);
    
    return source;
}
```

## 🔧 Security Configuration Properties

### JWT Configuration
```properties
# JWT Secret Key (should be environment variable in production)
security.jwt.secret-key=XK+1eTBW8EqvLQF5xgX9vQzSbAxB6lP1aOeY3oJ9ERo

# JWT Expiration Time (1 hour)
security.jwt.expiration-time=3600000
```

### Database Security
```properties
# Database credentials (should be environment variables)
spring.datasource.username=root
spring.datasource.password=
```

## 🚨 Security Best Practices

### Implemented
✅ **JWT Stateless Authentication**
✅ **Password Hashing with BCrypt**
✅ **Role-based Access Control**
✅ **Method-level Security**
✅ **CORS Configuration**
✅ **CSRF Protection Disabled** (appropriate for stateless API)

### Recommended Improvements

#### 1. Environment Variables
```bash
# Production environment variables
JWT_SECRET_KEY=your-production-secret-key
DB_USERNAME=your-db-username
DB_PASSWORD=your-db-password
STRIPE_SECRET_KEY=your-stripe-secret
PAYPAL_CLIENT_SECRET=your-paypal-secret
```

#### 2. Password Policy
```java
@Pattern(regexp = "^(?=.*[a-z])(?=.*[A-Z])(?=.*\\d)(?=.*[@$!%*?&])[A-Za-z\\d@$!%*?&]{8,}$",
         message = "Password must contain at least 8 characters, one uppercase, one lowercase, one digit and one special character")
private String password;
```

#### 3. Rate Limiting
```java
@Component
public class RateLimitingFilter implements Filter {
    // Implement rate limiting logic
}
```

#### 4. Input Validation
```java
@Valid
@RequestBody
public ResponseEntity<?> createCustomer(@Valid @RequestBody CustomerDTO customerDTO) {
    // Validation annotations ensure input safety
}
```

#### 5. Audit Logging
```java
@EventListener
public void handleAuthenticationSuccess(AuthenticationSuccessEvent event) {
    // Log successful authentication
}
```

## 🔍 Security Testing

### Authentication Tests
```java
@Test
public void testJwtTokenGeneration() {
    // Test JWT token creation and validation
}

@Test
public void testUnauthorizedAccess() {
    // Test access without valid token
}
```

### Authorization Tests
```java
@Test
@WithMockUser(roles = "USER")
public void testUserAccessToAdminEndpoint() {
    // Should return 403 Forbidden
}

@Test
@WithMockUser(roles = "ADMIN")
public void testAdminAccessToUserEndpoint() {
    // Should return 200 OK
}
```

## 🚨 Security Monitoring

### Available Endpoints
- **Health Check**: `/actuator/health`
- **Security Events**: Can be monitored via Spring Security events
- **Failed Authentication**: Logged automatically

### Recommended Monitoring
- Failed login attempts
- Token expiration events
- Unauthorized access attempts
- Role escalation attempts

This security implementation provides a solid foundation for protecting the car rental system with industry-standard practices and clear role-based access control.
