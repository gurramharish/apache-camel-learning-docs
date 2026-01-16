# Apache Camel Professional Development Guide
## A Comprehensive Curriculum for Advanced Java Developers

---

## Table of Contents

1. [Introduction & Curriculum Overview](#introduction)
2. [Part 1: Fundamentals](#part-1-fundamentals)
   - Chapter 1: Core Concepts & Architecture
   - Chapter 2: Java DSL Fundamentals
   - Chapter 3: Routing & Message Flow
3. [Part 2: Integration Patterns](#part-2-integration-patterns)
   - Chapter 4: REST API Integration
   - Chapter 5: File & SFTP Operations
   - Chapter 6: Content-Based Routing
   - Chapter 7: Error Handling & Resilience
4. [Part 3: Advanced Patterns](#part-3-advanced-patterns)
   - Chapter 8: Splitters & Aggregators
   - Chapter 9: Processors & Beans
   - Chapter 10: Complex Transformations
5. [Part 4: Production & Operations](#part-4-production--operations)
   - Chapter 11: Testing & Quality Assurance
   - Chapter 12: Security & OAuth2/JWT
   - Chapter 13: Observability & Monitoring
   - Chapter 14: Cloud-Native Integration
6. [Part 5: Real-World Projects](#part-5-real-world-projects)
   - Project 1: Multi-Channel Order Processing
   - Project 2: Real-Time Data Integration Pipeline
   - Project 3: Microservice API Gateway

---

## Introduction

Apache Camel is an open-source integration framework that implements Enterprise Integration Patterns (EIP) and simplifies the connection of diverse systems. As a seasoned Java developer with 13+ years of backend engineering experience, you already understand distributed systems, Spring Boot architecture, and complex asynchronous processing. This curriculum builds on those strengths to make you expert-level in Apache Camel—capable of architecting enterprise-grade integration solutions, optimizing for performance, and implementing production-ready patterns.

### What Makes This Different

This guide assumes:
- Proficiency with Spring Boot and Spring ecosystem
- Understanding of microservices architecture
- Familiarity with enterprise messaging and async patterns
- Comfort with building distributed systems
- Knowledge of Java 11+ features (records, var, pattern matching)

Rather than explaining basics like "what is a message broker," we focus on architectural decisions, performance optimization, production patterns, and advanced techniques.

### Learning Outcomes

By completing this curriculum, you will be able to:
- Design and implement scalable integration solutions using Apache Camel
- Build REST APIs, handle SFTP operations, and manage complex file processing
- Architect error handling and resilience patterns for production environments
- Optimize Camel applications for cloud-native deployment (Kubernetes, Quarkus)
- Implement security patterns (OAuth2, JWT) in integration routes
- Monitor and observe Camel applications effectively
- Build microservices integration architectures

---

# PART 1: FUNDAMENTALS

## Chapter 1: Core Concepts & Architecture

### 1.1 What is Apache Camel?

Apache Camel is a rules-based routing and mediation engine. It provides a framework for defining integration logic using enterprise integration patterns, eliminating the need to write boilerplate integration code.

**Core Philosophy**: Write integration logic once, apply it everywhere. Change endpoints (from file to JMS, from HTTP to SFTP) without changing route logic.

### 1.2 Three Core Concepts

#### 1.2.1 Endpoints
Endpoints represent connection points to external systems. They can be:
- **Consumers**: Receive messages from external systems (e.g., `from("sftp://server:22/inbox")`)
- **Producers**: Send messages to external systems (e.g., `.to("http://api.example.com")`)

Camel supports 300+ components with standardized URI syntax:
```
protocol://hostname:port/path?option1=value1&option2=value2
```

#### 1.2.2 Routes
Routes define how messages flow from source to destination, with transformations and processing along the way. A route always has a source endpoint and at least one destination.

#### 1.2.3 The Exchange
The Exchange is Camel's internal message wrapper containing:
- **In Message**: The incoming message with headers and body
- **Out Message**: The outgoing message (if request-reply pattern)
- **Properties**: Exchange-level metadata
- **Exception**: Any thrown exception
- **Pattern**: InOnly (fire-and-forget) or InOut (request-reply)

### 1.3 CamelContext Lifecycle

The `CamelContext` is your application's integration runtime. It manages routes, components, and endpoints.

```java
CamelContext context = new DefaultCamelContext();

// Add routes
context.addRoutes(new RouteBuilder() {
    public void configure() throws Exception {
        from("timer://ticker?fixedRate=true&period=1000")
            .to("log:ticker");
    }
});

// Start processing (CRITICAL - without this, routes don't run)
context.start();

// Keep running
Thread.sleep(10000);

// Graceful shutdown
context.stop();
```

**Best Practice**: Always call `context.start()` and `context.stop()` explicitly. In Spring Boot, this is automatic.

### 1.4 Spring Boot Integration

Camel's Spring Boot Starter provides auto-configuration:

```xml
<dependency>
    <groupId>org.apache.camel.springboot</groupId>
    <artifactId>camel-spring-boot-starter</artifactId>
    <version>4.15.0</version>
</dependency>
```

Routes are auto-discovered and registered as Spring beans:

```java
@SpringBootApplication
public class CamelApplication {
    public static void main(String[] args) {
        SpringApplication.run(CamelApplication.class, args);
    }
}

@Component
public class OrderRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("direct:orders")
            .log("Processing order: ${body}")
            .to("kafka:orders");
    }
}
```

### 1.5 Architecture Patterns

#### Simple Pipeline Pattern
```
from(source) → processor1 → processor2 → processor3 → to(destination)
```

Camel chains operations by default (pipeline pattern).

#### Choice Pattern
```
from(source) 
  → choice()
    → when(condition1) → processor1 → to(dest1)
    → when(condition2) → processor2 → to(dest2)
    → otherwise() → processor3 → to(dest3)
  → end()
```

#### Error Handling Pattern
```
from(source)
  → errorHandler(deadLetterChannel("jms:queue:dlq"))
  → processor
  → to(destination)
```

### 1.6 Key Components for This Course

For REST APIs:
```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-http</artifactId>
    <version>4.15.0</version>
</dependency>
```

For SFTP/FTP:
```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-ftp</artifactId>
    <version>4.15.0</version>
</dependency>
```

For messaging:
```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-kafka</artifactId>
    <version>4.15.0</version>
</dependency>
```

### 1.7 Chapter Exercise: Hello World Integration

**Exercise 1.1**: Create a Spring Boot application that:
1. Reads messages from a timer endpoint every 2 seconds
2. Logs the message with timestamp
3. Transforms the message body
4. Outputs to a file

```java
@Component
public class HelloWorldRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("timer:tick?period=2000")
            .setBody(simple("Hello from Camel at ${date:now:yyyy-MM-dd HH:mm:ss}"))
            .log("${body}")
            .to("file:target/messages?fileName=message-${date:now:yyyyMMddHHmmss}.txt");
    }
}
```

**Exercise 1.2**: Extend the above to use a processor for custom logic:

```java
@Component
public class GreeterProcessor implements Processor {
    @Override
    public void process(Exchange exchange) throws Exception {
        String body = exchange.getIn().getBody(String.class);
        exchange.getIn().setBody("PROCESSED: " + body);
    }
}

@Component
public class HelloWorldRoute extends RouteBuilder {
    @Autowired
    private GreeterProcessor processor;
    
    @Override
    public void configure() throws Exception {
        from("timer:tick?period=2000")
            .setBody(simple("Hello from Camel"))
            .process(processor)
            .log("${body}")
            .to("file:target/messages?fileName=message-${date:now:yyyyMMddHHmmss}.txt");
    }
}
```

---

## Chapter 2: Java DSL Fundamentals

### 2.1 RouteBuilder Pattern

The `RouteBuilder` is your canvas for defining integration routes. Unlike XML-based DSL, Java DSL provides type safety and IDE support.

```java
public abstract class RouteBuilder {
    public abstract void configure() throws Exception;
    
    // Provided by parent - used in configure()
    protected RouteDefinition from(String uri) { ... }
    protected ChoiceDefinition choice() { ... }
    protected ErrorHandlerBuilder errorHandler(...) { ... }
    // ... 300+ additional methods for DSL
}
```

### 2.2 Fluent Interface & Method Chaining

Camel DSL uses fluent interface heavily:

```java
from("direct:orders")              // RouteDefinition
    .filter(...)                    // FilterDefinition
    .to("log:orders")              // ToDefinition
    .setHeader("id", simple("...")) // SetHeaderDefinition
    .to("kafka:orders");           // ToDefinition returns RouteDefinition
```

Each method returns a definition object allowing further chaining. Terminal operations are `.to()`, `.end()`, or implicit route completion.

### 2.3 Expressions & Predicates

#### Simple Language
Quick expressions without external dependencies:

```java
// Expressions (return values)
from("kafka:orders")
    .setBody(simple("Order: ${body}"))
    .setHeader("timestamp", simple("${date:now:yyyy-MM-dd}"))
    .setHeader("order-id", simple("${header.CamelFileName}"));

// Predicates (return true/false)
from("kafka:orders")
    .filter(simple("${body.length} > 100"))
    .filter(simple("${header.priority} == 'HIGH'"))
    .to("direct:high-priority");
```

#### Using Headers & Properties
```java
// Headers come from the message
.filter(simple("${header.content-type} == 'application/json'"))

// Properties come from the exchange
.setProperty("processedAt", simple("${date:now:iso}"))
.filter(simple("${exchangeProperty.processedAt} != null"))
```

#### XPath for XML
```java
from("file:inbox")
    .filter(xpath("/order/priority = 'URGENT'"))
    .to("direct:urgent");
```

#### JSONPath
```java
from("http:localhost:8080/orders")
    .filter(jsonpath("$.amount > 1000"))
    .to("kafka:large-orders");
```

### 2.4 Setting Headers, Properties & Body

```java
from("direct:process")
    // Set body
    .setBody(simple("${body.toUpperCase()}"))
    
    // Set headers
    .setHeader("X-Processed", constant("true"))
    .setHeader("X-Order-ID", simple("${body.orderId}"))
    
    // Set properties (exchange-level, not message-level)
    .setProperty("startTime", simple("${date:now}"))
    
    // Remove headers
    .removeHeader("X-Temp")
    
    // Copy header to property
    .setProperty("originalBody", simple("${body}"))
    
    .to("log:processed");
```

**Important Distinction**:
- **Headers**: Message-level metadata, sent with the message
- **Properties**: Exchange-level metadata, not forwarded in external endpoints

### 2.5 Constants, Headers & Methods

```java
from("direct:process")
    // Constant values
    .setBody(constant("Fixed Value"))
    
    // Dynamic from headers
    .setBody(header("sourceSystem"))
    
    // Calling Java methods
    .setBody(method(orderService, "formatOrder"))
    
    // Lambda expressions
    .process(e -> e.getIn().setBody(
        "Order: " + e.getIn().getBody()
    ))
    
    .to("log:processed");
```

### 2.6 Direct Endpoints for Subroutes

Use `direct:` endpoints to create subroutes and reusable logic:

```java
@Component
public class OrderProcessingRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        // Main entry point
        from("kafka:orders")
            .to("direct:validate")
            .to("direct:enrich")
            .to("direct:save");
        
        // Subroutes
        from("direct:validate")
            .filter(simple("${body.customerId} != null"))
            .log("Order validated");
        
        from("direct:enrich")
            .enrich()
            .constant("direct:fetchCustomer")
            .to("log:enriched");
        
        from("direct:save")
            .to("kafka:saved-orders");
        
        from("direct:fetchCustomer")
            .bean(customerService, "getCustomer");
    }
}
```

### 2.7 Lambda-Based Route Definitions

Camel 3.0+ supports lambda-style route definitions—useful for microservices:

```java
@SpringBootApplication
public class CamelApp {
    public static void main(String[] args) {
        SpringApplication.run(CamelApp.class, args);
    }
    
    @Bean
    public LambdaRouteBuilder orderRoute() {
        return rb -> rb.from("kafka:orders")
                       .log("Order received: ${body}")
                       .to("direct:process");
    }
}
```

### 2.8 Chapter Exercise: Building Multi-Route Integration

**Exercise 2.1**: Create a multi-route integration with subroutes:

```java
@Component
public class InventoryRoute extends RouteBuilder {
    @Autowired
    private InventoryService inventoryService;
    
    @Override
    public void configure() throws Exception {
        // Main route
        from("timer:inventory?period=10000")
            .to("direct:checkLevels")
            .to("direct:updateMetrics");
        
        // Subroute: Check levels
        from("direct:checkLevels")
            .bean(inventoryService, "getLowStockItems")
            .choice()
                .when(simple("${body.size} > 0"))
                    .setHeader("alert", constant("true"))
                    .log("Low stock alert: ${body}")
                .otherwise()
                    .log("All inventory levels OK")
            .end();
        
        // Subroute: Update metrics
        from("direct:updateMetrics")
            .bean(inventoryService, "recordMetrics")
            .to("kafka:inventory-metrics");
    }
}
```

**Exercise 2.2**: Implement expressions and predicates:

```java
from("file:inbox")
    .filter(simple("${file:ext} == 'json'"))
    .filter(simple("${file:size} < 10485760")) // 10MB
    .setHeader("processedDate", simple("${date:now:yyyy-MM-dd}"))
    .setProperty("originalName", header(Exchange.FILE_NAME))
    .log("Processing file: ${header[CamelFileName]}")
    .to("direct:parse");
```

---

## Chapter 3: Routing & Message Flow

### 3.1 The Pipeline Pattern (Default)

By default, Camel processes messages through a pipeline:

```java
from("direct:start")
    .log("Step 1")
    .log("Step 2")
    .log("Step 3")
    .to("log:final");
```

Each step receives output from the previous step. The body flows through unchanged unless explicitly modified.

### 3.2 Direct Endpoints for Subrouting

`direct:` endpoints provide synchronous in-JVM routing:

```java
from("kafka:orders")
    .to("direct:validate");      // Forward to another route

from("direct:validate")
    .filter(...)
    .log("Validated");
```

**Key Point**: `direct:` is synchronous. Use `vm:` (virtual machine) for async, or messaging endpoints (`jms:`, `kafka:`) for true async.

### 3.3 Request-Reply Pattern (InOut)

Some endpoints support request-reply (InOut) exchange pattern:

```java
from("direct:start")
    .to("http://api.example.com/users")  // InOut - waits for response
    .log("Response: ${body}")
    .to("log:result");
```

The message exchange "pauses" until a response arrives, then continues with the response as the new body.

### 3.4 Fire-and-Forget Pattern (InOnly)

Some routes should fire-and-forget:

```java
from("kafka:orders")
    .setExchangePattern(ExchangePattern.InOnly)  // Explicit
    .to("file:processed-orders");

// Or implicitly with .to() on messaging endpoints
from("kafka:orders")
    .to("jms:orders");  // Fire-and-forget
```

### 3.5 Choice Routing (Content-Based Routing)

Route messages to different destinations based on content:

```java
from("direct:route-order")
    .choice()
        .when(simple("${header.priority} == 'URGENT'"))
            .log("URGENT order")
            .to("direct:urgent-processing")
        .when(simple("${header.priority} == 'HIGH'"))
            .log("HIGH priority order")
            .to("direct:high-processing")
        .when(jsonpath("$.amount > 5000"))
            .log("Large order")
            .to("direct:large-order-processing")
        .otherwise()
            .log("Standard order")
            .to("direct:standard-processing")
    .end();

from("direct:urgent-processing")
    .log("Processing URGENT");

from("direct:high-processing")
    .log("Processing HIGH");

from("direct:large-order-processing")
    .log("Processing LARGE");

from("direct:standard-processing")
    .log("Processing STANDARD");
```

### 3.6 Recipient List Pattern

Route to dynamic recipients:

```java
from("direct:notify")
    .setHeader("recipients", simple("${header.email1},${header.email2}"))
    .recipientList(header("recipients"))
    .delimiter(",")
    .parallelProcessing();  // Optional: process recipients in parallel
```

### 3.7 Multicast Pattern

Send the same message to multiple endpoints in parallel:

```java
from("direct:broadcast")
    .multicast()
        .parallelProcessing()
        .timeout(30000)  // 30 second timeout
        .aggregationStrategy(new ArrayListAggregationStrategy())
    .to("kafka:topic1", "kafka:topic2", "log:multicast");
```

### 3.8 Broadcast With Error Handling

```java
from("direct:broadcast-safe")
    .multicast()
        .parallelProcessing()
        .stopOnException(false)  // Continue even if one fails
    .to("kafka:topic1", "kafka:topic2", "log:broadcast");
```

### 3.9 Chapter Exercise: Complex Routing Logic

**Exercise 3.1**: Build a content-based routing system:

```java
@Component
public class OrderRouter extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("kafka:incoming-orders")
            .to("direct:route");
        
        from("direct:route")
            .choice()
                .when(jsonpath("$.type == 'DIGITAL'"))
                    .to("direct:digital-fulfillment")
                .when(jsonpath("$.type == 'PHYSICAL'"))
                    .to("direct:physical-fulfillment")
                .when(jsonpath("$.type == 'SERVICE'"))
                    .to("direct:service-fulfillment")
                .otherwise()
                    .to("direct:unknown-type")
            .end();
        
        from("direct:digital-fulfillment")
            .log("Digital order")
            .to("kafka:digital-orders");
        
        from("direct:physical-fulfillment")
            .log("Physical order")
            .to("kafka:physical-orders");
        
        from("direct:service-fulfillment")
            .log("Service order")
            .to("kafka:service-orders");
        
        from("direct:unknown-type")
            .log("ERROR: Unknown order type")
            .to("kafka:error-orders");
    }
}
```

**Exercise 3.2**: Implement multicast with aggregation:

```java
from("direct:notify-all")
    .multicast(new ArrayListAggregationStrategy())
        .timeout(5000)
    .to("kafka:email-notifications", 
        "kafka:sms-notifications", 
        "kafka:push-notifications")
    .log("Notifications aggregated: ${body}");
```

---

# PART 2: INTEGRATION PATTERNS

## Chapter 4: REST API Integration

### 4.1 REST DSL Overview

Camel's REST DSL provides declarative REST endpoint definition. It abstracts away HTTP details and generates OpenAPI specifications automatically.

```java
@Component
public class OrderRestApi extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        restConfiguration()
            .component("jetty")
            .host("0.0.0.0")
            .port(8080)
            .apiContextPath("api-doc");
        
        rest("/orders")
            .get("/{id}")
                .description("Get order by ID")
                .param()
                    .name("id")
                    .type(RestParamType.path)
                    .description("Order ID")
                    .dataType("string")
                    .required(true)
                .endParam()
                .responseMessage()
                    .code(200)
                    .message("Order found")
                .endResponseMessage()
                .responseMessage()
                    .code(404)
                    .message("Order not found")
                .endResponseMessage()
                .to("direct:getOrder")
            
            .post()
                .description("Create new order")
                .type(Order.class)
                .to("direct:createOrder");
        
        from("direct:getOrder")
            .log("Fetching order ${header.id}")
            .bean(orderService, "getOrderById(${header.id})");
        
        from("direct:createOrder")
            .log("Creating order: ${body}")
            .bean(orderService, "createOrder(${body})")
            .marshal().json(JsonLibrary.Jackson);
    }
}
```

### 4.2 REST Configuration

Configure the REST transport:

```java
restConfiguration()
    .component("jetty")  // HTTP component (jetty, servlet, netty, undertow)
    .host("localhost")
    .port(8080)
    .contextPath("/api")
    .apiContextPath("/api-docs")
    .apiContextRouteId("api-doc")
    .apiProperty("api.title", "Order API")
    .apiProperty("api.version", "1.0.0")
    .apiProperty("api.description", "Order management system")
    .apiProperty("api.contact.name", "API Support")
    .apiProperty("api.license.name", "Apache 2.0")
    .corsHeaders()
        .header("Access-Control-Allow-Origin", "*")
        .header("Access-Control-Allow-Methods", "GET,POST,PUT,DELETE");
```

### 4.3 HTTP Producer (Calling External APIs)

Consume external REST APIs using the HTTP component:

```java
@Component
public class ExternalApiConsumer extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("direct:call-external-api")
            .setHeader(Exchange.HTTP_METHOD, constant("GET"))
            .setHeader("Authorization", simple("Bearer ${header.token}"))
            .to("https://api.example.com/users/${header.userId}")
            .log("Response: ${body}")
            .unmarshal().json(JsonLibrary.Jackson, User.class);
        
        // POST request
        from("direct:create-user")
            .setHeader(Exchange.HTTP_METHOD, constant("POST"))
            .setHeader(Exchange.CONTENT_TYPE, constant("application/json"))
            .to("https://api.example.com/users")
            .unmarshal().json(JsonLibrary.Jackson, User.class);
    }
}
```

### 4.4 Query Parameters & Path Parameters

```java
rest("/users")
    .get("/{userId}/orders")
        .param()
            .name("userId")
            .type(RestParamType.path)
            .required(true)
        .endParam()
        .param()
            .name("status")
            .type(RestParamType.query)
            .defaultValue("ALL")
            .description("Filter by status")
        .endParam()
        .param()
            .name("limit")
            .type(RestParamType.query)
            .dataType("integer")
            .defaultValue("10")
        .endParam()
        .to("direct:getUserOrders")
    .post("/{userId}/preferences")
        .type(UserPreferences.class)
        .to("direct:updateUserPreferences");

from("direct:getUserOrders")
    .log("User: ${header.userId}, Status: ${header.status}, Limit: ${header.limit}")
    .bean(userService, "getOrders(${header.userId}, ${header.status}, ${header.limit})");
```

### 4.5 Request/Response Marshalling

```java
rest("/orders")
    .get("/{id}")
        // Response is automatically marshalled to JSON
        .to("direct:getOrder")
    
    .post()
        .consumes("application/json")
        .produces("application/json")
        .type(OrderRequest.class)  // Expect OrderRequest in request body
        .to("direct:createOrder");

from("direct:getOrder")
    .bean(orderService, "getById(${header.id})");
    // Output is automatically marshalled to JSON

from("direct:createOrder")
    .bean(orderService, "create(${body})")
    // Result is automatically marshalled to JSON
    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(201));
```

### 4.6 Error Handling in REST Routes

```java
rest("/orders")
    .post()
        .type(Order.class)
        .to("direct:processOrder");

from("direct:processOrder")
    .doTry()
        .bean(orderService, "validate(${body})")
        .bean(orderService, "process(${body})")
    .doCatch(ValidationException.class)
        .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(400))
        .setBody(simple("{\"error\":\"Invalid order: ${exception.message}\"}"))
    .doCatch(Exception.class)
        .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(500))
        .setBody(simple("{\"error\":\"Server error: ${exception.message}\"}"))
    .doFinally()
        .log("Order processing completed")
    .end();
```

### 4.7 Contract-First OpenAPI (Camel 4.6+)

Define REST endpoints using OpenAPI specification (contract-first):

**openapi.yaml**:
```yaml
openapi: 3.0.0
info:
  title: Order API
  version: 1.0.0
paths:
  /orders:
    get:
      operationId: listOrders
      responses:
        '200':
          description: List of orders
          content:
            application/json:
              schema:
                type: array
                items:
                  $ref: '#/components/schemas/Order'
    post:
      operationId: createOrder
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/Order'
      responses:
        '201':
          description: Order created
components:
  schemas:
    Order:
      type: object
      properties:
        id:
          type: string
        customerId:
          type: string
        amount:
          type: number
```

**Camel Route**:
```java
@Component
public class OpenApiRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        restConfiguration()
            .component("platform-http")
            .apiContextPath("/api-doc");
        
        rest()
            .openApi()
            .specification("classpath:openapi.yaml");
        
        // Routes automatically created for each operation
        // Route names follow pattern: direct:{operationId}
        
        from("direct:listOrders")
            .bean(orderService, "listAll");
        
        from("direct:createOrder")
            .bean(orderService, "create(${body})");
    }
}
```

### 4.8 Chapter Exercise: Building a REST API

**Exercise 4.1**: Implement a product catalog REST API:

```java
@Component
public class ProductRestApi extends RouteBuilder {
    @Autowired
    private ProductService productService;
    
    @Override
    public void configure() throws Exception {
        restConfiguration()
            .component("jetty")
            .host("0.0.0.0")
            .port(8080)
            .apiContextPath("api-docs");
        
        rest("/api/v1/products")
            .get()
                .description("List all products")
                .param()
                    .name("category")
                    .type(RestParamType.query)
                    .required(false)
                .endParam()
                .responseMessage()
                    .code(200)
                    .message("Products list")
                .endResponseMessage()
                .to("direct:listProducts")
            
            .get("/{productId}")
                .description("Get product by ID")
                .param()
                    .name("productId")
                    .type(RestParamType.path)
                    .required(true)
                .endParam()
                .to("direct:getProduct")
            
            .post()
                .description("Create new product")
                .type(Product.class)
                .responseMessage()
                    .code(201)
                    .message("Product created")
                .endResponseMessage()
                .to("direct:createProduct")
            
            .put("/{productId}")
                .description("Update product")
                .param()
                    .name("productId")
                    .type(RestParamType.path)
                    .required(true)
                .endParam()
                .type(Product.class)
                .to("direct:updateProduct");
        
        from("direct:listProducts")
            .choice()
                .when(header("category").isNotNull())
                    .bean(productService, "getByCategory(${header.category})")
                .otherwise()
                    .bean(productService, "getAll")
            .end()
            .marshal().json(JsonLibrary.Jackson);
        
        from("direct:getProduct")
            .bean(productService, "getById(${header.productId})")
            .choice()
                .when(simple("${body} != null"))
                    .marshal().json(JsonLibrary.Jackson)
                .otherwise()
                    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(404))
                    .setBody(constant("{\"error\":\"Product not found\"}"))
            .end();
        
        from("direct:createProduct")
            .bean(productService, "create(${body})")
            .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(201))
            .marshal().json(JsonLibrary.Jackson);
        
        from("direct:updateProduct")
            .bean(productService, "update(${header.productId}, ${body})")
            .marshal().json(JsonLibrary.Jackson);
    }
}

// Supporting classes
@Service
public class ProductService {
    public List<Product> getAll() { /*...*/ }
    public List<Product> getByCategory(String category) { /*...*/ }
    public Product getById(String id) { /*...*/ }
    public Product create(Product product) { /*...*/ }
    public Product update(String id, Product product) { /*...*/ }
}

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Product {
    private String id;
    private String name;
    private String category;
    private BigDecimal price;
    private Integer stock;
}
```

**Exercise 4.2**: Add JWT authentication to the REST API (see Chapter 12).

---

## Chapter 5: File & SFTP Operations

### 5.1 File Component Basics

The file component provides local file system access:

```java
from("file:inbox")  // Consumer: polls directory
    .log("Processing file: ${header.CamelFileName}")
    .to("file:processed");  // Producer: writes file

from("timer:daily?period=86400000")  // Daily timer
    .setBody(simple("Report: ${date:now}"))
    .to("file:reports?fileName=report-${date:now:yyyyMMdd}.txt");
```

### 5.2 File Consumer Options

```java
from("file:inbox?" +
     "noop=true&" +              // Don't delete after processing
     "readLock=changed&" +        // Wait for file to stabilize
     "readLockTimeout=5000&" +    // 5 second timeout
     "maxMessagesPerPoll=10&" +   // Process 10 files per poll
     "delay=60000&" +             // Poll every 60 seconds
     "recursive=true&" +          // Include subdirectories
     "include=.*\\.csv$")         // Regex filter: only CSV files
    .log("Processing: ${header.CamelFileName}")
    .to("direct:process");
```

**Important Options**:
- `noop=true`: Don't delete processed files (read-only)
- `delete=false`: Don't delete (default is true)
- `readLock=changed`: Lock file until it stops changing (best for file drops)
- `readLock=idempotent`: Use idempotent consumer (best for large files)
- `recursive=true`: Process subdirectories
- `include=/filter/`: Regex to match filenames
- `exclude=/filter/`: Regex to exclude filenames

### 5.3 Idempotent Consumer (Prevent Duplicates)

```java
from("file:inbox?noop=true")
    .idempotentConsumer(header(Exchange.FILE_NAME))  // Prevent reprocessing
    .messageIdRepository(fileIdempotentRepository)
    .log("Processing: ${header.CamelFileName}")
    .to("direct:process");

// Or with built-in memory store (not for production)
from("file:inbox?noop=true")
    .idempotentConsumer(header(Exchange.FILE_NAME))
    .log("Processing: ${header.CamelFileName}");

// Or using filename + size combination
from("file:inbox?noop=true")
    .idempotentConsumer(
        simple("${header.CamelFileName}-${header.CamelFileLength}"))
    .log("Processing: ${header.CamelFileName}");
```

### 5.4 File Producer Options

```java
from("direct:save")
    .to("file:output?" +
        "fileName=${header.orderId}-${date:now:yyyyMMdd}.json&" +
        "charset=UTF-8&" +
        "doneFileName=${header.orderId}.done&" +     // Mark completion
        "fileExist=Append");                         // Append mode

// Create subdirectories if missing
from("direct:archive")
    .to("file:archive/${date:now:yyyy/MM/dd}?" +
        "fileName=${header.CamelFileName}&" +
        "createMissingDirectories=true");
```

### 5.5 SFTP Component

SFTP (SSH File Transfer Protocol) for secure file operations:

```java
// SFTP Consumer - download files from remote server
from("sftp://user:password@sftp.example.com:22/inbox?" +
     "noop=true&" +
     "readLock=changed&" +
     "recursive=true&" +
     "knownHostsFile=classpath:known_hosts&" +
     "strictHostChecking=yes")
    .log("Downloaded: ${header.CamelFileName}")
    .to("file:inbox");

// SFTP Producer - upload files to remote server
from("file:outbox")
    .to("sftp://user:password@sftp.example.com:22/upload?" +
        "fileName=${header.CamelFileName}");
```

### 5.6 SFTP Authentication Methods

#### Password-Based Authentication

```java
from("sftp://user:password@sftp.example.com:22/files")
    .to("log:sftp");
```

#### Private Key Authentication

```java
from("sftp://user@sftp.example.com:22/files?" +
     "privateKeyFile=/path/to/id_rsa&" +
     "privateKeyFilePassphrase=passphrase&" +
     "knownHostsFile=~/.ssh/known_hosts&" +
     "strictHostChecking=yes")
    .log("SFTP with key auth")
    .to("file:inbox");
```

#### Configuration via Spring Properties

```properties
# application.properties
camel.component.sftp.username=${SFTP_USER}
camel.component.sftp.password=${SFTP_PASSWORD}
camel.component.sftp.known-hosts-file=${SFTP_KNOWN_HOSTS}
camel.component.sftp.strict-host-checking=true
```

### 5.7 SFTP File Operations Patterns

```java
@Component
public class SftpFileProcessor extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        // Download and process
        from("sftp://user@sftp.example.com/inbox?" +
             "privateKeyFile=classpath:id_rsa&" +
             "noop=false&" +               // Delete after processing
             "readLock=changed&" +
             "readLockTimeout=10000")
            .log("Received: ${header.CamelFileName}")
            .to("direct:validate");
        
        from("direct:validate")
            .choice()
                .when(simple("${header.CamelFileName} ends with '.csv'"))
                    .to("direct:process-csv")
                .when(simple("${header.CamelFileName} ends with '.json'"))
                    .to("direct:process-json")
                .otherwise()
                    .to("sftp://user@sftp.example.com/rejected")
            .end();
        
        // Upload processed files
        from("direct:save-result")
            .to("sftp://user@sftp.example.com/processed?" +
                "fileName=${header.originalName}-${date:now:yyyyMMdd}.result");
        
        from("direct:process-csv")
            .log("Processing CSV")
            .to("direct:save-result");
        
        from("direct:process-json")
            .log("Processing JSON")
            .to("direct:save-result");
    }
}
```

### 5.8 Handling Large Files

For large files, use streaming to avoid loading entire file in memory:

```java
from("file:inbox?" +
     "noop=true&" +
     "readLock=idempotent")
    .streamCaching()  // Enable streaming cache for large files
    .log("Processing: ${header.CamelFileName}")
    .split(body().tokenize("\n"))  // Split by newlines
        .streaming()               // Stream mode for split
        .to("direct:process-line");

from("direct:process-line")
    .log("Line: ${body}");
```

### 5.9 Chapter Exercise: File Integration System

**Exercise 5.1**: Build a multi-protocol file synchronization system:

```java
@Component
public class FileSyncRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        // Watch local inbox
        from("file:inbox?noop=false&readLock=changed&recursive=true")
            .setProperty("originalPath", header(Exchange.FILE_NAME))
            .log("Local file detected: ${header.CamelFileName}")
            .to("direct:sync");
        
        // Sync to SFTP
        from("direct:sync")
            .choice()
                .when(simple("${header.CamelFileName} ends with '.zip'"))
                    .log("Uploading compressed file")
                    .to("sftp://user@sftp.example.com/uploads/" +
                        "?privateKeyFile=classpath:id_rsa&" +
                        "fileName=${header.CamelFileName}")
                    .log("Upload complete: ${header.CamelFileName}")
                    .to("direct:archive-local")
                .when(simple("${header.CamelFileName} ends with '.csv'"))
                    .log("Processing CSV for upload")
                    .bean(csvProcessor, "validate")
                    .to("sftp://user@sftp.example.com/csv-uploads/")
                    .to("direct:archive-local")
                .otherwise()
                    .log("Unsupported file type")
                    .to("file:rejected?fileName=${header.CamelFileName}")
            .end();
        
        from("direct:archive-local")
            .setHeader("timestamp", simple("${date:now:yyyyMMdd_HHmmss}"))
            .to("file:archive/${date:now:yyyy/MM/dd}" +
                "?fileName=${header.CamelFileName}-${header.timestamp}")
            .log("Archived: ${header.CamelFileName}");
        
        // Periodic sync from SFTP
        from("timer:sync-remote?period=3600000")  // Every hour
            .to("direct:fetch-remote");
        
        from("direct:fetch-remote")
            .log("Syncing from SFTP")
            .from("sftp://user@sftp.example.com/outbox?" +
                  "privateKeyFile=classpath:id_rsa&" +
                  "noop=false&" +
                  "readLock=changed")
            .to("file:received/incoming");
    }
}

@Service
public class CsvProcessor {
    public void validate(Exchange exchange) {
        String filename = exchange.getIn().getHeader(Exchange.FILE_NAME, String.class);
        log.info("Validating CSV: {}", filename);
    }
}
```

**Exercise 5.2**: Implement error handling and retry logic for SFTP transfers:

```java
from("file:outbox")
    .errorHandler(deadLetterChannel("file:error-queue")
        .maximumRedeliveries(3)
        .redeliveryDelay(5000)
        .exponentialBackOff())
    .doTry()
        .to("sftp://user@sftp.example.com/uploads?" +
            "privateKeyFile=classpath:id_rsa")
    .doCatch(SftpOperationFailedException.class)
        .log("SFTP error: ${exception.message}")
        .setHeader("error-reason", simple("${exception.message}"))
    .doFinally()
        .log("Transfer attempt completed")
    .end();
```

---

## Chapter 6: Content-Based Routing

### 6.1 The Choice-When-Otherwise Pattern

Content-Based Routing (CBR) directs messages to different destinations based on message content:

```java
from("kafka:orders")
    .choice()
        .when(simple("${body.type} == 'DIGITAL'"))
            .log("Digital order")
            .to("direct:digital-fulfillment")
        .when(simple("${body.type} == 'PHYSICAL'"))
            .log("Physical order")
            .to("direct:physical-fulfillment")
        .when(jsonpath("$.amount > 1000"))
            .log("High-value order")
            .to("direct:vip-processing")
        .otherwise()
            .log("Standard order")
            .to("direct:standard-processing")
    .end();
```

### 6.2 Complex Predicates

Combine multiple conditions:

```java
from("kafka:transactions")
    .choice()
        .when(
            simple("${header.type} == 'CC'").and(
            simple("${body.amount} > 5000")).and(
            simple("${body.status} == 'PENDING'}")))
            .log("High-value CC transaction - needs verification")
            .to("direct:verify-cc-transaction")
        .when(
            simple("${body.amount} > 10000"))
            .log("Amount > $10K - escalate to supervisor")
            .to("direct:escalate")
        .otherwise()
            .to("direct:process-standard")
    .end();
```

### 6.3 XPath for XML Content

```java
from("file:inbox")
    .choice()
        .when(xpath("/order/customer/@vip = 'true'"))
            .log("VIP customer order")
            .to("direct:vip-orders")
        .when(xpath("/order/items/item[price > 1000]"))
            .log("Order contains expensive items")
            .to("direct:high-value-items")
        .when(xpath("count(/order/items/item) > 10"))
            .log("Bulk order (>10 items)")
            .to("direct:bulk-orders")
        .otherwise()
            .to("direct:standard-orders")
    .end();
```

### 6.4 JSONPath for JSON Content

```java
from("kafka:json-messages")
    .choice()
        .when(jsonpath("$.priority == 'URGENT'"))
            .log("URGENT message")
            .to("direct:urgent")
        .when(jsonpath("$.type == 'PAYMENT' && $.amount > 5000"))
            .log("Large payment")
            .to("direct:large-payment")
        .when(jsonpath("$.items[?(@.quantity > 100)]"))
            .log("Bulk items in order")
            .to("direct:bulk-items")
        .when(jsonpath("$.metadata.environment == 'PROD'"))
            .log("Production message")
            .to("direct:production")
        .otherwise()
            .to("direct:default")
    .end();
```

### 6.5 Dynamic Routing

Route to dynamic destinations:

```java
from("kafka:orders")
    .setHeader("destination",
        simple("${header.type == 'PHYSICAL' ? 'kafka:warehouse-orders' : 'kafka:digital-orders'}"))
    .to(header("destination"));  // Route to dynamic endpoint

// Or using bean
from("kafka:orders")
    .bean(routingService, "determineDestination")
    .log("Routing to: ${header.destination}")
    .to(header("destination"));

// Service
@Service
public class RoutingService {
    public void determineDestination(Exchange exchange) {
        String type = exchange.getIn().getHeader("type", String.class);
        String destination = type.equals("PHYSICAL") 
            ? "kafka:warehouse-orders" 
            : "kafka:digital-orders";
        exchange.getIn().setHeader("destination", destination);
    }
}
```

### 6.6 Filtering

Filter messages before processing:

```java
from("kafka:events")
    .filter(simple("${body.status} == 'ACTIVE'"))
    .filter(simple("${header.retryCount} < 3"))
    .filter(jsonpath("$.amount > 100"))
    .log("All filters passed")
    .to("direct:process");

// Inverse: filter to exclude
from("kafka:events")
    .filter(simple("${body.type} != 'SPAM'").not())
    .log("Non-spam message");
```

### 6.7 Chapter Exercise: Sophisticated Content Routing

**Exercise 6.1**: Build a complex order routing system:

```java
@Component
public class OrderRoutingEngine extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("kafka:orders")
            .to("direct:validate-order")
            .to("direct:route-order");
        
        from("direct:validate-order")
            .choice()
                .when(jsonpath("$.customerId == null"))
                    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(400))
                    .setBody(constant("{\"error\":\"Missing customerId\"}"))
                    .stop()
                .when(jsonpath("$.items.length() == 0"))
                    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(400))
                    .setBody(constant("{\"error\":\"No items in order\"}"))
                    .stop()
                .when(jsonpath("$.totalAmount <= 0"))
                    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(400))
                    .setBody(constant("{\"error\":\"Invalid amount\"}"))
                    .stop()
            .end()
            .log("Order validated: ${header.orderId}");
        
        from("direct:route-order")
            .choice()
                // VIP customers - priority processing
                .when(jsonpath("$.customerTier == 'PLATINUM'"))
                    .setHeader("routing", constant("vip"))
                    .to("direct:vip-processing")
                // Bulk orders
                .when(jsonpath("$.items.length() > 50"))
                    .setHeader("routing", constant("bulk"))
                    .to("direct:bulk-processing")
                // Digital only
                .when(jsonpath("$.items[0].type == 'DIGITAL'"))
                    .setHeader("routing", constant("digital"))
                    .to("direct:digital-fulfillment")
                // International
                .when(jsonpath("$.shippingCountry != 'US'"))
                    .setHeader("routing", constant("international"))
                    .to("direct:international-fulfillment")
                // Standard
                .otherwise()
                    .setHeader("routing", constant("standard"))
                    .to("direct:standard-fulfillment")
            .end();
        
        from("direct:vip-processing")
            .log("VIP processing for customer ${body.customerId}")
            .to("kafka:vip-orders");
        
        from("direct:bulk-processing")
            .log("Bulk processing: ${body.items.length()} items")
            .to("kafka:bulk-orders");
        
        from("direct:digital-fulfillment")
            .log("Digital fulfillment")
            .to("kafka:digital-orders");
        
        from("direct:international-fulfillment")
            .log("International fulfillment to ${body.shippingCountry}")
            .to("kafka:international-orders");
        
        from("direct:standard-fulfillment")
            .log("Standard fulfillment")
            .to("kafka:standard-orders");
    }
}
```

---

## Chapter 7: Error Handling & Resilience

### 7.1 Dead Letter Channel Pattern

The Dead Letter Channel is Camel's primary error handling mechanism for routes:

```java
from("kafka:orders")
    .errorHandler(deadLetterChannel("kafka:orders-error")
        .maximumRedeliveries(3)              // Retry up to 3 times
        .redeliveryDelay(1000)               // Wait 1 second between retries
        .backOffMultiplier(2.0)              // Double wait each retry (1s, 2s, 4s)
        .useExponentialBackOff()             // Enable exponential backoff
        .useCollisionAvoidance()             // Add randomness to avoid thundering herd
        .logExhaustedMessageHistory(true)    // Log full message history
        .logStackTrace(true))                // Include stack traces
    .log("Processing order")
    .bean(orderService, "process(${body})")
    .log("Order processed");
```

**Execution Flow**:
1. First attempt to process message
2. If exception: wait, then retry (up to maximumRedeliveries)
3. If still failing after all retries: send to dead letter queue
4. Continue processing next message

### 7.2 Redelivery Policy Configuration

Fine-grained control over retries:

```java
errorHandler(deadLetterChannel("kafka:errors")
    .maximumRedeliveries(5)                 // Attempt up to 5 times
    .redeliveryDelay(5000)                  // Initial 5 second delay
    .backOffMultiplier(1.5)                 // Increase by 1.5x
    .useExponentialBackOff()                // 5s, 7.5s, 11.25s, 16.87s, 25.3s
    .useCollisionAvoidance()                // Add ±10% random variance
    .maximumRedeliveryDelay(60000)          // Cap at 60 seconds
    .retryAttemptedLogLevel(LoggingLevel.WARN)  // Log level for retries
    .logStackTrace(true)
    .logExhaustedMessageHistory(true)
    .logNewException(true));
```

**Redelivery Timing**:
- Initial delay: 5000ms
- Attempt 2: 5000 * 1.5 = 7500ms
- Attempt 3: 7500 * 1.5 = 11250ms
- Attempt 4: 11250 * 1.5 = 16875ms (capped at 60000ms)
- Attempt 5: 16875 * 1.5 = 25312ms

### 7.3 Retry Forever

For some scenarios, retry forever until success:

```java
from("direct:critical-operation")
    .errorHandler(deadLetterChannel("direct:notify-admin")
        .maximumRedeliveries(-1)           // -1 = retry forever
        .redeliveryDelay(10000)            // Every 10 seconds
        .logExhaustedMessageHistory(false)) // Don't log (would log forever)
    .bean(criticalService, "execute(${body})");

from("direct:notify-admin")
    .log("CRITICAL: Operation still failing after hours of retries")
    .to("kafka:admin-alerts");
```

### 7.4 Per-Exception Error Handling

Different handling for different exception types:

```java
@Component
public class SelectiveErrorHandling extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        onException(ValidationException.class)
            .maximumRedeliveries(0)        // No retry for validation errors
            .handled(true)
            .log(LoggingLevel.WARN, "Validation error: ${exception.message}")
            .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(400))
            .setBody(simple("{\"error\":\"${exception.message}\"}"));
        
        onException(DatabaseException.class)
            .maximumRedeliveries(3)        // Retry DB errors
            .redeliveryDelay(2000)
            .handled(true)
            .log(LoggingLevel.ERROR, "Database error: ${exception.message}");
        
        onException(ExternalServiceException.class)
            .maximumRedeliveries(5)        // Aggressive retry for external services
            .redeliveryDelay(5000)
            .exponentialBackOff()
            .handled(true)
            .log(LoggingLevel.ERROR, "External service error: ${exception.message}");
        
        onException(Exception.class)       // Catch-all
            .maximumRedeliveries(1)
            .handled(true)
            .log(LoggingLevel.ERROR, "Unexpected error: ${exception.message}")
            .to("kafka:dead-letter-queue");
        
        from("kafka:orders")
            .log("Processing: ${body}")
            .bean(orderService, "validate")
            .bean(orderService, "process")
            .log("Success");
    }
}

// Custom exceptions
public class ValidationException extends Exception {}
public class DatabaseException extends Exception {}
public class ExternalServiceException extends Exception {}
```

### 7.5 doTry-doCatch-doFinally

Route-level exception handling:

```java
from("kafka:orders")
    .doTry()
        .log("Starting order processing")
        .bean(orderService, "validate(${body})")
        .bean(orderService, "process(${body})")
        .log("Order processed successfully")
    .doCatch(ValidationException.class)
        .log(LoggingLevel.WARN, "Validation failed: ${exception.message}")
        .setHeader("error-code", constant("VALIDATION_ERROR"))
        .to("kafka:invalid-orders")
    .doCatch(ProcessingException.class)
        .log(LoggingLevel.ERROR, "Processing failed: ${exception.message}")
        .setHeader("error-code", constant("PROCESSING_ERROR"))
        .to("kafka:processing-errors")
    .doCatch(Exception.class)
        .log(LoggingLevel.ERROR, "Unexpected error: ${exception.message}")
        .setHeader("error-code", constant("UNKNOWN_ERROR"))
        .to("kafka:unknown-errors")
    .doFinally()
        .log("Order processing completed (success or failure)")
    .end();
```

**doFinally is Always Executed**: Whether successful or exception caught.

### 7.6 Circuit Breaker Pattern

Prevent cascading failures:

```java
from("kafka:orders")
    .circuitBreaker()
    .to("http://slow-api.example.com/orders")
    .onFallback()
        .log("Circuit breaker activated - using fallback")
        .to("direct:fallback-processing");

from("direct:fallback-processing")
    .log("Processing with degraded service")
    .setHeader("fallback", constant("true"))
    .to("kafka:queued-orders");
```

### 7.7 Timeout Handling

Prevent hanging routes:

```java
from("kafka:orders")
    .doTry()
        .timeout(30000)  // 30 second timeout
        .to("http://api.example.com/process?connectTimeout=5000&socketTimeout=25000")
    .doCatch(TimeoutException.class)
        .log("Request timed out after 30 seconds")
        .to("kafka:timeout-orders")
    .doFinally()
        .log("Request completed or timed out")
    .end();
```

### 7.8 Chapter Exercise: Production-Grade Error Handling

**Exercise 7.1**: Implement comprehensive error handling:

```java
@Component
public class ProductionErrorHandling extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        // Global exception handlers
        onException(PaymentGatewayException.class)
            .maximumRedeliveries(5)
            .exponentialBackOff()
            .redeliveryDelay(3000)
            .handled(true)
            .log(LoggingLevel.ERROR, "Payment gateway error: ${exception.message}")
            .setHeader("retry-count", simple("${header[org.apache.camel.Redelivered] ? ${header[org.apache.camel.redeliveryCount]} : 0}"));
        
        onException(InventoryException.class)
            .maximumRedeliveries(2)
            .redeliveryDelay(1000)
            .handled(true)
            .log(LoggingLevel.WARN, "Inventory error: ${exception.message}");
        
        onException(Exception.class)
            .maximumRedeliveries(1)
            .handled(true)
            .log(LoggingLevel.ERROR, "Unhandled exception: ${exception.message}")
            .setHeader("error-timestamp", simple("${date:now}"))
            .to("kafka:critical-errors");
        
        // Main order processing route
        from("kafka:orders")
            .routeId("order-processing")
            .log("Order received: ${body.orderId}")
            .to("direct:process-order");
        
        from("direct:process-order")
            .doTry()
                .setProperty("startTime", simple("${date:now}"))
                .to("direct:validate")
                .to("direct:process-payment")
                .to("direct:update-inventory")
                .to("direct:notify-customer")
                .setHeader("status", constant("SUCCESS"))
            .doCatch(PaymentGatewayException.class)
                .log("Payment failed, retrying...")
                .rethrow()
            .doCatch(InventoryException.class)
                .log("Inventory unavailable")
                .setHeader("status", constant("INVENTORY_ERROR"))
                .to("kafka:out-of-stock-orders")
            .doCatch(Exception.class)
                .log("Unexpected error in order processing")
                .setHeader("status", constant("ERROR"))
                .to("kafka:processing-errors")
            .doFinally()
                .setProperty("endTime", simple("${date:now}"))
                .log("Order processing completed with status: ${header.status}")
            .end();
        
        from("direct:validate")
            .log("Validating order")
            .bean(orderService, "validate");
        
        from("direct:process-payment")
            .timeout(15000)
            .log("Processing payment")
            .bean(paymentService, "process");
        
        from("direct:update-inventory")
            .log("Updating inventory")
            .bean(inventoryService, "reserve");
        
        from("direct:notify-customer")
            .log("Notifying customer")
            .bean(notificationService, "sendConfirmation");
    }
}

// Custom exceptions
public class PaymentGatewayException extends Exception {}
public class InventoryException extends Exception {}
```

---

# PART 3: ADVANCED PATTERNS

## Chapter 8: Splitters & Aggregators

### 8.1 Splitter Pattern

Break a single message into multiple sub-messages for parallel processing:

```java
from("kafka:bulk-orders")
    .log("Order received with ${body.items.size()} items")
    .split(jsonpath("$.items[*]"))  // Split array into individual items
        .log("Processing item: ${body.sku}")
        .to("direct:process-item")
    .end();

from("direct:process-item")
    .log("Item processing: ${body.sku}");
```

### 8.2 Splitter with Aggregation

Split, process in parallel, then reassemble:

```java
from("kafka:large-orders")
    .split(jsonpath("$.lineItems[*]"), new OrderLineAggregationStrategy())
        .log("Processing line item: ${body.itemId}")
        .bean(inventoryService, "reserve(${body})")
        .setBody(simple("${body.itemId}"))
    .aggregationStrategy(new OrderLineAggregationStrategy())
    .completionSize(simple("${exchangeProperty.CamelSplitSize}"))
    .log("Order fully processed: ${body}")
    .to("kafka:fulfilled-orders");

// Aggregation strategy
public class OrderLineAggregationStrategy implements AggregationStrategy {
    @Override
    public Exchange aggregate(Exchange oldExchange, Exchange newExchange) {
        if (oldExchange == null) {
            return newExchange;
        }
        
        List<String> items = oldExchange.getIn().getBody(List.class);
        if (items == null) {
            items = new ArrayList<>();
        }
        
        String newItem = newExchange.getIn().getBody(String.class);
        items.add(newItem);
        
        oldExchange.getIn().setBody(items);
        return oldExchange;
    }
}
```

### 8.3 Splitter Options

```java
from("kafka:large-files")
    .split(simple("${body}"))
        .streaming()                    // Stream mode (memory efficient)
        .parallelProcessing()           // Process items concurrently
        .executorService(executor)      // Custom thread pool
        .timeout(30000)                 // 30 second timeout
        .stopOnException(true)          // Stop if item fails
    .log("Processed: ${body}");
```

### 8.4 Batch Aggregation

Aggregate messages into batches:

```java
from("kafka:transactions")
    .aggregate(constant(true), new ListAggregationStrategy())
        .completionSize(100)                // Batch size
        .completionTimeout(60000)           // Or timeout after 60 seconds
    .log("Processing batch of ${body.size()} transactions")
    .bean(batchProcessor, "processBatch(${body})")
    .to("kafka:processed-transactions");

// Simple list aggregation strategy
public class ListAggregationStrategy implements AggregationStrategy {
    @Override
    public Exchange aggregate(Exchange oldExchange, Exchange newExchange) {
        List<Object> list = (oldExchange == null) 
            ? new ArrayList<>() 
            : oldExchange.getIn().getBody(List.class);
        
        list.add(newExchange.getIn().getBody());
        oldExchange = oldExchange == null ? newExchange : oldExchange;
        oldExchange.getIn().setBody(list);
        return oldExchange;
    }
}
```

### 8.5 Correlation-Based Aggregation

Aggregate related messages using correlation ID:

```java
from("kafka:order-events")
    .aggregate(header("orderId"), new EventAggregationStrategy())
        .completionTimeout(300000)  // 5 minutes
        .completionPredicate(
            simple("${exchangeProperty.CamelAggregatedSize} == ${exchangeProperty.CamelAggregatedCompletedSize}"))
    .log("Order ${header.orderId} fully processed")
    .bean(orderService, "finalize(${body})")
    .to("kafka:order-complete");

// Aggregation that waits for all expected events
public class EventAggregationStrategy implements AggregationStrategy {
    @Override
    public Exchange aggregate(Exchange oldExchange, Exchange newExchange) {
        List<OrderEvent> events = (oldExchange == null)
            ? new ArrayList<>()
            : oldExchange.getIn().getBody(List.class);
        
        events.add(newExchange.getIn().getBody(OrderEvent.class));
        
        Exchange result = oldExchange == null ? newExchange : oldExchange;
        result.getIn().setBody(events);
        
        // Check if we have all expected events
        int received = events.size();
        int expected = result.getIn().getHeader("expectedEvents", Integer.class, 4);
        
        if (received >= expected) {
            result.setProperty(Exchange.AGGREGATION_COMPLETE_CURRENT_GROUP, true);
        }
        
        return result;
    }
}
```

### 8.6 Chapter Exercise: Complex Splitting & Aggregation

**Exercise 8.1**: Implement order splitting and fulfillment:

```java
@Component
public class OrderFulfillmentEngine extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("kafka:orders")
            .log("Received order: ${body.orderId} with ${body.items.size()} items")
            .to("direct:split-order");
        
        from("direct:split-order")
            .setHeader("originalOrderId", simple("${body.orderId}"))
            .split(jsonpath("$.items[*]"), new OrderItemAggregationStrategy())
                .streaming()
                .parallelProcessing(8)  // 8 threads
                .timeout(120000)
                .stopOnException(false)
                .log("Processing item: ${body.sku}")
                .choice()
                    .when(simple("${body.type} == 'DIGITAL'"))
                        .to("direct:digital-fulfillment")
                    .when(simple("${body.type} == 'PHYSICAL'"))
                        .to("direct:warehouse-fulfillment")
                    .when(simple("${body.type} == 'SERVICE'"))
                        .to("direct:service-fulfillment")
                .end()
            .aggregationStrategy(new OrderItemAggregationStrategy())
            .completionSize(simple("${exchangeProperty.CamelSplitSize}"))
            .log("Order ${header.originalOrderId} fully aggregated")
            .to("direct:consolidate-fulfillment");
        
        from("direct:digital-fulfillment")
            .log("Digital fulfillment: ${body.sku}")
            .bean(digitalService, "fulfill");
        
        from("direct:warehouse-fulfillment")
            .log("Warehouse fulfillment: ${body.sku}")
            .bean(warehouseService, "reserve");
        
        from("direct:service-fulfillment")
            .log("Service scheduling: ${body.sku}")
            .bean(serviceService, "schedule");
        
        from("direct:consolidate-fulfillment")
            .log("Consolidating fulfillment results")
            .bean(fulfillmentService, "consolidate(${body})")
            .to("kafka:shipments");
    }
}

public class OrderItemAggregationStrategy implements AggregationStrategy {
    @Override
    public Exchange aggregate(Exchange oldExchange, Exchange newExchange) {
        List<OrderItem> items = (oldExchange == null)
            ? new ArrayList<>()
            : oldExchange.getIn().getBody(List.class);
        
        OrderItem item = newExchange.getIn().getBody(OrderItem.class);
        item.setFulfillmentResult(newExchange.getIn().getHeader("fulfillmentResult", String.class));
        items.add(item);
        
        Exchange result = oldExchange == null ? newExchange : oldExchange;
        result.getIn().setBody(items);
        return result;
    }
}
```

---

## Chapter 9: Processors & Beans

### 9.1 Custom Processors

Implement `Processor` interface for complex transformations:

```java
public class EnrichmentProcessor implements Processor {
    private final CustomerService customerService;
    
    public EnrichmentProcessor(CustomerService customerService) {
        this.customerService = customerService;
    }
    
    @Override
    public void process(Exchange exchange) throws Exception {
        String customerId = exchange.getIn().getBody(String.class);
        Customer customer = customerService.getById(customerId);
        
        // Enrich exchange with customer data
        exchange.getIn().setHeader("customerName", customer.getName());
        exchange.getIn().setHeader("customerTier", customer.getTier());
        exchange.getIn().setBody(customer);  // Replace body with customer object
    }
}

// Usage in route
from("kafka:customer-ids")
    .process(new EnrichmentProcessor(customerService))
    .log("Customer: ${header.customerName} (${header.customerTier})")
    .to("direct:process-customer");
```

### 9.2 Bean Component

Call methods on Spring beans directly:

```java
@Component
public class OrderServiceRoute extends RouteBuilder {
    @Autowired
    private OrderService orderService;
    
    @Override
    public void configure() throws Exception {
        from("kafka:orders")
            // Call bean method, passing body as parameter
            .bean(orderService, "validate(${body})")
            
            // Call with multiple parameters
            .bean(orderService, "process(${body}, ${header.orderId})")
            
            // Call with body transformation
            .bean(orderService, "toDto(${body})")
            
            // Call without parameters, body becomes input
            .bean(orderService, "calculateTax")
            
            .to("kafka:processed-orders");
    }
}

@Service
public class OrderService {
    public void validate(Order order) throws ValidationException {
        if (order.getCustomerId() == null) {
            throw new ValidationException("Missing customerId");
        }
    }
    
    public Order process(Order order, String orderId) {
        order.setId(orderId);
        order.setProcessedAt(LocalDateTime.now());
        return order;
    }
    
    public OrderDTO toDto(Order order) {
        return new OrderDTO(order.getId(), order.getCustomerId());
    }
    
    public Order calculateTax(Order order) {
        order.setTax(order.getAmount() * 0.08);
        return order;
    }
}
```

### 9.3 Processor vs Bean

| Aspect | Processor | Bean |
|--------|-----------|------|
| Interface | `Processor` | Regular Java method |
| Spring Integration | Requires manual wiring | Automatic injection |
| Error Handling | Can wrap in try-catch | Propagates exceptions |
| Access to Exchange | Direct access | Via parameters |
| Complexity | Good for complex logic | Good for simple transformations |
| Testing | Requires CamelContext | Easy unit tests |

### 9.4 Transform & Set Body

Use fluent methods for transformations:

```java
from("kafka:raw-events")
    // Transform body
    .transform(body().append(" - processed"))
    
    // Using simple language
    .transform(simple("${body.toUpperCase()}"))
    
    // Using bean reference
    .transform().method(transformService, "toJson")
    
    // Using processor
    .process(new MyProcessor())
    
    // Using lambda (Java 8+)
    .transform(e -> {
        String input = e.getIn().getBody(String.class);
        return "Processed: " + input;
    })
    
    .log("${body}")
    .to("kafka:transformed-events");
```

### 9.5 Enrichment Pattern

Add data from external sources to messages:

```java
from("kafka:orders")
    // Enrich with customer data
    .enrich("direct:get-customer")
        .shareUnitOfWork()
    
    // Enrich with inventory data
    .enrich("direct:get-inventory")
        .shareUnitOfWork()
    
    .log("Order enriched with customer and inventory data")
    .to("direct:process-enriched");

from("direct:get-customer")
    .bean(customerService, "getById(${header.customerId})");

from("direct:get-inventory")
    .bean(inventoryService, "getAvailable(${body.sku})");

from("direct:process-enriched")
    .log("Processing enriched order")
    .bean(orderService, "process");
```

### 9.6 Chapter Exercise: Advanced Processing

**Exercise 9.1**: Implement complex bean-based processing pipeline:

```java
@Component
public class AdvancedProcessingRoute extends RouteBuilder {
    @Autowired
    private OrderValidator validator;
    
    @Autowired
    private OrderEnricher enricher;
    
    @Autowired
    private OrderProcessor processor;
    
    @Autowired
    private OrderNotifier notifier;
    
    @Override
    public void configure() throws Exception {
        from("kafka:orders")
            .routeId("advanced-processing")
            .log("Order: ${body.orderId}")
            .setProperty("receivedAt", simple("${date:now}"))
            
            // Validate
            .bean(validator, "validate(${body})")
            .log("Validation passed")
            
            // Enrich with external data
            .bean(enricher, "addCustomerData(${body})")
            .bean(enricher, "addInventoryData(${body})")
            .bean(enricher, "calculateShipping(${body})")
            .log("Enrichment complete")
            
            // Process
            .bean(processor, "createFulfillmentOrder(${body})")
            .log("Fulfillment order created")
            
            // Notify
            .bean(notifier, "notifyWarehouse(${body})")
            .bean(notifier, "notifyCustomer(${body})")
            
            .setHeader("processedAt", simple("${date:now}"))
            .to("kafka:orders-processed");
    }
}

@Service
public class OrderValidator {
    public void validate(Order order) {
        if (order.getCustomerId() == null) {
            throw new IllegalArgumentException("Missing customer ID");
        }
        if (order.getItems().isEmpty()) {
            throw new IllegalArgumentException("No items in order");
        }
    }
}

@Service
public class OrderEnricher {
    @Autowired
    private CustomerService customerService;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private ShippingService shippingService;
    
    public Order addCustomerData(Order order) {
        Customer customer = customerService.getById(order.getCustomerId());
        order.setCustomerEmail(customer.getEmail());
        order.setCustomerTier(customer.getTier());
        return order;
    }
    
    public Order addInventoryData(Order order) {
        order.getItems().forEach(item -> {
            int available = inventoryService.checkStock(item.getSku());
            item.setAvailable(available);
        });
        return order;
    }
    
    public Order calculateShipping(Order order) {
        BigDecimal shippingCost = shippingService.calculate(
            order.getShippingAddress(),
            order.getTotalWeight());
        order.setShippingCost(shippingCost);
        return order;
    }
}
```

---

## Chapter 10: Complex Transformations

### 10.1 JSON Transformations

```java
from("kafka:api-responses")
    // Parse JSON
    .unmarshal().json(JsonLibrary.Jackson, Map.class)
    
    // Transform structure
    .setBody(simple("{\n" +
        "  \"id\": ${body[id]},\n" +
        "  \"name\": ${body[user][name]},\n" +
        "  \"email\": ${body[user][email]}\n" +
        "}"))
    
    // Marshal back to JSON string
    .marshal().json(JsonLibrary.Jackson)
    
    .log("${body}")
    .to("kafka:transformed-responses");
```

### 10.2 XML Transformations

```java
from("file:xml-files")
    // Transform using XSLT
    .to("xslt:classpath:transform.xsl")
    
    // Or inline XPath transformation
    .setBody(xpath("/order/items/item[price > 1000]"))
    
    .log("${body}")
    .to("file:processed-xml");
```

### 10.3 Using Jackson for JSON

```java
from("kafka:json-messages")
    .unmarshal().json(JsonLibrary.Jackson, OrderRequest.class)
    .process(e -> {
        OrderRequest order = e.getIn().getBody(OrderRequest.class);
        OrderResponse response = new OrderResponse(
            order.getId(),
            order.getCustomerId(),
            order.getItems().size(),
            order.getTotal()
        );
        e.getIn().setBody(response);
    })
    .marshal().json(JsonLibrary.Jackson)
    .log("${body}")
    .to("kafka:responses");
```

### 10.4 Using Freemarker Templates

```java
from("direct:generate-report")
    .to("freemarker:classpath:report-template.ftl")
    .log("Report: ${body}")
    .to("file:reports?fileName=report-${date:now:yyyyMMdd}.html");
```

**report-template.ftl**:
```
<html>
  <body>
    <h1>Order Report</h1>
    <p>Order ID: ${exchangeProperty.orderId}</p>
    <p>Total Items: ${body.size()}</p>
    <ul>
    <#list body as item>
      <li>${item.sku} - $${item.price}</li>
    </#list>
    </ul>
  </body>
</html>
```

### 10.5 Data Mapping with JAXB (for XML)

```java
from("file:incoming-xml")
    .unmarshal().jaxb()  // Convert XML to POJO
    .setBody(simple("Order processed: ${body.orderId}"))
    .marshal().xml()     // Convert POJO back to XML
    .to("file:processed-xml");

// POJO
@XmlRootElement
@Data
public class Order {
    @XmlElement
    private String orderId;
    
    @XmlElement
    private BigDecimal total;
    
    @XmlElementWrapper(name = "items")
    @XmlElement(name = "item")
    private List<OrderItem> items;
}
```

---

# PART 4: PRODUCTION & OPERATIONS

## Chapter 11: Testing & Quality Assurance

### 11.1 Spring Boot Test Setup

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-spring-junit5</artifactId>
    <version>4.15.0</version>
    <scope>test</scope>
</dependency>

<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-test</artifactId>
    <version>${spring-boot.version}</version>
    <scope>test</scope>
</dependency>
```

### 11.2 Unit Testing with Mock Endpoints

```java
@CamelSpringBootTest
@SpringBootApplication
@MockEndpoints("kafka:*")  // Mock all Kafka endpoints
public class OrderRouteTest {
    
    @Autowired
    private ProducerTemplate template;
    
    @Autowired
    private CamelContext context;
    
    @EndpointInject("mock:kafka:orders")
    private MockEndpoint mockOrders;
    
    @Test
    public void testOrderProcessing() throws InterruptedException {
        // Arrange
        mockOrders.expectedMessageCount(1);
        mockOrders.expectedBodiesReceived("order-123");
        
        // Act
        template.sendBody("direct:process-order", "order-123");
        
        // Assert
        mockOrders.assertIsSatisfied();
    }
    
    @Test
    public void testOrderValidation() throws InterruptedException {
        mockOrders.expectedMessageCount(0);  // No message should reach kafka
        
        template.sendBody("direct:process-invalid", "invalid");
        
        mockOrders.assertIsSatisfied();
    }
}
```

### 11.3 Testing Exchanges & Headers

```java
@CamelSpringBootTest
public class ExchangeHeaderTest {
    
    @Autowired
    private ProducerTemplate template;
    
    @EndpointInject("mock:result")
    private MockEndpoint mock;
    
    @Test
    public void testHeaderTransformation() throws Exception {
        mock.expectedMessageCount(1);
        mock.expectedHeaderReceived("customerId", "CUST-123");
        mock.expectedHeaderReceived("processed", "true");
        
        template.sendBodyAndHeader("direct:enrich", 
            "customer data", 
            "customerId", "CUST-123");
        
        mock.assertIsSatisfied();
        
        Exchange exchange = mock.getReceivedExchanges().get(0);
        assertEquals("true", exchange.getIn().getHeader("processed"));
    }
}
```

### 11.4 Testing with Test Data

```java
@CamelSpringBootTest
public class ComplexRouteTest {
    
    @Autowired
    private ProducerTemplate template;
    
    @EndpointInject("mock:kafka:orders")
    private MockEndpoint mockOrders;
    
    @EndpointInject("mock:kafka:errors")
    private MockEndpoint mockErrors;
    
    @Test
    public void testBulkOrderProcessing() throws InterruptedException {
        mockOrders.expectedMessageCount(2);
        mockErrors.expectedMessageCount(1);
        
        // Send bulk order
        template.sendBody("kafka:incoming", createBulkOrder());
        
        MockEndpoint.assertIsSatisfied(
            3000,  // timeout
            TimeUnit.MILLISECONDS,
            mockOrders, mockErrors);
        
        // Verify processing
        assertEquals(2, mockOrders.getReceivedExchanges().size());
        assertEquals(1, mockErrors.getReceivedExchanges().size());
    }
}
```

### 11.5 Integration Testing

```java
@CamelSpringBootTest
@SpringBootTest
@TestContainers
public class FullIntegrationTest {
    
    @Container
    static KafkaContainer kafka = new KafkaContainer();
    
    @DynamicPropertySource
    static void kafkaProperties(DynamicPropertyRegistry registry) {
        registry.add("kafka.bootstrap.servers", 
            kafka::getBootstrapServers);
    }
    
    @Autowired
    private ProducerTemplate template;
    
    @Autowired
    private CamelContext context;
    
    @Test
    @Timeout(30)
    public void testEndToEndOrderProcessing() throws Exception {
        // Arrange
        Order order = createTestOrder();
        
        // Act
        template.sendBody("kafka:orders", order);
        
        // Wait for processing (route should save to database)
        Thread.sleep(5000);
        
        // Assert
        // Verify order was saved to database
        Order saved = orderRepository.findById(order.getId()).orElseThrow();
        assertEquals(OrderStatus.PROCESSED, saved.getStatus());
    }
}
```

### 11.6 Testing Error Handling

```java
@CamelSpringBootTest
public class ErrorHandlingTest {
    
    @Autowired
    private ProducerTemplate template;
    
    @EndpointInject("mock:kafka:dead-letter")
    private MockEndpoint dlq;
    
    @Test
    public void testDeadLetterRouting() throws InterruptedException {
        dlq.expectedMessageCount(1);
        
        // Send invalid message (will fail validation and go to DLQ)
        template.sendBody("direct:process", new InvalidOrder());
        
        dlq.assertIsSatisfied();
        assertEquals(1, dlq.getReceivedExchanges().size());
    }
    
    @Test
    public void testRetryLogic() throws InterruptedException {
        // Create a message that fails twice then succeeds
        MockEndpoint result = context.getEndpoint("mock:result", MockEndpoint.class);
        result.expectedMessageCount(1);
        
        template.sendBody("direct:retry-route", "test");
        
        result.assertIsSatisfied();
        // Verify it was retried (check headers or logs)
    }
}
```

### 11.7 Performance Testing

```java
@CamelSpringBootTest
public class PerformanceTest {
    
    @Autowired
    private ProducerTemplate template;
    
    @EndpointInject("mock:kafka:processed")
    private MockEndpoint result;
    
    @Test
    @Timeout(60)
    public void testThroughput() throws InterruptedException {
        int messageCount = 10000;
        result.expectedMessageCount(messageCount);
        result.setResultWaitTime(60000);
        
        long startTime = System.currentTimeMillis();
        
        // Send 10K messages
        for (int i = 0; i < messageCount; i++) {
            template.sendBody("kafka:orders", createOrder(i));
        }
        
        result.assertIsSatisfied();
        
        long duration = System.currentTimeMillis() - startTime;
        double throughput = (double) messageCount / (duration / 1000.0);
        
        System.out.println("Throughput: " + throughput + " msg/sec");
        assertTrue(throughput > 1000, "Should process > 1000 msg/sec");
    }
}
```

### 11.8 Chapter Exercise: Comprehensive Test Suite

[Exercise code would go here - similar structure to above examples]

---

## Chapter 12: Security & OAuth2/JWT

### 12.1 OAuth2 Authentication with Spring Security

```java
@Configuration
@EnableWebSecurity
public class SecurityConfig {
    
    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http
            .authorizeHttpRequests(authorize -> authorize
                .requestMatchers("/api/public/**").permitAll()
                .requestMatchers("/api/orders/**").hasScope("orders:read")
                .requestMatchers("/api/admin/**").hasRole("ADMIN")
                .anyRequest().authenticated())
            .oauth2ResourceServer(oauth2 -> oauth2
                .jwt(jwt -> jwt
                    .jwtAuthenticationConverter(jwtAuthenticationConverter())));
        
        return http.build();
    }
    
    @Bean
    public JwtAuthenticationConverter jwtAuthenticationConverter() {
        JwtAuthenticationConverter converter = new JwtAuthenticationConverter();
        JwtGrantedAuthoritiesConverter authoritiesConverter = new JwtGrantedAuthoritiesConverter();
        authoritiesConverter.setAuthoritiesClaimName("roles");
        authoritiesConverter.setAuthorityPrefix("ROLE_");
        converter.setJwtGrantedAuthoritiesConverter(authoritiesConverter);
        return converter;
    }
}
```

### 12.2 JWT Token Extraction in Camel Routes

```java
@Component
public class SecureOrderRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        restConfiguration()
            .component("jetty")
            .port(8080);
        
        rest("/api/v1/orders")
            .post()
                .to("direct:create-order");
        
        from("direct:create-order")
            // Extract JWT from Authorization header
            .setHeader("jwt", simple("${header.Authorization}"))
            
            // Validate JWT (can use custom processor)
            .process(new JwtValidationProcessor())
            
            // Extract user info from JWT claims
            .setHeader("userId", simple("${header.sub}"))  // subject claim
            .setHeader("userRoles", simple("${header.roles}"))
            
            // Log authenticated user
            .log("Order from user: ${header.userId}")
            
            // Check authorization
            .choice()
                .when(simple("${header.userRoles} contains 'ADMIN'"))
                    .log("Admin creating order")
                .when(simple("${header.userRoles} contains 'USER'"))
                    .log("User creating order")
                .otherwise()
                    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(403))
                    .setBody(constant("{\"error\":\"Forbidden\"}"))
                    .stop()
            .end()
            
            .bean(orderService, "create(${body}, ${header.userId})")
            .marshal().json(JsonLibrary.Jackson);
    }
}

// JWT Validation Processor
public class JwtValidationProcessor implements Processor {
    @Autowired
    private JwtDecoder jwtDecoder;
    
    @Override
    public void process(Exchange exchange) throws Exception {
        String authHeader = exchange.getIn().getHeader("Authorization", String.class);
        
        if (authHeader == null || !authHeader.startsWith("Bearer ")) {
            throw new UnauthorizedException("Missing or invalid Authorization header");
        }
        
        String token = authHeader.substring("Bearer ".length());
        
        try {
            Jwt jwt = jwtDecoder.decode(token);
            exchange.setProperty("jwt", jwt);
            exchange.getIn().setHeader("sub", jwt.getSubject());
            exchange.getIn().setHeader("roles", jwt.getClaimAsStringList("roles"));
        } catch (JwtException e) {
            throw new UnauthorizedException("Invalid JWT: " + e.getMessage());
        }
    }
}
```

### 12.3 HTTPS/SSL Configuration

```properties
# application.properties
server.ssl.key-store=classpath:keystore.p12
server.ssl.key-store-password=${SSL_PASSWORD}
server.ssl.key-store-type=PKCS12
server.ssl.key-alias=tomcat

camel.ssl.config.cert-alias=tomcat
camel.ssl.config.key-store=classpath:keystore.p12
camel.ssl.config.key-store-password=${SSL_PASSWORD}
```

### 12.4 API Key Authentication

```java
@Component
public class ApiKeyAuthRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        rest("/api/v1/products")
            .get()
                .to("direct:list-products");
        
        from("direct:list-products")
            // Validate API key
            .choice()
                .when(simple("${header.X-API-Key} == null"))
                    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(401))
                    .setBody(constant("{\"error\":\"Missing API key\"}"))
                    .stop()
                .when(simple("${header.X-API-Key} == '${properties:api.key}'"))
                    .log("Valid API key")
                .otherwise()
                    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(403))
                    .setBody(constant("{\"error\":\"Invalid API key\"}"))
                    .stop()
            .end()
            
            .bean(productService, "listAll")
            .marshal().json(JsonLibrary.Jackson);
    }
}
```

### 12.5 Chapter Exercise: Secure Microservice Integration

[Exercise code for comprehensive security implementation]

---

## Chapter 13: Observability & Monitoring

### 13.1 Health Checks

```java
@Configuration
public class HealthCheckConfig {
    
