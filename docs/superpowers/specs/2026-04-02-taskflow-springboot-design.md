# Taskflow — Spring Boot 练习项目设计文档

**日期：** 2026-04-02
**背景：** Go 转 Spring Boot，有后端架构基础，目标是通过手写练熟 Spring API
**技术栈：** Spring Boot + Maven + MySQL + JPA/Hibernate + JWT

---

## 一、项目定位

`taskflow` 是一个后端 API 服务，支持：
- 用户注册 / 登录（JWT 签发）
- 任务 CRUD（创建、查询、修改、删除）
- 任务状态流转（TODO → IN_PROGRESS → DONE / CANCELLED）
- 按状态、负责人分页查询任务

**任务状态机：**
```
TODO  →  IN_PROGRESS  →  DONE
              ↓
           CANCELLED
```
状态只能按规则流转，非法流转抛业务异常。

---

## 二、目录结构

```
taskflow/
├── pom.xml
└── src/main/java/com/example/taskflow/
    ├── TaskflowApplication.java
    ├── config/
    │   ├── SecurityConfig.java
    │   └── JwtConfig.java
    ├── filter/
    │   └── JwtAuthFilter.java
    ├── user/
    │   ├── UserController.java
    │   ├── UserService.java
    │   ├── UserRepository.java
    │   ├── User.java
    │   └── dto/
    │       ├── RegisterRequest.java
    │       ├── LoginRequest.java
    │       └── UserResponse.java
    ├── task/
    │   ├── TaskController.java
    │   ├── TaskService.java
    │   ├── TaskRepository.java
    │   ├── Task.java
    │   ├── TaskStatus.java
    │   └── dto/
    │       ├── CreateTaskRequest.java
    │       ├── UpdateTaskRequest.java
    │       └── TaskResponse.java
    └── common/
        ├── exception/
        │   ├── AppException.java
        │   └── GlobalExceptionHandler.java
        ├── response/
        │   └── ApiResponse.java
        └── util/
            └── JwtUtil.java
```

**各层职责：**

| 层 | 职责 | 不该做的事 |
|----|------|-----------|
| Controller | 接参数、调 Service、返回响应 | 不写业务逻辑 |
| Service | 业务规则、事务控制 | 不直接写 SQL |
| Repository | 数据库读写 | 不写业务判断 |
| Entity | 映射数据库表结构 | 不暴露给前端 |
| DTO | 入参/出参的数据结构 | 不含业务逻辑 |
| Filter | 请求预处理（认证） | 不写业务逻辑 |

---

## 三、数据库表设计

### users 表
| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT PK AUTO_INCREMENT | 主键 |
| username | VARCHAR(50) UNIQUE NOT NULL | 用户名 |
| password | VARCHAR(255) NOT NULL | BCrypt 加密后的密码 |
| email | VARCHAR(100) UNIQUE | 邮箱（可选） |
| created_at | DATETIME | 创建时间 |

### tasks 表
| 字段 | 类型 | 说明 |
|------|------|------|
| id | BIGINT PK AUTO_INCREMENT | 主键 |
| title | VARCHAR(200) NOT NULL | 任务标题 |
| description | TEXT | 任务描述 |
| status | VARCHAR(20) NOT NULL | 枚举值字符串 |
| assignee_id | BIGINT FK → users.id | 负责人 |
| creator_id | BIGINT FK → users.id | 创建人 |
| due_date | DATE | 截止日期 |
| created_at | DATETIME | 创建时间 |
| updated_at | DATETIME | 更新时间 |

---

## 四、API 接口一览

### 用户模块（无需认证）
| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /api/auth/register | 注册 |
| POST | /api/auth/login | 登录，返回 JWT |

### 用户模块（需认证）
| 方法 | 路径 | 说明 |
|------|------|------|
| GET | /api/users/me | 获取当前登录用户信息 |
| GET | /api/users/{id} | 查询指定用户 |

### 任务模块（需认证）
| 方法 | 路径 | 说明 |
|------|------|------|
| POST | /api/tasks | 创建任务 |
| GET | /api/tasks/{id} | 查询单个任务 |
| GET | /api/tasks | 分页查询（支持按 status / assigneeId 过滤） |
| PUT | /api/tasks/{id} | 修改任务（标题、描述、截止日期） |
| PATCH | /api/tasks/{id}/status | 流转任务状态 |
| DELETE | /api/tasks/{id} | 删除任务（仅创建人可删） |

