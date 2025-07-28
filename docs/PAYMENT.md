# Payment System Documentation

This document describes the comprehensive payment processing system implemented in the Car Rental System, including multiple payment strategies, provider integrations, and design patterns.

## 💳 Payment System Overview

The payment system supports multiple payment methods and providers through a flexible architecture using:
- **Strategy Pattern** for payment method selection
- **Factory Pattern** for payment service creation
- **Template Method Pattern** for card payment processing
- **Multiple Payment Providers** (Stripe, PayPal)
- **Flexible Payment Methods** (Card, On Arrival)

## 🏗️ Payment Architecture

### Design Pattern Implementation

```
PaymentController
       ↓
PaymentFactoryService (Factory Pattern)
       ↓
PaymentStrategy (Strategy Pattern)
       ↓
CardPaymentTemplate (Template Method Pattern)
       ↓
Provider Implementation (Stripe/PayPal)
```

## 🔧 Payment Components

### 1. Payment Strategy Interface

```java
public interface PaymentStrategy {
    boolean pay(PaymentDTO paymentDTO);
}
```

### 2. Payment Factory Service

```java
@Service
public class PaymentFactoryService {
    
    @Autowired
    private CardPaymentContext cardPaymentContext;
    
    @Autowired
    private OnArrivalPaymentContext onArrivalPaymentContext;
    
    public PaymentStrategy getPaymentService(PaymentDTO paymentDTO) {
        if("onArrival".equalsIgnoreCase(paymentDTO.getPaymentMethod())) {
            return onArrivalPaymentContext;
        } else if("Card".equalsIgnoreCase(paymentDTO.getPaymentMethod())) {
            return cardPaymentContext;
        } else {
            throw new IllegalArgumentException("Unsupported payment method!");
        }
    }
}
```

### 3. Card Payment Context

```java
@Service
public class CardPaymentContext implements PaymentStrategy {
    
    @Autowired
    private StripePayment stripePayment;
    
    @Autowired
    private PayPalPayment payPalPayment;
    
    @Override
    public boolean pay(PaymentDTO paymentDTO) {
        if ("Stripe".equalsIgnoreCase(paymentDTO.getCardType())) {
            return stripePayment.pay(paymentDTO);
        } else if ("PayPal".equalsIgnoreCase(paymentDTO.getCardType())) {
            return payPalPayment.pay(paymentDTO);
        } else {
            throw new IllegalArgumentException("Unsupported card type!");
        }
    }
}
```

## 💰 Payment Methods

### 1. Card Payment

#### Supported Card Types
- **Stripe**: Credit/Debit card processing
- **PayPal**: PayPal account payments

#### Card Payment Template

```java
public abstract class CardPaymentTemplate implements PaymentStrategy {
    
    @Override
    public final boolean pay(PaymentDTO paymentDTO) {
        validateCard(paymentDTO);
        boolean isProcessed = processPaymentWithProvider(paymentDTO);
        confirmPayment(paymentDTO);
        return isProcessed;
    }
    
    private void validateCard(PaymentDTO paymentDTO) {
        System.out.println("Validating card information...");
        // Card validation logic
    }
    
    protected abstract boolean processPaymentWithProvider(PaymentDTO paymentDTO);
    
    private void confirmPayment(PaymentDTO paymentDTO) {
        System.out.println("Payment confirmation sent.");
        // Confirmation logic
    }
}
```

### 2. On Arrival Payment

```java
@Service
public class OnArrivalPaymentContext implements PaymentStrategy {
    
    @Override
    public boolean pay(PaymentDTO paymentDTO) {
        System.out.println("Payment will be made on arrival.");
        return true;
    }
}
```

## 🔌 Payment Provider Integrations

### Stripe Integration

#### Configuration
```properties
# Stripe API Keys
stripe.api.secret-key=sk_test_51QFrt0GBPHHSXwUhggJodgK4a9MwfPSoFB1PUDRH2mbo3FdcdSQGNeTDTxlHdlWI9MZSh8oUrKUUltuIsNrgF3vR00otxW8wPB
stripe.api.publishable-key=pk_test_51QFrt0GBPHHSXwUh7T0qS4TNcmv4x087EQEd5cPfKMvDYakYtoj1ordIlK1CgiQnkS2PfHl7jLYAcCHCHPvboS0400Tx02Tn1m
```

#### Stripe Configuration Class
```java
@Configuration
public class StripeConfig {
    
    @Value("${stripe.api.secret-key}")
    private String secretKey;
    
    @Value("${stripe.api.publishable-key}")
    private String publishableKey;
    
    // Getters
}
```