    @Bean
    public HealthCheckRegistry healthCheckRegistry() {
        return new DefaultHealthCheckRegistry();
    }
}

// application.properties
management.endpoint.health.enabled=true
management.endpoint.health.probes.enabled=true
management.health.livenessState.enabled=true
management.health.readinessState.enabled=true
camel.health.includeRoutes=true
camel.health.includeConsumers=true
```

Check health via HTTP:
```bash
curl http://localhost:8080/actuator/health
curl http://localhost:8080/actuator/health/live
curl http://localhost:8080/actuator/health/ready
```

### 13.2 Metrics with Micrometer

```xml
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-micrometer</artifactId>
    <version>4.15.0</version>
</dependency>

<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-registry-prometheus</artifactId>
</dependency>
```

```properties
# application.properties
management.endpoints.web.exposure.include=prometheus,health,metrics
camel.metrics.enabled=true
```

```java
@Component
public class MetricizedRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("kafka:orders")
            .routeId("order-processing")
            .log("Processing order")
            .bean(orderService, "process")
            .to("kafka:processed-orders");
    }
}
```

Metrics available at:
```bash
curl http://localhost:8080/actuator/prometheus | grep camel
```

### 13.3 Structured Logging

```xml
<dependency>
    <groupId>io.micrometer</groupId>
    <artifactId>micrometer-tracing-bridge-brave</artifactId>
