# Spring Boot 规范（Kotlin）

## 分层架构

```
controller/  → 接收请求、参数校验、调用 service、返回响应
service/     → 业务逻辑，注入 repository
repository/  → 数据访问，Spring Data JPA/MongoDB
model/       → 实体类
dto/         → 数据传输对象（请求/响应）
config/      → 配置类
```

- Controller 不写业务逻辑，只做参数校验和转发
- Service 不依赖 Controller 层的类型（HttpServletRequest 等）
- Repository 不写业务逻辑，只做数据查询

## Controller

- RESTful 风格：资源用名词复数，动作用 HTTP 方法
- 统一返回格式，不要有的接口返回裸数据有的返回包装对象
- 参数校验用 `@Valid` + JSR 303 注解，不要手写 if 判断
- 分页查询用 `Pageable` 参数

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

- 接口 + 实现分离（`UserService` 接口 + `UserServiceImpl`）
- 事务注解 `@Transactional` 加在 Service 方法上
- 只读操作用 `@Transactional(readOnly = true)`
- Service 之间可以互相调用，但禁止循环依赖

## DTO

- 请求用 `XxxRequest`，响应用 `XxxDto`
- 禁止直接把实体类返回给前端（泄露内部结构、懒加载问题）
- 实体和 DTO 之间的转换封装为扩展函数

```kotlin
// 实体转 DTO 的扩展函数
fun User.toDto() = UserDto(
    id = id,
    name = name,
    email = email
)
```

## 异常处理

- 用 `@RestControllerAdvice` 全局处理异常
- 业务异常继承自定义基类，携带错误码
- 禁止在 Controller 里 try-catch，让异常自然抛到全局处理器

```kotlin
abstract class BusinessException(
    val code: String,
    override val message: String
) : RuntimeException(message)

class UserNotFoundException(id: Long) :
    BusinessException("USER_NOT_FOUND", "用户不存在: $id")
```

## 配置

- 配置项用 `@ConfigurationProperties`，不要散落的 `@Value`
- 敏感配置（密码、密钥）不提交到代码仓库
- 不同环境用 `application-{profile}.yml` 区分

## 依赖注入

- 优先构造器注入（Kotlin 主构造器天然支持）
- 禁止字段注入（`@Autowired` 在字段上）
- 需要延迟初始化用 `lateinit` 或 `by lazy`