---

## 五、统一响应结构

所有接口返回：
```json
{
  "code": 200,
  "message": "success",
  "data": { ... }
}
```

错误示例：
```json
{
  "code": 400,
  "message": "非法状态流转：DONE 不能变更为 TODO",
  "data": null
}
```

---

## 六、分步开发指南（核心）

### Step 1 — 项目初始化（15 min）

**目标：** 用 Spring Initializr 生成项目，连通 MySQL，跑起来

**涉及知识点：**
- `@SpringBootApplication`
- `application.yml` 配置数据源
- Maven 依赖管理

**需要查的关键词：**
- `spring initializr`
- `spring.datasource` 配置项
- `spring.jpa.hibernate.ddl-auto`

**关键依赖（填入 pom.xml）：**
```xml
spring-boot-starter-web
spring-boot-starter-data-jpa
spring-boot-starter-validation
mysql-connector-j
jjwt-api / jjwt-impl / jjwt-jackson   <!-- JWT 库，version: 0.11.x -->
```

**验收标准：** 启动不报错，MySQL 连通，JPA 自动建表

---

### Step 2 — 统一响应结构 & 全局异常（20 min）

**目标：** 写好 `ApiResponse<T>` 和 `GlobalExceptionHandler`，后续所有接口复用

**涉及知识点：**
- 泛型类设计
- `@RestControllerAdvice`
- `@ExceptionHandler`

**需要查的关键词：**
- `@RestControllerAdvice` Spring 文档
- `ResponseEntity` 用法
- `@ExceptionHandler` 指定异常类型

**函数签名示例（只给签名，逻辑自己填）：**
```java
// ApiResponse.java
public class ApiResponse<T> {
    private int code;
    private String message;
    private T data;

    public static <T> ApiResponse<T> success(T data) { ... }
    public static ApiResponse<Void> error(int code, String message) { ... }
}

// GlobalExceptionHandler.java
@RestControllerAdvice
public class GlobalExceptionHandler {
    @ExceptionHandler(AppException.class)
    public ResponseEntity<ApiResponse<Void>> handleAppException(AppException e) { ... }

    @ExceptionHandler(MethodArgumentNotValidException.class)
    public ResponseEntity<ApiResponse<Void>> handleValidation(MethodArgumentNotValidException e) { ... }
}
```

**验收标准：** 故意触发一个异常，返回格式统一的 JSON 错误响应

---

### Step 3 — 用户实体 & Repository（20 min）

**目标：** 写 `User` 实体类，映射 users 表，写 `UserRepository`

**涉及知识点：**
- `@Entity` / `@Table` / `@Column`
- `@Id` / `@GeneratedValue`
- `@CreationTimestamp`
- `JpaRepository<T, ID>` 泛型继承

**需要查的关键词：**
- JPA `@Entity` 注解说明
- `@GeneratedValue(strategy = GenerationType.IDENTITY)`
- `JpaRepository` 内置方法列表
- `Optional<User> findByUsername(String username)` — Spring Data 方法命名规则

**函数签名示例：**
```java
// UserRepository.java
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByUsername(String username);
    boolean existsByUsername(String username);
}
```

**验收标准：** 启动后 users 表自动创建，字段与设计一致

---

### Step 4 — 注册 & 登录接口（30 min）

**目标：** 实现 `/api/auth/register` 和 `/api/auth/login`，登录返回 JWT

**涉及知识点：**
- `@RestController` / `@RequestMapping` / `@PostMapping`
- `@RequestBody` / `@Valid`
- `BCryptPasswordEncoder` 密码加密
- `@Bean` 注入工具类
- JWT 生成（`Jwts.builder()`）

**需要查的关键词：**
- `BCryptPasswordEncoder` Spring Security Crypto（不用引入完整 Security）
- `jjwt Jwts.builder()` 用法
- `@Bean` 在 `@Configuration` 类中声明