</dependency>

<dependency>
    <groupId>io.zipkin.reporter2</groupId>
    <artifactId>zipkin-reporter-java</artifactId>
</dependency>
```

```properties
# application.properties
logging.pattern.console=%d{yyyy-MM-dd HH:mm:ss} - %msg%n
logging.level.org.apache.camel=INFO
logging.level.root=WARN

# Tracing
management.tracing.sampling.probability=1.0
```

### 13.4 Custom Metrics

```java
@Component
public class CustomMetricsRoute extends RouteBuilder {
    
    @Override
    public void configure() throws Exception {
        from("kafka:orders")
            .process(new OrderMetricsProcessor())
            .to("kafka:processed-orders");
    }
}

@Component
public class OrderMetricsProcessor implements Processor {
    
    private final MeterRegistry meterRegistry;
    
    public OrderMetricsProcessor(MeterRegistry meterRegistry) {
        this.meterRegistry = meterRegistry;
    }
    
    @Override
    public void process(Exchange exchange) throws Exception {
        String customerId = exchange.getIn().getHeader("customerId", String.class);
        BigDecimal amount = exchange.getIn().getBody(Order.class).getTotal();
        
        // Record metrics
        meterRegistry.counter("orders.processed").increment();
        meterRegistry.gauge("order.amount", amount.doubleValue());
        meterRegistry.timer("order.processing.duration").record(() -> {
            // processing time
        });
        
        meterRegistry.counter("orders.by.customer", "customerId", customerId).increment();
    }
}
```

### 13.5 Distributed Tracing

```java
@Component
public class TracedRoute extends RouteBuilder {
    @Override
    public void configure() throws Exception {
        from("kafka:orders")
            // Headers are automatically propagated for tracing
            .log("Order: ${header[b3]} (trace ID)")
            .bean(orderService, "process")
            .to("kafka:processed-orders");
    }
}
```

Traces appear in Zipkin dashboard at `http://localhost:9411`.

