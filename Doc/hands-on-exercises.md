# Apache Camel Quick Start & Hands-On Exercises

## � Apache Camel Maven Dependencies Reference

This section lists the main Apache Camel Maven dependencies, their purposes, and their transitive dependencies. Apache Camel has hundreds of components, but these are the core and commonly used ones. Each dependency brings specific functionality and may include transitive dependencies for underlying libraries.

### Core Dependencies

- **camel-core** (org.apache.camel:camel-core)
  - **Purpose**: The fundamental Camel engine providing routing, Enterprise Integration Patterns (EIPs), message exchange, and core APIs. Required for any Camel application.
  - **Used in Code**: Enables RouteBuilder, from(), to(), log(), etc.
  - **Transitive Dependencies**:
    - `slf4j-api`: Logging facade for flexible logging backends.
    - `commons-lang3`: Utility classes for string manipulation, validation, etc.
    - `camel-util`: Internal utilities for Camel operations.
    - `camel-api`: Core APIs for exchanges, endpoints, etc.
  - **How They Help**: slf4j enables logging; commons-lang provides helpers for data processing; camel-util/camel-api provide the foundation for Camel functionality.

- **camel-main** (org.apache.camel:camel-main)
  - **Purpose**: Allows running Camel applications as standalone Java applications without external containers like Spring Boot.
  - **Used in Code**: Main class for bootstrapping Camel in standalone apps.
  - **Transitive Dependencies**:
    - `camel-core`: Inherits all core functionality.
    - `jline`: Command-line interface support for interactive console.
    - `commons-cli`: Command-line argument parsing.
  - **How They Help**: jline enables interactive shell features; commons-cli allows parsing startup arguments for configuration.

### Spring Boot Integration

- **camel-spring-boot-starter** (org.apache.camel.springboot:camel-spring-boot-starter)
  - **Purpose**: Integrates Camel with Spring Boot for auto-configuration, dependency injection, and Spring ecosystem features.
  - **Used in Code**: @Component on RouteBuilder classes, @Autowired for Camel beans.
  - **Transitive Dependencies**:
    - `spring-boot-starter`: Core Spring Boot functionality.
    - `camel-core`: Camel engine.
    - `camel-spring-boot-engine`: Spring Boot-specific Camel engine.
    - `camel-spring-boot`: Auto-configuration for Camel components.
  - **How They Help**: spring-boot-starter provides web server, configuration; camel-spring-boot enables automatic component registration and Spring integration.

- **camel-spring-boot** (org.apache.camel.springboot:camel-spring-boot)
  - **Purpose**: Provides Spring Boot auto-configuration for Camel components and routes.
  - **Used in Code**: Automatic configuration of Camel contexts and components.
  - **Transitive Dependencies**:
    - `camel-core`: Core engine.
    - `spring-boot-autoconfigure`: Spring Boot auto-configuration framework.
  - **How They Help**: Enables Camel components to be configured via application.properties without manual setup.

### Component Dependencies

- **camel-timer** (org.apache.camel:camel-timer)
  - **Purpose**: Provides timer-based message sources for periodic tasks.
  - **Used in Code**: from("timer:name?period=2000")
  - **Transitive Dependencies**: Minimal, mainly camel-core.
  - **How They Help**: Uses Java Timer API for scheduling; no additional libs needed.

- **camel-file** (org.apache.camel:camel-file)
  - **Purpose**: Enables reading from and writing to file systems.
  - **Used in Code**: from("file:input"), to("file:output")
  - **Transitive Dependencies**:
    - `commons-io`: File I/O utilities.
    - `camel-core`: Core functionality.
  - **How They Help**: commons-io provides robust file operations like copying, moving, and directory handling.

- **camel-log** (org.apache.camel:camel-log)
  - **Purpose**: Provides logging endpoints for debugging and monitoring.
  - **Used in Code**: .log("message"), to("log:category")
  - **Transitive Dependencies**: Minimal, uses slf4j from camel-core.
  - **How They Help**: Integrates with logging frameworks via slf4j for consistent logging across applications.

- **camel-jetty** (org.apache.camel:camel-jetty)
  - **Purpose**: Embeds Jetty HTTP server for REST endpoints and HTTP-based integrations.
  - **Used in Code**: restConfiguration().component("jetty")
  - **Transitive Dependencies**:
    - `jetty-server`, `jetty-security`, `jetty-util`: Jetty web server components.
    - `servlet-api`: Java Servlet API.
  - **How They Help**: jetty-* provide HTTP server capabilities; servlet-api enables servlet-based web applications.

- **camel-rest** (org.apache.camel:camel-rest)
  - **Purpose**: Enables REST DSL for defining RESTful APIs.
  - **Used in Code**: rest("/api").post()..., restConfiguration()
  - **Transitive Dependencies**: Minimal, mainly camel-core.
  - **How They Help**: Provides DSL for REST endpoint definition without manual HTTP handling.

- **camel-jackson** (org.apache.camel:camel-jackson)
  - **Purpose**: Supports JSON marshalling/unmarshalling using Jackson.
  - **Used in Code**: .marshal().json(), .unmarshal().json()
  - **Transitive Dependencies**:
    - `jackson-core`, `jackson-databind`, `jackson-annotations`: Jackson JSON processing.
  - **How They Help**: jackson-* enable fast, flexible JSON serialization/deserialization for data exchange.

- **camel-jdbc** (org.apache.camel:camel-jdbc)
  - **Purpose**: Provides JDBC connectivity for database operations.
  - **Used in Code**: to("jdbc:dataSource")
  - **Transitive Dependencies**:
    - `spring-jdbc`: Spring JDBC support.
    - `commons-dbcp2`: Database connection pooling.
  - **How They Help**: spring-jdbc simplifies JDBC operations; commons-dbcp2 manages connection pools for performance.