**函数签名示例：**
```java
// UserService.java
public UserResponse register(RegisterRequest request) { ... }
public String login(LoginRequest request) { ... }  // 返回 JWT token

// JwtUtil.java
public String generateToken(Long userId, String username) { ... }
public Claims parseToken(String token) { ... }
```

**验收标准：** Postman 调用登录接口，拿到 JWT token 字符串

---

### Step 5 — JWT Filter & Security 配置（30 min）

**目标：** 写 `JwtAuthFilter`，每次请求自动验证 token，注入当前用户到上下文

**涉及知识点：**
- `OncePerRequestFilter`（Spring 提供的 Filter 基类）
- `SecurityContextHolder` 存储当前用户
- `UsernamePasswordAuthenticationToken`
- `SecurityConfig` 配置放行路径

**需要查的关键词：**
- `OncePerRequestFilter` 用法
- `SecurityContextHolder.getContext().setAuthentication()`
- Spring Security `SecurityFilterChain` Bean 配置（Spring Boot 3.x 写法）
- `http.authorizeHttpRequests()` 放行 `/api/auth/**`

**函数签名示例：**
```java
// JwtAuthFilter.java
@Component
public class JwtAuthFilter extends OncePerRequestFilter {
    @Override
    protected void doFilterInternal(HttpServletRequest request,
                                    HttpServletResponse response,
                                    FilterChain filterChain) throws ... { ... }
}

// 获取当前登录用户（在 Service 中用）
Long currentUserId = (Long) SecurityContextHolder
    .getContext().getAuthentication().getPrincipal();
```

**验收标准：** 不带 token 访问 `/api/users/me` 返回 401；带正确 token 返回用户信息

---

### Step 6 — 任务实体 & Repository（20 min）

**目标：** 写 `Task` 实体，建立与 `User` 的外键关联，写 `TaskStatus` 枚举

**涉及知识点：**
- `@ManyToOne` / `@JoinColumn` 关联映射
- `@Enumerated(EnumType.STRING)` 枚举持久化
- `@UpdateTimestamp` 自动更新时间

**需要查的关键词：**
- JPA `@ManyToOne` 懒加载 vs 急加载（`fetch = FetchType.LAZY`）
- `@Enumerated` 存 STRING 还是 ORDINAL 的区别
- Spring Data `findByAssigneeId` 命名规则

**函数签名示例：**
```java
// TaskRepository.java
public interface TaskRepository extends JpaRepository<Task, Long> {
    Page<Task> findByAssigneeId(Long assigneeId, Pageable pageable);
    Page<Task> findByStatus(TaskStatus status, Pageable pageable);
    Page<Task> findByAssigneeIdAndStatus(Long assigneeId, TaskStatus status, Pageable pageable);
}
```

**验收标准：** 启动后 tasks 表自动创建，外键约束正确

---

### Step 7 — 任务 CRUD 接口（30 min）

**目标：** 实现创建、查询单个、修改基本信息、删除任务

**涉及知识点：**
- `@PathVariable` / `@RequestBody`
- `@Transactional` 事务注解
- `@PreAuthorize` 或手动鉴权（只有创建人才能删除）
- Service 层调用 Repository，转换 Entity → DTO

**需要查的关键词：**
- `@Transactional` 加在 Service 方法上的时机
- `orElseThrow()` 配合 `Optional` 用法
- Entity 转 DTO 的常见方式（手写 / MapStruct）

**函数签名示例：**
```java
// TaskService.java
public TaskResponse createTask(CreateTaskRequest request, Long creatorId) { ... }
public TaskResponse getTask(Long taskId) { ... }
public TaskResponse updateTask(Long taskId, UpdateTaskRequest request, Long currentUserId) { ... }
public void deleteTask(Long taskId, Long currentUserId) { ... }
```

**验收标准：** CRUD 四个接口均可通过 Postman 测试通过

---

### Step 8 — 状态流转接口（30 min）

**目标：** 实现 `PATCH /api/tasks/{id}/status`，加入状态机校验逻辑

**涉及知识点：**
- 业务异常设计（`AppException`）
- `@Transactional` 保证状态更新原子性
- 枚举方法（在枚举里写校验逻辑）