### 13.6 Chapter Exercise: Observable Integration System

[Comprehensive monitoring setup]

---

## Chapter 14: Cloud-Native Integration

### 14.1 Kubernetes Deployment

```yaml
# application.yaml - Spring Boot config
spring:
  application:
    name: camel-order-service
  kafka:
    bootstrap-servers: kafka:9092

management:
  endpoints:
    web:
      exposure:
        include: health,prometheus

camel:
  main:
    routes-include-pattern: classpath:routes/*.xml
```

```yaml
# kubernetes/deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: camel-order-service
spec:
  replicas: 3
  selector:
    matchLabels:
      app: camel-order-service
  template:
    metadata:
      labels:
        app: camel-order-service
    spec:
      containers:
      - name: camel-service
        image: camel-order-service:1.0.0
        ports:
        - containerPort: 8080
        env:
        - name: KAFKA_BOOTSTRAP_SERVERS
          value: kafka:9092
        - name: SPRING_PROFILES_ACTIVE
          value: prod
        resources:
          requests:
            cpu: "100m"
            memory: "256Mi"
          limits:
            cpu: "500m"
            memory: "512Mi"
        livenessProbe:
          httpGet:
            path: /actuator/health/live
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 10
        readinessProbe:
          httpGet:
            path: /actuator/health/ready
            port: 8080
          initialDelaySeconds: 10
          periodSeconds: 5
```