#### Stripe Payment Implementation
```java
@Service
public class StripePayment extends CardPaymentTemplate {
    
    @Autowired
    private StripeConfig stripeConfig;
    
    @Override
    protected boolean processPaymentWithProvider(PaymentDTO paymentDTO) {
        try {
            Stripe.apiKey = stripeConfig.getSecretKey();
            
            PaymentIntentCreateParams params = PaymentIntentCreateParams.builder()
                    .setAmount((long) (paymentDTO.getTotalAmount() * 100)) // Convert to cents
                    .setCurrency("usd")
                    .build();
            
            PaymentIntent paymentIntent = PaymentIntent.create(params);
            System.out.println("Payment processed through Stripe: " + paymentIntent.getId());
            
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
    }
}
```

### PayPal Integration

#### Configuration
```properties
# PayPal API Credentials
paypal.client.id=ASmtwKnlW_8cG9U4viONcNv-RKhqah0H1Gy18Hqo9fwVLKjE8f_yRfV1jR0sUnAw5fD4d7CsamqbEXo6
paypal.client.secret=EFkaxSkpMGA0ZrdTna1ao733susDkerusrkyCFYjAxSAsGNdYoveOFhA_7NNxRYxmu0BH_E0rGKoRQ5e
```

#### PayPal Configuration Class
```java
@Configuration
public class PayPalConfig {
    
    @Value("${paypal.client.id}")
    private String clientId;
    
    @Value("${paypal.client.secret}")
    private String clientSecret;
    
    // Getters
}
```

#### PayPal Payment Implementation
```java
@Service
public class PayPalPayment extends CardPaymentTemplate {
    
    @Autowired
    private PayPalConfig payPalConfig;
    
    @Override
    protected boolean processPaymentWithProvider(PaymentDTO paymentDTO) {
        APIContext apiContext = new APIContext(
            payPalConfig.getClientId(), 
            payPalConfig.getClientSecret(), 
            "sandbox"
        );
        
        try {
            // Create payment object
            Payment payment = new Payment();
            payment.setIntent("sale");
            payment.setPayer(createPayer());
            payment.setTransactions(createTransactionList(paymentDTO));
            payment.setRedirectUrls(createRedirectUrls());
            
            // Create payment
            Payment createdPayment = payment.create(apiContext);
            System.out.println("PayPal payment created: " + createdPayment.getId());
            
            return true;
        } catch (PayPalRESTException e) {
            e.printStackTrace();
            return false;
        }
    }
    
    private Payer createPayer() {
        Payer payer = new Payer();
        payer.setPaymentMethod("paypal");
        return payer;
    }
    
    private List<Transaction> createTransactionList(PaymentDTO paymentDTO) {
        Transaction transaction = new Transaction();
        
        Amount amount = new Amount();
        amount.setCurrency("USD");
        amount.setTotal(String.valueOf(paymentDTO.getTotalAmount()));
        
        transaction.setAmount(amount);
        transaction.setDescription("Car rental payment");
        
        return List.of(transaction);
    }
    
    private RedirectUrls createRedirectUrls() {
        RedirectUrls redirectUrls = new RedirectUrls();
        redirectUrls.setCancelUrl("http://localhost:9090/payment/cancel");
        redirectUrls.setReturnUrl("http://localhost:9090/payment/success");
        return redirectUrls;
    }
}
```

## 📊 Payment Data Model

### Payment Entity
```java
@Entity
public class Payment {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private int paymentId;
    
    private double totalAmount;
    private String paymentMethod;  // "card" or "onArrival"
    private String cardType;       // "Stripe" or "PayPal"
    
    @Enumerated(EnumType.STRING)
    private PaymentStatus status;  // PENDING, COMPLETED, FAILED
    
    private Date paymentDate;
}
```

### Payment DTO
```java
@Data
@NoArgsConstructor
@AllArgsConstructor
@Builder
public class PaymentDTO {
    private int paymentId;
    private double totalAmount;
    private String paymentMethod;
    private String cardType;
    private PaymentStatus status;
    private Date paymentDate;
}
```

### Payment Status Enum
```java
public enum PaymentStatus {
    PENDING,
    COMPLETED,
    FAILED
}
```

## 🔄 Payment Processing Flow

### 1. Payment Request Flow
```
1. Client sends payment request → PaymentController
2. PaymentController → PaymentService.processPayment()
3. PaymentService → PaymentFactoryService.getPaymentService()
4. Factory returns appropriate PaymentStrategy
5. Strategy.pay() → Provider-specific implementation
6. Provider processes payment → Returns result
7. PaymentService saves payment record
8. Response sent to client
```

### 2. Payment API Usage

#### Process Payment Request
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

#### Successful Response
```json
{
  "message": "Payment processed successfully"
}
```