- **camel-kafka** (org.apache.camel:camel-kafka)
  - **Purpose**: Integrates with Apache Kafka for message streaming.
  - **Used in Code**: from("kafka:topic"), to("kafka:topic")
  - **Transitive Dependencies**:
    - `kafka-clients`: Kafka producer/consumer clients.
    - `zookeeper`: For Kafka coordination (if needed).
  - **How They Help**: kafka-clients provide high-throughput messaging; zookeeper enables distributed coordination.

- **camel-aws-s3** (org.apache.camel:camel-aws-s3)
  - **Purpose**: Integrates with Amazon S3 for cloud storage.
  - **Used in Code**: from("aws-s3://bucket"), to("aws-s3://bucket")
  - **Transitive Dependencies**:
    - `aws-java-sdk-s3`: AWS S3 SDK.
    - `aws-java-sdk-core`: Core AWS SDK functionality.
  - **How They Help**: aws-java-sdk-* provide authenticated access to S3 APIs for object storage operations.

- **camel-mail** (org.apache.camel:camel-mail)
  - **Purpose**: Supports email sending and receiving via SMTP/IMAP/POP3.
  - **Used in Code**: from("imap://host"), to("smtp://host")
  - **Transitive Dependencies**:
    - `javax.mail`: JavaMail API.
    - `activation`: MIME type handling.
  - **How They Help**: javax.mail provides email protocol support; activation handles content types.

### Testing Dependencies

- **camel-test-spring-junit5** (org.apache.camel:camel-test-spring-junit5)
  - **Purpose**: Testing framework for Camel routes with Spring Boot and JUnit 5.
  - **Used in Code**: @CamelSpringBootTest, ProducerTemplate for testing routes.
  - **Transitive Dependencies**:
    - `junit-jupiter`: JUnit 5 testing framework.
    - `mockito-core`: Mocking framework.
    - `camel-core`: Core testing utilities.
  - **How They Help**: junit-jupiter runs tests; mockito-core enables mocking dependencies; camel-core provides Camel-specific test helpers.

- **camel-test** (org.apache.camel:camel-test)
  - **Purpose**: Core testing utilities for Camel without Spring.
  - **Used in Code**: CamelTestSupport, MockEndpoint.
  - **Transitive Dependencies**:
    - `junit`: JUnit 4 testing framework.
    - `camel-core`: Testing extensions.
  - **How They Help**: junit provides test execution; camel-core adds Camel-specific assertions and mocking.

### Utility Dependencies

- **camel-bean** (org.apache.camel:camel-bean)
  - **Purpose**: Enables calling Java beans and methods in routes.
  - **Used in Code**: .bean(MyClass.class, "method")
  - **Transitive Dependencies**: Minimal.
  - **How They Help**: Uses reflection to invoke methods on POJOs for custom processing.

- **camel-direct** (org.apache.camel:camel-direct)
  - **Purpose**: Provides direct (synchronous) routing between routes.
  - **Used in Code**: from("direct:name"), to("direct:name")
  - **Transitive Dependencies**: None significant.
  - **How They Help**: Enables in-memory routing for connecting route segments.

- **camel-seda** (org.apache.camel:camel-seda)
  - **Purpose**: Provides asynchronous routing using in-memory queues.
  - **Used in Code**: from("seda:name"), to("seda:name")
  - **Transitive Dependencies**: None.
  - **How They Help**: Uses concurrent queues for decoupling producers and consumers.

### Data Format Dependencies

- **camel-csv** (org.apache.camel:camel-csv)
  - **Purpose**: Supports CSV data format marshalling/unmarshalling.
  - **Used in Code**: .marshal().csv(), .unmarshal().csv()
  - **Transitive Dependencies**:
    - `commons-csv`: CSV parsing library.
  - **How They Help**: commons-csv provides robust CSV reading/writing with proper escaping.

- **camel-xml-jaxp** (org.apache.camel:camel-xml-jaxp)
  - **Purpose**: XML processing using JAXP (Java API for XML Processing).
  - **Used in Code**: .marshal().xml(), .unmarshal().xml()
  - **Transitive Dependencies**:
    - `jaxp-ri`: Reference implementation of JAXP.
  - **How They Help**: jaxp-ri enables standard XML parsing and transformation.

### Error Handling Dependencies

- **camel-hystrix** (org.apache.camel:camel-hystrix)
  - **Purpose**: Circuit breaker pattern implementation using Hystrix.
  - **Used in Code**: .hystrix().circuitBreaker()...
  - **Transitive Dependencies**:
    - `hystrix-core`: Hystrix circuit breaker library.
    - `rxjava`: Reactive programming support.
  - **How They Help**: hystrix-core provides fault tolerance; rxjava enables reactive error handling.

### Notes on Dependencies
- **Version Management**: All dependencies should use the same Camel version (e.g., 4.15.0) for compatibility.
- **Transitive Conflicts**: Be aware of version conflicts in transitive dependencies; use Maven's dependency management to resolve them.
- **Minimal Dependencies**: Start with camel-core and add components as needed to keep applications lightweight.
- **Spring Boot vs Standalone**: Use camel-spring-boot-starter for Spring Boot apps; use camel-main for standalone apps.
- **Testing**: Always include appropriate test dependencies for reliable integration testing.