### 14.2 Camel K for Serverless

```yaml
apiVersion: camel.apache.org/v1
kind: Integration
metadata:
  name: order-processor
spec:
  flows:
  - routeId: process-orders
    from:
      uri: kafka:orders
    steps:
    - to:
        uri: direct:validate
    - to:
        uri: direct:process
  - from:
      uri: direct:validate
    steps:
    - filter:
        simple: ${body.customerId} != null
    - to:
        uri: direct:process
  - from:
      uri: direct:process
    steps:
    - log: "Order: ${body}"
    - to:
        uri: kafka:processed-orders
```

Deploy:
```bash
kamel run order-processor.yaml -n production
```

### 14.3 Quarkus Native Compilation

```xml
<dependency>
    <groupId>org.apache.camel.quarkus</groupId>
    <artifactId>camel-quarkus-kafka</artifactId>
    <version>4.15.0</version>
</dependency>
```

```bash
# Build native executable
mvn package -Dnative -Dquarkus.native.container-build=true

# Run native image
./target/camel-app-1.0.0-runner
```

Native image benefits:
- Startup time: ~50ms (vs 3-5s JVM)
- Memory: ~30MB (vs 200-300MB JVM)
- Container size: ~200MB (vs 700MB+ with JVM)

### 14.4 Docker Deployment