**需要查的关键词：**
- Java enum 定义方法
- `@Transactional` 隔离级别（了解即可）
- 业务异常 vs 系统异常的设计原则

**函数签名示例：**
```java
// TaskStatus.java
public enum TaskStatus {
    TODO, IN_PROGRESS, DONE, CANCELLED;

    // 在枚举里写流转规则
    public boolean canTransitionTo(TaskStatus next) { ... }
}

// TaskService.java
@Transactional
public TaskResponse transitionStatus(Long taskId, TaskStatus newStatus, Long currentUserId) {
    // 1. 查任务
    // 2. 调用 canTransitionTo 校验
    // 3. 不合法 → throw new AppException(...)
    // 4. 合法 → 更新保存
}
```

**验收标准：** `DONE → TODO` 返回 400 业务错误；合法流转正确更新数据库

---

### Step 9 — 分页查询接口（20 min）

**目标：** 实现 `GET /api/tasks?status=TODO&assigneeId=1&page=0&size=10`

**涉及知识点：**
- `Pageable` 接口 & `PageRequest`
- `@RequestParam(required = false)` 可选参数
- `Page<T>` 返回结构（含 totalPages、totalElements）
- 动态条件查询（多条件组合）

**需要查的关键词：**
- Spring Data `Pageable` 用法
- `Page<Task>` 转 `Page<TaskResponse>` 方法：`page.map(...)`
- `JpaSpecificationExecutor` 或多方法命名查询处理可选条件

**函数签名示例：**
```java
// TaskController.java
@GetMapping
public ApiResponse<?> listTasks(
        @RequestParam(required = false) TaskStatus status,
        @RequestParam(required = false) Long assigneeId,
        @RequestParam(defaultValue = "0") int page,
        @RequestParam(defaultValue = "10") int size) { ... }
```

**验收标准：** 不同参数组合（只传 status、只传 assigneeId、都传、都不传）均返回正确分页结果

---

### Step 10 — 收尾：参数校验 & 优化（20 min）

**目标：** 给所有 DTO 加上 Bean Validation 注解，完善错误信息

**涉及知识点：**
- `@NotBlank` / `@NotNull` / `@Size` / `@Email`
- `@Valid` 在 Controller 方法参数上触发校验
- `GlobalExceptionHandler` 中处理 `MethodArgumentNotValidException`

**需要查的关键词：**
- Jakarta Bean Validation 注解列表（Spring Boot 3.x 用 `jakarta.validation`）
- `BindingResult` vs 直接抛异常的区别
- `@Validated` vs `@Valid` 区别

**函数签名示例：**
```java
// RegisterRequest.java
public class RegisterRequest {
    @NotBlank(message = "用户名不能为空")
    @Size(min = 3, max = 20)
    private String username;

    @NotBlank
    @Size(min = 6, message = "密码至少6位")
    private String password;
}
```

**验收标准：** 传入空用户名注册，返回包含字段错误信息的 400 响应

---

## 七、学习节奏建议

| 阶段 | 步骤 | 预计时间 | 核心收获 |
|------|------|---------|---------|
| 地基 | Step 1-2 | 1h | 项目骨架、错误处理 |
| 用户 | Step 3-4 | 1h | JPA 基础、接口开发 |
| 认证 | Step 5 | 30min | Filter 链、Security |
| 任务 | Step 6-7 | 1h | 关联映射、事务 |
| 业务 | Step 8-9 | 1h | 状态机、分页查询 |
| 收尾 | Step 10 | 20min | 参数校验 |
| **合计** | | **约 5h** | |

---

## 八、贴近真实公司开发的几个习惯

1. **Entity 不对外暴露** — Controller 永远返回 DTO，不返回 `User` / `Task` 对象
2. **Service 层加 `@Transactional`** — 写操作（增改删）的 Service 方法都加
3. **Repository 不写业务逻辑** — 复杂条件查询用方法命名或 `@Query`，不在 Service 里拼 SQL
4. **统一异常处理** — 业务错误用自定义异常 + `@RestControllerAdvice`，不在 Controller 里 try-catch
5. **密码绝不明文存储** — 注册时 `BCryptPasswordEncoder.encode()`，登录时 `matches()`

---

*设计文档版本：v1.0 | 2026-04-02*