This reference covers the most commonly used Camel dependencies. For specialized integrations (e.g., cloud services, databases, messaging systems), check the [Camel Component Reference](https://camel.apache.org/components/) for specific dependencies.

---

## �🚀 15-Minute Setup & First Route

### Prerequisites
```bash
java -version          # 11+
mvn -version          # 3.6+
git version           # Latest
```

### Create Project
```bash
mvn archetype:generate \
  -DgroupId=com.example \
  -DartifactId=camel-quickstart \
  -DarchetypeGroupId=org.apache.camel.archetypes \
  -DarchetypeArtifactId=camel-archetype-spring-boot \
  -DarchetypeVersion=4.15.0

cd camel-quickstart
mvn spring-boot:run
```

---

## ✅ Exercise 1: Hello World (15 minutes)

### Goal
Create a timer that fires every 2 seconds and logs "Hello from Camel"

### Code
**src/main/java/com/example/MyRoute.java**:
```java
package com.example;

import org.apache.camel.builder.RouteBuilder;
import org.springframework.stereotype.Component;

@Component
public class MyRoute extends RouteBuilder {
    
    @Override
    public void configure() throws Exception {
        from("timer:hello?period=2000")
            .log("Hello from Camel!")
            .to("log:myapp");
    }
}
```

### Scenario Description
In this exercise, you will build a simple Apache Camel route that demonstrates the basic concepts of routing and logging. The route will use a timer component to trigger messages every 2 seconds, log a greeting message, and send it to a logger endpoint. This simulates a periodic task that could be used for monitoring, health checks, or scheduled operations in a real-world application.

### What the Code Does
- `from("timer:hello?period=2000")`: Creates a consumer that generates a message every 2000 milliseconds (2 seconds). The timer component is useful for scheduled tasks.
- `.log("Hello from Camel!")`: Logs the message "Hello from Camel!" to the console using Camel's logging framework.
- `.to("log:myapp")`: Sends the message to a log endpoint with the category "myapp". This is redundant with the log() method but shows how to route to different endpoints.

The route runs indefinitely, printing the log message every 2 seconds.

### Dependencies
Add these to your pom.xml:

```xml
<dependencies>
    <!-- Spring Boot Starter for Camel -->
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-spring-boot-starter</artifactId>
        <version>4.15.0</version>
    </dependency>
    <!-- Timer component for scheduled triggers -->
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-timer</artifactId>
        <version>4.15.0</version>
    </dependency>
    <!-- Log component for logging endpoints -->
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-log</artifactId>
        <version>4.15.0</version>
    </dependency>
    <!-- Spring Boot Web for REST if needed later -->
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

- `camel-spring-boot-starter`: Provides the core Camel functionality integrated with Spring Boot.
- `camel-timer`: Enables the timer component for periodic message generation.
- `camel-log`: Allows routing messages to logging endpoints.
- `spring-boot-starter-web`: Includes Spring Web for HTTP support, useful for later exercises.

### Full Working Standalone Example
To run this as a standalone application, you can use the Spring Boot main class. The archetype generates it, but here's the complete setup:

**pom.xml** (generated by archetype, but ensure these dependencies):

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>camel-quickstart</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.apache.camel.springboot</groupId>
            <artifactId>camel-spring-boot-starter</artifactId>
            <version>4.15.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-timer</artifactId>
            <version>4.15.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-log</artifactId>
            <version>4.15.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**src/main/java/com/example/CamelApplication.java** (generated by archetype):

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class CamelApplication {

    public static void main(String[] args) {
        SpringApplication.run(CamelApplication.class, args);
    }
}
```

**src/main/java/com/example/MyRoute.java** (as above).

To run: `mvn spring-boot:run`

### More Examples
1. **Custom Timer Name and Message**:
```java
from("timer:myTimer?period=5000")
    .setBody(constant("Custom message"))
    .log("${body}");
```
This sets a custom body and logs it every 5 seconds.

2. **Timer with Fixed Rate**:
```java
from("timer:fixed?fixedRate=true&period=3000")
    .log("Fixed rate timer: ${date:now}");
```
Uses fixed rate instead of delay between executions.

### Test It
```bash
# Terminal 1: Start app
mvn spring-boot:run

# You should see:
# Hello from Camel!
# Hello from Camel!
# ...every 2 seconds
```

### Challenge
Modify the route to:
- Log the current timestamp
- Log a custom message with the timestamp
- Change interval to 5 seconds

**Solution**:
```java
from("timer:hello?period=5000")
    .setBody(simple("Hello at ${date:now:yyyy-MM-dd HH:mm:ss}"))
    .log("${body}")
    .to("file:target/messages?fileName=log-${date:now:yyyyMMddHHmmss}.txt");
```

---

## ✅ Exercise 2: REST API (30 minutes)

### Goal
Create a REST API endpoint that accepts POST requests

### Code
**src/main/java/com/example/RestRoute.java**:
```java
package com.example;

import org.apache.camel.builder.RouteBuilder;
import org.springframework.stereotype.Component;

@Component
public class RestRoute extends RouteBuilder {
    
    @Override
    public void configure() throws Exception {
        restConfiguration()
            .component("jetty")
            .host("0.0.0.0")
            .port(8080);
        
        rest("/api/greet")
            .post()
                .type(String.class)
                .to("direct:process-greeting");
        
        from("direct:process-greeting")
            .log("Greeting received: ${body}")
            .transform().simple("Hello, ${body}!")
            .marshal().json();
    }
}
```

### Scenario Description
In this exercise, you will create a RESTful API using Apache Camel's REST DSL. The API will expose a POST endpoint at /api/greet that accepts a string (e.g., a name) and responds with a greeting message in JSON format. This demonstrates how Camel can be used to build microservices or API gateways that integrate with other systems.

### What the Code Does
- `restConfiguration().component("jetty").host("0.0.0.0").port(8080)`: Configures the REST component to use Jetty as the HTTP server, listening on all interfaces at port 8080.
- `rest("/api/greet").post().type(String.class).to("direct:process-greeting")`: Defines a REST endpoint for POST requests to /api/greet, expecting a String body, and routes the request to a direct endpoint for processing.
- `from("direct:process-greeting")`: Consumes messages from the direct endpoint.
- `.log("Greeting received: ${body}")`: Logs the received body.
- `.transform().simple("Hello, ${body}!")`: Transforms the message body to a greeting string.
- `.marshal().json()`: Converts the response to JSON format.

The route processes incoming POST requests, logs them, transforms the input into a greeting, and returns it as JSON.

### Dependencies
Add these to your pom.xml:

```xml
<dependencies>
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-spring-boot-starter</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-jetty</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-rest</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-jackson</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
</dependencies>
```

- `camel-spring-boot-starter`: Core Camel with Spring Boot.
- `camel-jetty`: Provides the Jetty HTTP server for REST endpoints.
- `camel-rest`: Enables the REST DSL for defining RESTful routes.
- `camel-jackson`: Supports JSON marshalling/unmarshalling using Jackson.
- `spring-boot-starter-web`: Spring Web support.

### Full Working Standalone Example
**pom.xml**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>camel-rest-example</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.apache.camel.springboot</groupId>
            <artifactId>camel-spring-boot-starter</artifactId>
            <version>4.15.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-jetty</artifactId>
            <version>4.15.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-rest</artifactId>
            <version>4.15.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-jackson</artifactId>
            <version>4.15.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**src/main/java/com/example/CamelApplication.java**:

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class CamelApplication {

    public static void main(String[] args) {
        SpringApplication.run(CamelApplication.class, args);
    }
}
```

**src/main/java/com/example/RestRoute.java** (as above).

To run: `mvn spring-boot:run`

### More Examples
1. **GET Endpoint with Path Parameter**:
```java
rest("/api/user/{id}")
    .get()
        .to("direct:get-user");

from("direct:get-user")
    .setBody(simple("User ID: ${header.id}"))
    .marshal().json();
```
Test: `curl http://localhost:8080/api/user/123`

2. **PUT Endpoint with JSON Input**:
```java
rest("/api/update")
    .put()
        .type(User.class)
        .to("direct:update-user");

from("direct:update-user")
    .log("Updating user: ${body.name}")
    .transform().simple("Updated: ${body.name}")
    .marshal().json();
```
Where User is a POJO.

### Test It
```bash
# Terminal 1: Start app
mvn spring-boot:run

# Terminal 2: Test the API
curl -X POST http://localhost:8080/api/greet \
  -H "Content-Type: application/json" \
  -d '"World"'

# Response: "Hello, World!"
```

### Challenge
1. Add query parameter for greeting type (formal/casual)
2. Add response code 201 for success
3. Add error handling for empty input

**Solution**:
```java
rest("/api/greet")
    .post()
        .type(String.class)
        .param()
            .name("style")
            .type(RestParamType.query)
            .defaultValue("casual")
        .endParam()
        .to("direct:process-greeting");

from("direct:process-greeting")
    .choice()
        .when(simple("${header.style} == 'formal'"))
            .transform().simple("Greetings, ${body}")
        .otherwise()
            .transform().simple("Hey ${body}!")
    .end()
    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(201));
```

---

## ✅ Exercise 3: File Processing (45 minutes)

### Goal
Monitor a directory for CSV files and process them

### Setup
```bash
mkdir -p data/inbox data/processed data/errors
```

### Code
**src/main/java/com/example/FileRoute.java**:
```java
package com.example;

import org.apache.camel.Exchange;
import org.apache.camel.Processor;
import org.apache.camel.builder.RouteBuilder;
import org.springframework.stereotype.Component;

@Component
public class FileRoute extends RouteBuilder {
    
    @Override
    public void configure() throws Exception {
        from("file:data/inbox?noop=false&readLock=changed&include=.*\\.csv$")
            .routeId("csv-processor")
            .log("Processing file: ${header.CamelFileName}")
            .process(new CsvValidationProcessor())
            .choice()
                .when(simple("${header.valid} == 'true'"))
                    .log("File valid, moving to processed")
                    .to("file:data/processed?fileName=${header.CamelFileName}")
                .otherwise()
                    .log("File invalid, moving to error")
                    .to("file:data/errors?fileName=${header.CamelFileName}")
            .end();
    }
    
    public static class CsvValidationProcessor implements Processor {
        @Override
        public void process(Exchange exchange) throws Exception {
            String body = exchange.getIn().getBody(String.class);
            
            // Simple validation: check if CSV has data
            boolean valid = body != null && body.split("\n").length > 1;
            exchange.getIn().setHeader("valid", valid);
        }
    }
}
```

### Scenario Description
In this exercise, you will implement file-based integration where Apache Camel monitors a directory for incoming CSV files, validates their content, and routes them to appropriate folders based on validation results. This is common in ETL (Extract, Transform, Load) processes, data ingestion pipelines, or file-based messaging systems.

### What the Code Does
- `from("file:data/inbox?noop=false&readLock=changed&include=.*\\.csv$")`: Monitors the data/inbox directory for CSV files. Options: noop=false deletes processed files, readLock=changed prevents reading incomplete files, include filters for .csv files.
- `.routeId("csv-processor")`: Assigns an ID to the route for management.
- `.log("Processing file: ${header.CamelFileName}")`: Logs the filename being processed.
- `.process(new CsvValidationProcessor())`: Applies custom validation logic.
- `.choice()`: Starts conditional routing based on validation.
- `.when(simple("${header.valid} == 'true'"))`: If valid, log and move to processed folder.
- `.otherwise()`: If invalid, log and move to errors folder.
- `.to("file:data/processed?fileName=${header.CamelFileName}")`: Moves valid files to processed directory.

The CsvValidationProcessor checks if the CSV has more than one line (header + data).

### Dependencies
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-spring-boot-starter</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-file</artifactId>
        <version>4.15.0</version>
    </dependency>
</dependencies>
```

- `camel-spring-boot-starter`: Core Camel.
- `camel-file`: Enables file component for reading/writing files.

### Full Working Standalone Example
**pom.xml**:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>camel-file-example</artifactId>
    <version>1.0-SNAPSHOT</version>

    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.1.0</version>
    </parent>

    <dependencies>
        <dependency>
            <groupId>org.apache.camel.springboot</groupId>
            <artifactId>camel-spring-boot-starter</artifactId>
            <version>4.15.0</version>
        </dependency>
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-file</artifactId>
            <version>4.15.0</version>
        </dependency>
    </dependencies>

    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
        </plugins>
    </build>
</project>
```

**src/main/java/com/example/CamelApplication.java** (same as before).

**src/main/java/com/example/FileRoute.java** (as above).

To run: `mvn spring-boot:run`

### More Examples
1. **File Copy with Transformation**:
```java
from("file:data/input?include=*.txt")
    .transform().simple("Processed: ${body}")
    .to("file:data/output?fileName=processed-${header.CamelFileName}");
```

2. **File Split and Process Lines**:
```java
from("file:data/input?include=*.csv")
    .split().tokenize("\n")
    .log("Line: ${body}")
    .to("file:data/lines");
```

### Test It
```bash
# Create test CSV
cat > data/inbox/test.csv << EOF
id,name,email
1,John,john@example.com
2,Jane,jane@example.com
EOF

# File should move to data/processed/test.csv

# Create invalid CSV
echo "" > data/inbox/empty.csv

# File should move to data/errors/empty.csv
```

### Challenge
1. Add line count validation
2. Parse CSV and validate email format
3. Write processing results to log file

---

## ✅ Exercise 4: Content-Based Routing (45 minutes)

### Goal
Route messages to different endpoints based on content

### Code
**src/main/java/com/example/OrderRoute.java**:
```java
package com.example;

import org.apache.camel.builder.RouteBuilder;
import org.springframework.stereotype.Component;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    private String id;
    private String type;  // DIGITAL, PHYSICAL, SERVICE
    private Double amount;
}

@Component
public class OrderRoute extends RouteBuilder {
    
    @Override
    public void configure() throws Exception {
        restConfiguration()
            .component("jetty")
            .host("0.0.0.0")
            .port(8080);
        
        rest("/api/orders")
            .post()
                .type(Order.class)
                .to("direct:route-order");
        
        from("direct:route-order")
            .log("Order received: ${body.id} (${body.type})")
            .choice()
                .when(simple("${body.type} == 'DIGITAL'"))
                    .log("Routing to digital fulfillment")
                    .to("direct:digital-fulfillment")
                .when(simple("${body.type} == 'PHYSICAL'"))
                    .log("Routing to warehouse")
                    .to("direct:physical-fulfillment")
                .when(simple("${body.type} == 'SERVICE'"))
                    .log("Routing to service")
                    .to("direct:service-fulfillment")
                .otherwise()
                    .log("ERROR: Unknown type")
                    .to("direct:error-handling")
            .end();
        
        from("direct:digital-fulfillment")
            .transform().simple("Digital order ${body.id} fulfilled immediately")
            .marshal().json();
        
        from("direct:physical-fulfillment")
            .transform().simple("Physical order ${body.id} sent to warehouse")
            .marshal().json();
        
        from("direct:service-fulfillment")
            .transform().simple("Service order ${body.id} scheduled")
            .marshal().json();
        
        from("direct:error-handling")
            .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(400))
            .transform().simple("{\"error\": \"Invalid order type\"}")
            .marshal().json();
    }
}
```

### Scenario Description
In this exercise, you will implement content-based routing where incoming order messages are routed to different processing paths based on the order type (DIGITAL, PHYSICAL, SERVICE). This pattern is essential for enterprise integration, allowing different business logic for different message types.

### What the Code Does
- Defines an Order POJO with id, type, amount.
- REST endpoint /api/orders accepts POST with Order JSON.
- Routes to direct:route-order.
- Uses .choice() to route based on ${body.type}.
- Each branch transforms the response differently and marshals to JSON.
- Error handling for unknown types.

### Dependencies
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-spring-boot-starter</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-jetty</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-rest</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-jackson</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.projectlombok</groupId>
        <artifactId>lombok</artifactId>
        <version>1.18.30</version>
        <scope>provided</scope>
    </dependency>
</dependencies>
```

- `camel-spring-boot-starter`: Core Camel.
- `camel-jetty`: HTTP server.
- `camel-rest`: REST DSL.
- `camel-jackson`: JSON support.
- `lombok`: For @Data annotation to generate getters/setters.

### Full Working Standalone Example
**pom.xml** (include lombok dependency).

**src/main/java/com/example/Order.java**:
```java
package com.example;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;

@Data
@AllArgsConstructor
@NoArgsConstructor
public class Order {
    private String id;
    private String type;  // DIGITAL, PHYSICAL, SERVICE
    private Double amount;
}
```

**src/main/java/com/example/OrderRoute.java** (as above).

To run: `mvn spring-boot:run`

### More Examples
1. **Routing Based on Amount**:
```java
.choice()
    .when(simple("${body.amount} > 100"))
        .to("direct:vip-processing")
    .otherwise()
        .to("direct:standard-processing");
```

2. **Multiple Conditions**:
```java
.when(simple("${body.type} == 'PHYSICAL' && ${body.amount} > 500"))
    .to("direct:premium-shipping");
```

### Test It
```bash
# Test Digital Order
curl -X POST http://localhost:8080/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "id": "ORD-001",
    "type": "DIGITAL",
    "amount": 99.99
  }'