```dockerfile
# Dockerfile
FROM eclipse-temurin:21-jre-jammy

WORKDIR /app

COPY target/camel-app-*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```

```bash
docker build -t camel-order-service:1.0.0 .
docker run -e KAFKA_BOOTSTRAP_SERVERS=kafka:9092 -p 8080:8080 camel-order-service:1.0.0
```

### 14.5 Configuration Management

```properties
# application-prod.properties (for production)
spring.kafka.bootstrap-servers=${KAFKA_SERVERS}
camel.component.kafka.security-protocol=${KAFKA_SECURITY}

# Secrets via environment variables or ConfigMaps
server.ssl.key-store=${SSL_KEYSTORE_PATH}
server.ssl.key-store-password=${SSL_PASSWORD}
```

```yaml
# kubernetes/configmap.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: camel-config
data:
  application.properties: |
    spring.kafka.bootstrap-servers=kafka:9092
    camel.main.trace=false
    logging.level.org.apache.camel=INFO
---
apiVersion: v1
kind: Secret
metadata:
  name: camel-secrets
type: Opaque
stringData:
  ssl.password: ENC[AyM1SysPpbyDfgZld3umj1qzKObwVMkoqQ-EstJQLr_T-1qS0gZH75aKtMN3Yj0]
```

---

