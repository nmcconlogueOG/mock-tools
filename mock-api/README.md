# mock-api

A Spring Boot reactive web application demonstrating the use of the `mock-service` library for testing.

## Overview

This project is a sample reactive REST API built with Spring WebFlux that showcases:
- Reactive programming with Project Reactor (Mono/Flux)
- RESTful API endpoints
- Integration with the `mock-service` testing library
- Clean separation of concerns (Controller, Service, Model layers)

## Technologies

- Java 17
- Spring Boot 3.5.3
- Spring WebFlux
- Project Reactor
- Lombok
- mock-service (testing library)

## Prerequisites

Before running this application, you need to install the `mock-service` dependency to your local Maven repository:

```bash
cd ../mock-service
mvn clean install
```

## Running the Application

```bash
cd mock-api
mvn spring-boot:run
```

The application will start on `http://localhost:8080`

## API Endpoints

### User Management

- **GET** `/api/users` - Get all users
- **GET** `/api/users/{id}` - Get user by ID
- **POST** `/api/users` - Create a new user
- **PUT** `/api/users/{id}` - Update an existing user
- **DELETE** `/api/users/{id}` - Delete a user

### Example Requests

#### Get all users
```bash
curl http://localhost:8080/api/users
```

#### Get user by ID
```bash
curl http://localhost:8080/api/users/1
```

#### Create a new user
```bash
curl -X POST http://localhost:8080/api/users \
  -H "Content-Type: application/json" \
  -d '{"name":"John Doe","email":"john@example.com"}'
```

#### Update a user
```bash
curl -X PUT http://localhost:8080/api/users/1 \
  -H "Content-Type: application/json" \
  -d '{"name":"Jane Doe","email":"jane@example.com"}'
```

#### Delete a user
```bash
curl -X DELETE http://localhost:8080/api/users/1
```

## Testing with mock-service

This project demonstrates how to use the `mock-service` library for testing reactive services. The library allows you to:

1. Define mock behaviors in JSON configuration files
2. Avoid writing verbose Mockito code
3. Test complex reactive flows with Mono and Flux types

### Example Test

See `src/test/java/net/mcfarb/mockapi/service/UserServiceTest.java` for a complete example of:
- Setting up `MonoMockProvider`
- Configuring JSON-based mocks in `src/test/resources/mockdata/UserServiceTest.json`
- Testing reactive service methods with StepVerifier

### Running Tests

```bash
mvn test
```

## Fallback to Real Endpoints

The mock-api supports automatic fallback to real endpoints when no mock configuration is found. This is useful for:
- Partial mocking (mock some endpoints, proxy others to real services)
- Development environments where some services are mocked and others are real
- Testing scenarios where you want to mix mocked and real responses

### Configuring Fallback

Add the following to your `application.properties`:

```properties
# Enable fallback functionality
mock.api.fallback.enabled=true

# Global fallback base URL (applies to all controllers by default)
mock.api.fallback.base-url=http://localhost:9090

# Optional: Timeout for fallback requests (default: 30000ms)
mock.api.fallback.timeout-ms=30000

# Optional: Forward request headers to fallback endpoint (default: true)
mock.api.fallback.forward-headers=true
```

### Per-Controller Fallback URLs

You can configure different fallback URLs for each controller:

```properties
# Enable fallback
mock.api.fallback.enabled=true

# Default fallback URL
mock.api.fallback.base-url=http://localhost:9090

# Override fallback URL for specific controllers
mock.api.controllers.user.fallback-url=http://localhost:9091
mock.api.controllers.product.fallback-url=http://localhost:9092
```

### Fallback Priority

The system determines the fallback URL using the following priority:

1. **Code-based override**: Controller overrides `getFallbackUrl()` method
2. **Configuration-based controller-specific**: `mock.api.controllers.{name}.fallback-url`
3. **Global configuration**: `mock.api.fallback.base-url`

### Example: Code-Based Fallback Override

```java
@RestController
@RequestMapping("/api/user")
public class UserController extends BaseRestController {

    @Override
    protected String getBasePath() {
        return "api/user";
    }

    @Override
    protected String getConfigFileName() {
        return "user";
    }

    @Override
    protected String getFallbackUrl() {
        // Override fallback URL for this controller
        return "http://user-service.example.com";
    }
}
```

### How Fallback Works

1. Request comes to a controller endpoint (e.g., `GET /api/user/123`)
2. System checks for matching mock configuration
3. If mock found → return mocked response
4. If no mock found and fallback enabled → proxy request to fallback URL
5. If no mock found and fallback disabled → return 404

### Fallback Features

- **Method preservation**: Proxies the same HTTP method (GET, POST, PUT, etc.)
- **Query parameters**: Forwards all query parameters to the fallback endpoint
- **Request headers**: Optionally forwards headers (configurable)
- **Request body**: Forwards body for POST/PUT/PATCH requests
- **Error handling**: Returns 502 Bad Gateway if fallback endpoint fails

## Project Structure

```
mock-api/
├── src/
│   ├── main/
│   │   ├── java/net/mcfarb/mockapi/
│   │   │   ├── controller/
│   │   │   │   └── UserController.java
│   │   │   ├── model/
│   │   │   │   └── User.java
│   │   │   ├── service/
│   │   │   │   └── UserService.java
│   │   │   └── MockApiApplication.java
│   │   └── resources/
│   │       └── application.properties
│   └── test/
│       ├── java/net/mcfarb/mockapi/
│       │   └── service/
│       │       └── UserServiceTest.java
│       └── resources/
│           └── mockdata/
│               └── UserServiceTest.json
└── pom.xml
```

## Building the Project

```bash
mvn clean package
```

This will create a JAR file in the `target/` directory.

## License

TBD