# Test Physical Order
curl -X POST http://localhost:8080/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "id": "ORD-002",
    "type": "PHYSICAL",
    "amount": 199.99
  }'

# Test Invalid Type
curl -X POST http://localhost:8080/api/orders \
  -H "Content-Type: application/json" \
  -d '{
    "id": "ORD-003",
    "type": "INVALID",
    "amount": 99.99
  }'
```

### Challenge
1. Add amount-based routing (>$1000 = VIP processing)
2. Log routing decisions to file
3. Add timestamp to all responses

---

## ✅ Exercise 5: Error Handling & Retries (60 minutes)

### Goal
Implement resilient error handling with retries

### Code
**src/main/java/com/example/ResilientRoute.java**:
```java
package com.example;

import org.apache.camel.builder.RouteBuilder;
import org.springframework.stereotype.Component;

@Component
public class ResilientRoute extends RouteBuilder {
    
    @Override
    public void configure() throws Exception {
        // Global error handler
        errorHandler(deadLetterChannel("direct:dead-letter-queue")
            .maximumRedeliveries(3)
            .redeliveryDelay(1000)
            .backOffMultiplier(2.0)
            .useExponentialBackOff()
            .logStackTrace(true)
            .logExhaustedMessageHistory(true));
        
        // Route with retry
        from("timer:failing-api?period=5000")
            .setBody(constant("{\"action\": \"process\"}"))
            .doTry()
                .to("direct:call-unreliable-service")
                .log("Success!")
            .doCatch(Exception.class)
                .log("Error caught, will retry due to global handler")
                .rethrow()
            .doFinally()
                .log("Processing complete (success or error)")
            .end();
        
        from("direct:call-unreliable-service")
            .log("Calling unreliable service")
            .process(exchange -> {
                // Simulate 80% failure rate
                if (Math.random() > 0.2) {
                    throw new RuntimeException("Service call failed!");
                }
            })
            .log("Service call succeeded");
        
        from("direct:dead-letter-queue")
            .log("DEAD LETTER: Message failed after all retries: ${body}")
            .to("file:data/dlq?fileName=failed-${date:now:yyyyMMddHHmmss}.json");
    }
}
```

### Scenario Description
In this exercise, you will implement error handling and retry mechanisms to make your Camel routes resilient to failures. This includes global error handlers with dead letter queues, exponential backoff, and local try-catch blocks.

### What the Code Does
- `errorHandler(deadLetterChannel(...))`: Sets global error handler that retries up to 3 times with increasing delay, then sends to DLQ.
- `from("timer:...")`: Timer triggers every 5s.
- `.doTry()...doCatch()...doFinally()`: Local error handling.
- `from("direct:call-unreliable-service")`: Simulates a service that fails 80% of the time.
- DLQ route logs and saves failed messages to file.

### Dependencies
```xml
<dependencies>
    <dependency>
        <groupId>org.apache.camel.springboot</groupId>
        <artifactId>camel-spring-boot-starter</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-timer</artifactId>
        <version>4.15.0</version>
    </dependency>
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-file</artifactId>
        <version>4.15.0</version>
    </dependency>