#### Error Response
```json
{
  "message": "Payment processing failed"
}
```

## 💡 Payment Service Implementation

### Main Payment Service
```java
@Service
public class PaymentService {
    
    @Autowired
    private PaymentRepository paymentRepository;
    
    @Autowired
    private PaymentFactoryService paymentFactoryService;
    
    public String processPayment(PaymentDTO paymentDTO) {
        try {
            // Get appropriate payment strategy
            PaymentStrategy paymentStrategy = paymentFactoryService.getPaymentService(paymentDTO);
            
            // Process payment
            boolean isSuccessful = paymentStrategy.pay(paymentDTO);
            
            if (isSuccessful) {
                // Save payment record
                Payment payment = convertToEntity(paymentDTO);
                payment.setStatus(PaymentStatus.COMPLETED);
                payment.setPaymentDate(new Date());
                paymentRepository.save(payment);
                
                return "Payment processed successfully";
            } else {
                return "Payment processing failed";
            }
        } catch (Exception e) {
            return "Payment processing error: " + e.getMessage();
        }
    }
    
    public List<PaymentDTO> getAllPayments() {
        return paymentRepository.findAll()
                .stream()
                .map(this::convertToDTO)
                .collect(Collectors.toList());
    }
}
```

## 🔒 Payment Security

### Security Measures
1. **API Key Protection**: Stored in configuration files (should use environment variables)
2. **HTTPS Required**: All payment communications should use HTTPS
3. **Token Validation**: JWT authentication required for payment endpoints
4. **Input Validation**: Payment amounts and methods validated
5. **Error Handling**: Sensitive information not exposed in error messages

### Security Best Practices

#### Environment Variables (Recommended)
```bash
# Production environment variables
STRIPE_SECRET_KEY=your-production-stripe-secret
STRIPE_PUBLISHABLE_KEY=your-production-stripe-publishable
PAYPAL_CLIENT_ID=your-production-paypal-client-id
PAYPAL_CLIENT_SECRET=your-production-paypal-secret
```

#### Secure Configuration
```java
@Configuration
public class PaymentSecurityConfig {
    
    @Value("${STRIPE_SECRET_KEY:#{null}}")
    private String stripeSecretKey;
    
    @PostConstruct
    public void validateConfiguration() {
        if (stripeSecretKey == null) {
            throw new IllegalStateException("Stripe secret key not configured");
        }
    }
}
```

## 🧪 Testing Payment System

### Unit Tests
```java
@ExtendWith(MockitoExtension.class)
class PaymentServiceTest {
    
    @Mock
    private PaymentRepository paymentRepository;
    
    @Mock
    private PaymentFactoryService paymentFactoryService;
    
    @InjectMocks
    private PaymentService paymentService;
    
    @Test
    void testSuccessfulPaymentProcessing() {
        // Test successful payment flow
    }
    
    @Test
    void testFailedPaymentProcessing() {
        // Test failed payment handling
    }
}
```

### Integration Tests
```java
@SpringBootTest
@TestPropertySource(properties = {
    "stripe.api.secret-key=test-key",
    "paypal.client.id=test-id"
})
class PaymentIntegrationTest {
    
    @Test
    void testStripePaymentIntegration() {
        // Test Stripe integration
    }
    
    @Test
    void testPayPalPaymentIntegration() {
        // Test PayPal integration
    }
}
```

## 📈 Payment Analytics

### Available Metrics
- Total payments processed
- Payment success/failure rates
- Payment method distribution
- Revenue by time period

### Monitoring Endpoints
```java
@GetMapping("/payments/stats")
public ResponseEntity<PaymentStats> getPaymentStatistics() {
    // Return payment analytics
}
```

## 🔄 Extending Payment System

### Adding New Payment Provider

1. **Create Provider Implementation**
```java
@Service
public class NewProviderPayment extends CardPaymentTemplate {
    @Override
    protected boolean processPaymentWithProvider(PaymentDTO paymentDTO) {
        // Implement new provider logic
    }
}
```

2. **Update Card Payment Context**
```java
@Override
public boolean pay(PaymentDTO paymentDTO) {
    switch (paymentDTO.getCardType().toLowerCase()) {
        case "stripe": return stripePayment.pay(paymentDTO);
        case "paypal": return payPalPayment.pay(paymentDTO);
        case "newprovider": return newProviderPayment.pay(paymentDTO);
        default: throw new IllegalArgumentException("Unsupported card type!");
    }
}
```

3. **Add Configuration**
```properties
newprovider.api.key=your-api-key
newprovider.api.secret=your-api-secret
```

This payment system provides a flexible, extensible architecture that can easily accommodate new payment methods and providers while maintaining security and reliability.