# PART 5: REAL-WORLD PROJECTS

## Project 1: Multi-Channel Order Processing System

### Requirements
- Accept orders from REST API, Kafka, and CSV files
- Validate, enrich, and route orders based on type
- Process payment and inventory
- Notify customers via email/SMS
- Handle failures gracefully

### Architecture

```
┌─────────────────────────────────────────────┐
│         Order Input Channels                │
│  ┌─────────────────────────────────────┐   │
│  │ REST API │ Kafka │ CSV File Import  │   │
│  └─────────────────────────────────────┘   │
└────────────────────┬────────────────────────┘
                     │
            ┌────────▼────────┐
            │  Validation     │
            │  Enrichment     │
            └────────┬────────┘
                     │
     ┌───────────────┼───────────────┐
     │               │               │
  ┌──▼──┐     ┌──────▼────┐  ┌──────▼─────┐
  │Digital  │   │ Physical   │  │ Service   │
  │Delivery │   │ Fulfillment│  │ Scheduling│
  └────┬───┘   └──────┬────┘  └──────┬─────┘
       │              │              │
       └──────────────┼──────────────┘
                      │
              ┌───────▼────────┐
              │ Notifications  │
              │ (Email/SMS)    │
              └────────────────┘
```

### Implementation

```java
@Component
public class OrderProcessingRoute extends RouteBuilder {
    
    @Autowired
    private OrderService orderService;
    
    @Autowired
    private PaymentService paymentService;
    
    @Autowired
    private InventoryService inventoryService;
    
    @Autowired
    private NotificationService notificationService;
    
    @Override
    public void configure() throws Exception {
        // Global error handler
        errorHandler(deadLetterChannel("kafka:order-errors")
            .maximumRedeliveries(3)
            .exponentialBackOff()
            .logStackTrace(true));
        
        // REST API endpoint
        restConfiguration()
            .component("jetty")
            .host("0.0.0.0")
            .port(8080);
        
        rest("/api/v1/orders")
            .post()
                .type(OrderRequest.class)
                .to("direct:receive-order");
        
        // Inbound routes
        from("direct:receive-order")
            .routeId("rest-to-processing")
            .to("direct:process-order");
        
        from("kafka:orders")
            .routeId("kafka-to-processing")
            .to("direct:process-order");
        
        from("file:inbox?noop=false&readLock=changed&include=.*\\.csv$")
            .routeId("csv-to-processing")
            .split(body().tokenize("\n")).streaming()
                .unmarshal().csv()
                .bean(csvConverter, "toOrder")
                .to("direct:process-order")
            .end();
        
        // Main processing pipeline
        from("direct:process-order")
            .routeId("order-processing")
            .setProperty("receivedAt", simple("${date:now}"))
            
            // Validate order
            .doTry()
                .bean(orderService, "validate(${body})")
                .log("Order validated: ${body.orderId}")
            .doCatch(ValidationException.class)
                .handled(true)
                .setHeader("error", simple("${exception.message}"))
                .to("direct:handle-validation-error")
                .stop()
            .end()
            
            // Enrich order with customer data
            .enrich()
                .simple("direct:get-customer")
                .aggregationStrategy(new CustomerEnrichmentStrategy())
                .log("Order enriched with customer data")
            
            // Route based on type
            .choice()
                .when(jsonpath("$.type == 'DIGITAL'"))
                    .to("direct:digital-fulfillment")
                .when(jsonpath("$.type == 'PHYSICAL'"))
                    .to("direct:physical-fulfillment")
                .when(jsonpath("$.type == 'SERVICE'"))
                    .to("direct:service-fulfillment")
                .otherwise()
                    .to("direct:unknown-order-type")
            .end()
            
            // Payment processing
            .doTry()
                .bean(paymentService, "processPayment(${body})")
                .log("Payment processed")
            .doCatch(PaymentException.class)
                .handled(true)
                .to("direct:handle-payment-error")
                .stop()
            .end()
            
            // Notifications
            .multicast()
                .to("direct:send-confirmation-email",
                    "direct:send-notification-sms")
            
            .setHeader("status", constant("PROCESSING"))
            .to("kafka:orders-processing");
        
        // Fulfillment routes
        from("direct:digital-fulfillment")
            .log("Digital fulfillment")
            .bean(orderService, "fulfillDigital(${body})")
            .to("kafka:digital-orders");
        
        from("direct:physical-fulfillment")
            .log("Physical fulfillment - checking inventory")
            .doTry()
                .bean(inventoryService, "reserveItems(${body})")
                .log("Inventory reserved")
                .to("kafka:warehouse-orders")
            .doCatch(InsufficientInventoryException.class)
                .handled(true)
                .log("Insufficient inventory")
                .to("direct:backorder")
            .end();
        
        from("direct:service-fulfillment")
            .log("Service scheduling")
            .bean(orderService, "scheduleService(${body})")
            .to("kafka:service-orders");
        
        from("direct:unknown-order-type")
            .log(LoggingLevel.ERROR, "Unknown order type: ${body.type}")
            .to("kafka:rejected-orders");
        
        // Error handling
        from("direct:handle-validation-error")
            .log("Validation error: ${header.error}")
            .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(400))
            .setBody(simple("{\"error\":\"${header.error}\"}"))
            .to("kafka:validation-errors");
        
        from("direct:handle-payment-error")
            .log("Payment error: ${header.error}")
            .to("kafka:payment-errors");
        
        from("direct:backorder")
            .log("Creating backorder")
            .to("kafka:backorders");
        
        // Notification routes
        from("direct:send-confirmation-email")
            .log("Sending confirmation email")
            .bean(notificationService, "sendEmail(${body})");
        
        from("direct:send-notification-sms")
            .log("Sending SMS notification")
            .bean(notificationService, "sendSms(${body})");
        
        // Get customer data
        from("direct:get-customer")
            .bean(orderService, "getCustomer");
    }
}

// Supporting classes
public class CustomerEnrichmentStrategy implements AggregationStrategy {
    @Override
    public Exchange aggregate(Exchange oldExchange, Exchange newExchange) {
        Order order = oldExchange.getIn().getBody(Order.class);
        Customer customer = newExchange.getIn().getBody(Customer.class);
        
        order.setCustomerEmail(customer.getEmail());
        order.setCustomerPhone(customer.getPhone());
        order.setCustomerTier(customer.getTier());
        
        oldExchange.getIn().setBody(order);
        return oldExchange;
    }
}
```