</dependencies>
```

- `camel-spring-boot-starter`: Core Camel.
- `camel-timer`: For timer component.
- `camel-file`: For DLQ file output.

### Full Working Standalone Example
**pom.xml** (as above).

**src/main/java/com/example/ResilientRoute.java** (as above).

To run: `mvn spring-boot:run`

### More Examples
1. **Different Retry for Different Exceptions**:
```java
onException(ConnectionException.class)
    .maximumRedeliveries(5)
    .redeliveryDelay(500);

onException(ValidationException.class)
    .maximumRedeliveries(0)
    .handled(true);
```

2. **Circuit Breaker** (requires camel-hystrix):
```java
.hystrix()
    .hystrixConfiguration()
        .circuitBreakerEnabled(true)
        .circuitBreakerRequestVolumeThreshold(5)
    .end()
.to("direct:service");
```

### Test It
```bash
mkdir -p data/dlq

# Start app - watch logs for retries
mvn spring-boot:run

# Observe:
# 1st attempt
# Error caught, will retry
# 2nd attempt (after 1s)
# Error caught, will retry
# 3rd attempt (after 2s)
# Error caught, will retry
# 4th attempt (after 4s)
# DEAD LETTER: Message failed...
# File saved to data/dlq/
```

### Challenge
1. Different retry counts for different error types
2. Exponential backoff with jitter (randomness)
3. Alert/notification when message hits DLQ

**Solution**:
```java
onException(ConnectionException.class)
    .maximumRedeliveries(5)
    .redeliveryDelay(500)
    .exponentialBackOff();

