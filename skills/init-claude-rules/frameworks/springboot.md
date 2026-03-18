# Spring Boot Guidelines (Kotlin)

## Layered Architecture

```
controller/  → Receives requests, validates parameters, calls services, returns responses
service/     → Business logic, injects repositories
repository/  → Data access, Spring Data JPA/MongoDB
model/       → Entity classes
dto/         → Data Transfer Objects (request/response)
config/      → Configuration classes
```

- Controllers must not contain business logic; they only validate parameters and delegate
- Services must not depend on Controller-layer types (HttpServletRequest, etc.)
- Repositories must not contain business logic; they only perform data queries

## Controller

- Follow RESTful conventions: use plural nouns for resources, HTTP methods for actions
- Use a unified response format; do not mix bare data and wrapped objects across endpoints
- Use `@Valid` + JSR 303 annotations for parameter validation; do not write manual if-checks
- Use the `Pageable` parameter for paginated queries

```kotlin
@RestController
@RequestMapping("/api/v1/users")
class UserController(
    private val userService: UserService
) {
    @GetMapping("/{id}")
    fun getUser(@PathVariable id: Long): ApiResponse<UserDto> {
        return ApiResponse.ok(userService.getById(id))
    }

    @PostMapping
    fun createUser(@Valid @RequestBody request: CreateUserRequest): ApiResponse<UserDto> {
        return ApiResponse.ok(userService.create(request))
    }
}
```

## Service

- Separate interface and implementation (`UserService` interface + `UserServiceImpl`)
- Apply the `@Transactional` annotation on Service methods
- Use `@Transactional(readOnly = true)` for read-only operations
- Services may call each other, but circular dependencies are forbidden

## DTO

- Name request DTOs `XxxRequest`, response DTOs `XxxDto`
- Returning entity classes directly to the frontend is forbidden (exposes internal structure, causes lazy-loading issues)
- Encapsulate entity-to-DTO conversion as extension functions

```kotlin
// Extension function for entity-to-DTO conversion
fun User.toDto() = UserDto(
    id = id,
    name = name,
    email = email
)
```

## Exception Handling

- Use `@RestControllerAdvice` for global exception handling
- Business exceptions should extend a custom base class and carry an error code
- Do not use try-catch in Controllers; let exceptions propagate to the global handler

```kotlin
abstract class BusinessException(
    val code: String,
    override val message: String
) : RuntimeException(message)

class UserNotFoundException(id: Long) :
    BusinessException("USER_NOT_FOUND", "User not found: $id")
```

## Configuration

- Use `@ConfigurationProperties` for configuration; do not use scattered `@Value` annotations
- Sensitive configuration (passwords, keys) must not be committed to the repository
- Use `application-{profile}.yml` to differentiate between environments

## Dependency Injection

- Prefer constructor injection (Kotlin primary constructors support this naturally)
- Field injection (`@Autowired` on fields) is forbidden
- Use `lateinit` or `by lazy` when lazy initialization is needed