[Continues with Projects 2 & 3...]

---

## Appendix: Quick Reference

### Maven Dependencies
```xml
<!-- Core -->
<dependency>
    <groupId>org.apache.camel.springboot</groupId>
    <artifactId>camel-spring-boot-starter</artifactId>
    <version>4.15.0</version>
</dependency>

<!-- Common components -->
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-kafka</artifactId>
    <version>4.15.0</version>
</dependency>

<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-http</artifactId>
    <version>4.15.0</version>
</dependency>

<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-ftp</artifactId>
    <version>4.15.0</version>
</dependency>

<!-- Testing -->
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-spring-junit5</artifactId>
    <version>4.15.0</version>
    <scope>test</scope>
</dependency>
```

### Common DSL Patterns

| Pattern | Example |
|---------|---------|
| **Simple routing** | `from("kafka:orders").to("direct:process")` |
| **Content routing** | `.choice().when(...).to(...).otherwise().to(...)` |
| **Filtering** | `.filter(simple("${body.amount} > 100"))` |
| **Transformation** | `.setBody(simple("...")).setHeader("key", ...)` |
| **Error handling** | `.errorHandler(deadLetterChannel("...").maximumRedeliveries(3))` |
| **Splitting** | `.split(jsonpath("$.items[*]"))...end()` |
| **Aggregation** | `.aggregate(...).completionSize(10)...end()` |
| **Enrichment** | `.enrich("direct:fetch").aggregationStrategy(...)` |

---

## Conclusion

This curriculum provides a comprehensive path from Apache Camel fundamentals to production-grade enterprise integration. The progression from basic routing through complex patterns, cloud-native deployment, and real-world projects ensures you can architect and implement sophisticated integration solutions.

**Next Steps**:
1. Complete chapters 1-3 to master fundamentals
2. Work through chapters 4-7 focusing on your specific integration needs
3. Study chapters 11-14 before deploying to production
4. Implement the three real-world projects to solidify knowledge
5. Explore Camel's 300+ components as you encounter new integration requirements

**Resources**:
- Apache Camel Documentation: https://camel.apache.org
- Camel GitHub: https://github.com/apache/camel
- Community: https://camel.apache.org/community/

Your deep Java expertise and distributed systems background position you perfectly for mastering Apache Camel. Focus on understanding the patterns rather than memorizing syntax, and you'll quickly become the go-to expert for integration architecture in your organization.