onException(ValidationException.class)
    .maximumRedeliveries(0)
    .handled(true)
    .setHeader(Exchange.HTTP_RESPONSE_CODE, constant(400));
```

---

## ✅ Exercise 6: Testing (60 minutes)

### Goal
Write comprehensive tests for your routes

### Setup
```xml
<!-- Add to pom.xml -->
<dependency>
    <groupId>org.apache.camel</groupId>
    <artifactId>camel-test-spring-junit5</artifactId>
    <version>4.15.0</version>
    <scope>test</scope>
</dependency>
```

### Code
**src/test/java/com/example/OrderRouteTest.java**:
```java
package com.example;

import org.apache.camel.CamelContext;
import org.apache.camel.ProducerTemplate;
import org.apache.camel.test.spring.junit5.CamelSpringBootTest;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.test.context.SpringBootTest;
import org.junit.jupiter.api.Test;

@CamelSpringBootTest
@SpringBootTest
public class OrderRouteTest {
    
    @Autowired
    private ProducerTemplate template;
    
    @Autowired
    private CamelContext context;
    
    @Test
    public void testDigitalOrderRouting() throws Exception {
        String response = template.requestBody(
            "direct:route-order",
            new Order("ORD-001", "DIGITAL", 99.99),
            String.class
        );
        
        assert(response.contains("Digital order"));
        assert(response.contains("ORD-001"));
    }
    
    @Test
    public void testPhysicalOrderRouting() throws Exception {
        String response = template.requestBody(
            "direct:route-order",
            new Order("ORD-002", "PHYSICAL", 199.99),
            String.class
        );
        
        assert(response.contains("warehouse"));
    }
    
    @Test
    public void testInvalidOrderType() throws Exception {
        // Should throw or return error
        try {
            template.requestBody(
                "direct:route-order",
                new Order("ORD-003", "INVALID", 99.99),
                String.class
            );
        } catch (Exception e) {
            assert(e.getMessage().contains("Invalid"));
        }
    }
}
```

### Scenario Description
In this exercise, you will write unit and integration tests for your Camel routes using Spring Boot Test and Camel Test frameworks. Testing ensures that your integration logic works correctly and helps prevent regressions.

### What the Code Does
- `@CamelSpringBootTest`: Enables Camel testing with Spring Boot.
- `@SpringBootTest`: Loads the full application context.
- `ProducerTemplate`: Used to send messages to routes.
- Tests send Order objects to "direct:route-order" and assert responses.

### Dependencies
```xml
<dependencies>
    <!-- Test dependencies -->
    <dependency>
        <groupId>org.apache.camel</groupId>
        <artifactId>camel-test-spring-junit5</artifactId>
        <version>4.15.0</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-test</artifactId>
        <scope>test</scope>
    </dependency>
</dependencies>
```

- `camel-test-spring-junit5`: Camel testing support for JUnit 5.
- `spring-boot-starter-test`: Spring Boot testing utilities.

### Full Working Standalone Example
**pom.xml** with test dependencies.

**src/test/java/com/example/OrderRouteTest.java** (as above).

To run: `mvn test`

### More Examples
1. **Mock Endpoint Test**:
```java
@Autowired
private MockEndpoint mockEndpoint;

@Test
public void testMockEndpoint() throws Exception {
    mockEndpoint.expectedMessageCount(1);
    mockEndpoint.expectedBodiesReceived("expected body");
    
    template.sendBody("direct:input", "test");
    
    mockEndpoint.assertIsSatisfied();
}
```

2. **AdviceWith for Route Modification in Tests**:
```java
@CamelSpringBootTest
public class RouteTest {
    
    @Autowired
    private CamelContext context;
    
    @Test
    public void testRoute() throws Exception {
        AdviceWith.adviceWith(context, "route-id", route -> 
            route.replaceFromWith("direct:test")
        );
        
        // Test the modified route
    }
}
```

### Run Tests
```bash
mvn test

# Should see all tests PASS
```

### Challenge
1. Test error cases explicitly
2. Mock external service calls
3. Test timeout scenarios

---

## 📊 Exercise Progression Matrix

| Exercise | Duration | Difficulty | Core Concepts | Next Topic |
|----------|----------|------------|---------------|-----------|
| 1: Hello World | 15 min | ⭐ | Timer, logging | REST APIs |
| 2: REST API | 30 min | ⭐⭐ | REST DSL, JSON | File handling |
| 3: File Processing | 45 min | ⭐⭐ | File consumer, validation | Routing |
| 4: Content Routing | 45 min | ⭐⭐⭐ | Choice, DSL | Error handling |
| 5: Error Handling | 60 min | ⭐⭐⭐ | DLQ, retries | Testing |
| 6: Testing | 60 min | ⭐⭐⭐⭐ | Unit/integration tests | Production |

**Total Time**: ~4.5 hours to complete all exercises

---

## 🎯 Progressive Challenge: Order Processing Pipeline

After completing exercises 1-6, implement this **complete system**:

### Requirements
1. Accept orders via REST API (POST /api/orders)
2. Validate order data
3. Route based on order type (DIGITAL/PHYSICAL/SERVICE)
4. Simulate payment processing (with 10% failure rate)
5. Handle errors with retries
6. Log successful orders to file
7. Write failed orders to DLQ
8. Expose Prometheus metrics
9. Write integration tests

### Estimated Time
**4-6 hours for someone who's completed exercises 1-6**

### Bonus Challenges
- Add customer enrichment from mock service
- Split physical orders by warehouse region
- Aggregate orders by customer daily
- Add health checks
- Deploy to Docker

---

## 📝 Exercise Checklist

Mark your progress:

### Fundamentals (Week 1)
- [ ] Exercise 1: Hello World
- [ ] Exercise 2: REST API
- [ ] Exercise 3: File Processing

### Integration Patterns (Week 2-3)
- [ ] Exercise 4: Content Routing
- [ ] Exercise 5: Error Handling
- [ ] Exercise 6: Testing

### Real-World (Week 4)
- [ ] Progressive Challenge: Order Pipeline
- [ ] Bonus: Add monitoring
- [ ] Bonus: Deploy to Kubernetes

---

## 🐛 Troubleshooting Common Issues

### Route Not Starting
```bash
# Check if route ID is unique
# Check no syntax errors in configure()
# Check dependencies in pom.xml
mvn clean compile
```

### File Not Processing
```bash
# Verify directory exists
mkdir -p data/inbox

# Check file permissions
ls -la data/inbox/

# Add debug logging
.log(LoggingLevel.DEBUG, "${body}")
```

### REST Endpoint 404
```bash
# Verify port (default 8080)
curl http://localhost:8080/actuator/health

# Check REST configuration
# Restart application
mvn spring-boot:run
```

### Test Failures
```bash
# Check mock endpoint names match route URIs
# Verify message types (String vs Object)
# Check timeout values for async operations
mvn test -X
```

---

## 🚀 Next Steps After Exercises

1. **Read Chapter 4** (REST APIs) → Build production API
2. **Read Chapter 7** (Error Handling) → Implement resilience
3. **Read Chapter 11** (Testing) → Write test suite
4. **Implement Project 1** (Order Processing) → Full system

---

## 💬 Getting Help

When stuck:
1. Check Apache Camel docs: camel.apache.org
2. Search Stack Overflow: `[apache-camel] your-question`
3. Read chapter examples in the guide
4. Check component-specific docs for your integration

**Happy coding!** 🎉

---

## 🚀 Standalone Camel Example: File Processing without Spring Boot

### Scenario Description
This example demonstrates how to run Apache Camel as a standalone application without Spring Boot. The application monitors a directory for incoming files, reads their content, logs it to the console, and moves the files to a "processed" folder. This is useful for simple integration tasks that don't require the full Spring ecosystem.

### What the Code Does
- Uses `camel-main` to bootstrap Camel from a main method.
- The route polls the `input` directory for files.
- For each file, it logs the filename and content.
- Moves the file to the `processed` directory.
- Runs indefinitely until stopped (Ctrl+C).

### Dependencies
```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.example</groupId>
    <artifactId>camel-standalone-file</artifactId>
    <version>1.0-SNAPSHOT</version>

    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <camel.version>4.15.0</camel.version>
    </properties>

    <dependencies>
        <!-- Core Camel functionality -->
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-core</artifactId>
            <version>${camel.version}</version>
        </dependency>
        <!-- Main class for standalone execution -->
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-main</artifactId>
            <version>${camel.version}</version>
        </dependency>
        <!-- File component for file operations -->
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-file</artifactId>
            <version>${camel.version}</version>
        </dependency>
        <!-- Logging support -->
        <dependency>
            <groupId>org.apache.camel</groupId>
            <artifactId>camel-log</artifactId>
            <version>${camel.version}</version>
        </dependency>
    </dependencies>
</project>
```

- `camel-core`: Provides the core Camel engine and routing capabilities.
- `camel-main`: Enables running Camel applications from a main method without external containers.
- `camel-file`: Supports file system operations like reading from and writing to directories.
- `camel-log`: Allows logging messages to the console or log files.

### Full Working Standalone Example

**src/main/java/com/example/FileProcessorMain.java**:
```java
package com.example;

import org.apache.camel.main.Main;

public class FileProcessorMain {
    
    public static void main(String[] args) throws Exception {
        // Create Camel Main instance
        Main main = new Main();
        
        // Add the route builder
        main.configure().addRoutesBuilder(new FileRoute());
        
        // Start Camel and run until stopped
        main.run();
    }
}
```

**src/main/java/com/example/FileRoute.java**:
```java
package com.example;

import org.apache.camel.builder.RouteBuilder;

public class FileRoute extends RouteBuilder {
    
    @Override
    public void configure() throws Exception {
        // Monitor input directory for files
        from("file:input?delete=true")
            .routeId("file-processor")
            // Log the filename being processed
            .log("Processing file: ${header.CamelFileName}")
            // Log the file content
            .log("File content: ${body}")
            // Move file to processed directory
            .to("file:processed");
    }
}
```

### How to Run
1. Create the project structure:
```
mkdir -p camel-standalone-file/src/main/java/com/example
mkdir -p input processed
```

2. Place the `pom.xml` in the root.

3. Create the Java files as shown.

4. Compile and run:
```bash
mvn compile
mvn exec:java -Dexec.mainClass="com.example.FileProcessorMain"
```

5. Test by placing a file in the `input` directory:
```bash
echo "Hello, Camel!" > input/test.txt
```

You should see logs like:
```
Processing file: test.txt
File content: Hello, Camel!
```

The file will be moved to `processed/test.txt`.

### More Examples
1. **Process only .txt files**:
```java
from("file:input?include=*.txt&delete=true")
```

2. **Add custom processing**:
```java
from("file:input?delete=true")
    .process(exchange -> {
        String content = exchange.getIn().getBody(String.class);
        String upper = content.toUpperCase();
        exchange.getIn().setBody(upper);
    })
    .log("Processed content: ${body}")
    .to("file:processed");
```

3. **Handle errors**:
```java
from("file:input?delete=true")
    .doTry()
        .log("Processing: ${header.CamelFileName}")
        .to("file:processed")
    .doCatch(Exception.class)
        .log("Error processing file: ${exception.message}")
        .to("file:errors")
    .end();
```
