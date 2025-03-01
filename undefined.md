---
description: Controller 계층과 Service 계층 메서드 분리 원칙에 대해 알아보자
---

# 메서드 분리 원칙

## 메서드 분리 원칙 (Spring Boot Best Practice)

Spring Boot에서 **컨트롤러와 서비스의 메서드를 효과적으로 분리하는 원칙**을 알아본다.\
올바른 메서드 분리는 **코드의 가독성, 유지보수성, 테스트 용이성**을 높여줌.

***

### 📌 1. 메서드 분리를 해야 하는 이유

* **단일 책임 원칙 (SRP, Single Responsibility Principle)**\
  → 하나의 메서드는 하나의 역할만 수행해야 한다.
* **가독성 향상**\
  → 코드가 길어지는 것을 방지하고, 의미 있는 단위로 나눌 수 있다.
* **중복 코드 제거**\
  → 동일한 로직이 반복될 경우, 별도의 메서드로 분리하여 재사용성을 높일 수 있다.
* **테스트 용이성 증가**\
  → 작은 단위로 분리하면 개별적인 단위 테스트가 가능해진다.

***

### 📌 2. 컨트롤러에서의 메서드 분리 원칙

**컨트롤러는 HTTP 요청을 처리하는 역할**만 담당해야 하며, 다음과 같은 경우에만 메서드를 분리하는 것이 바람직하다.

✅ **분리할 수 있는 메서드 (컨트롤러 내부)**

* **공통적인 응답 처리** (`ResponseEntity` 변환)
* **DTO 변환**
* **요청 데이터 검증 및 변환 (`parse`, `validate` 등)**



✅ 올바른 컨트롤러 메서드 분리 예시

```java
@RestController
@RequestMapping("/users")
public class UserController {

    private final UserService userService;

    public UserController(UserService userService) {
        this.userService = userService;
    }

    @GetMapping("/{id}")
    public ResponseEntity<ApiResponse<UserDto>> getUserById(@PathVariable String id) {
        Long userId = parseUserId(id); // ✅ ID 변환 로직 분리
        return buildResponse(userService.getUserById(userId), "User retrieved successfully");
    }

    // ✅ URL 파라미터를 Long 타입으로 변환하는 private 메서드
    private Long parseUserId(String id) {
        try {
            return Long.parseLong(id);
        } catch (NumberFormatException e) {
            throw new InvalidRequestException("Invalid user ID format: " + id);
        }
    }

    // ✅ 공통 응답 처리를 위한 private 메서드
    private <T> ResponseEntity<ApiResponse<T>> buildResponse(T data, String message) {
        return ResponseEntity.ok(new ApiResponse<>(message, data));
    }
}
```

✅ **올바른 이유**

* **컨트롤러는 서비스 호출만 담당하고, 로직이 분리되어 있음**
* **공통 응답 형식을 `buildResponse()`에서 처리하여 코드 중복 제거**
* **잘못된 요청을 `parseUserId()`에서 예외 처리하여 코드 가독성 향상**

***

### 📌 3. 서비스 계층에서의 메서드 분리 원칙

서비스 계층은 **비즈니스 로직을 담당**해야 하므로, 다음과 같은 경우에 메서드를 분리해야 한다.

#### ✅ 분리할 수 있는 메서드 (서비스 내부)

* **데이터베이스 조회 로직 (`findById` 등)**
* **DTO 변환 로직 (`convertToDto` 등)**
* **비즈니스 로직 검증 (`validate` 등)**

#### ✅ 올바른 서비스 메서드 분리 예시

```java
@Service
public class UserServiceImpl implements UserService {

    private final UserRepository userRepository;

    public UserServiceImpl(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    @Override
    public UserDto getUserById(Long id) {
        User user = findUserById(id); // ✅ 데이터 조회 분리
        return convertToDto(user);    // ✅ DTO 변환 분리
    }

    @Override
    public UserDto createUser(UserDto userDto) {
        validateUser(userDto);         // ✅ 데이터 검증 분리
        User savedUser = userRepository.save(new User(userDto));
        return convertToDto(savedUser);
    }

    // ✅ 데이터베이스 조회를 위한 private 메서드
    private User findUserById(Long id) {
        return userRepository.findById(id)
                .orElseThrow(() -> new UserNotFoundException("User not found with id: " + id));
    }

    // ✅ DTO 변환을 위한 private 메서드
    private UserDto convertToDto(User user) {
        return new UserDto(user);
    }

    // ✅ 사용자 데이터 검증을 위한 private 메서드
    private void validateUser(UserDto userDto) {
        if (userDto.getEmail() == null || !userDto.getEmail().contains("@")) {
            throw new InvalidUserException("Invalid email format");
        }
    }
}
```

✅ **올바른 이유**

* **각 역할별로 메서드를 분리하여 코드의 응집도가 높아짐**
* **데이터 검증, 변환, 조회 등을 명확히 구분하여 유지보수 용이**
* **비즈니스 로직을 서비스 계층에서 처리하고, 컨트롤러는 단순히 호출만 하도록 설계**

***



### 📌 4. 예외 처리 시 메서드 분리 원칙

예외 처리는 **예외의 성격에 따라 컨트롤러 또는 서비스 계층에서 분리**해야 한다.

#### ✅ 예외 처리 위치 결정

| 예외 유형                                       | 컨트롤러에서 처리 | 서비스에서 처리 |
| ------------------------------------------- | --------- | -------- |
| 요청 데이터 검증 (`@RequestParam`, `@RequestBody`) | ✅         | ❌        |
| URL 파라미터 변환 오류 (`NumberFormatException`)    | ✅         | ❌        |
| 존재하지 않는 데이터 조회 (`UserNotFoundException`)    | ❌         | ✅        |
| 비즈니스 로직 관련 (`InvalidUserException`)         | ❌         | ✅        |

***

### 📌 5. 전역 예외 처리 (`@ControllerAdvice`)

#### ✅ 전역 예외 핸들러 예시

```java
@RestControllerAdvice
public class GlobalExceptionHandler {

    @ExceptionHandler(UserNotFoundException.class)
    public ResponseEntity<ApiResponse<String>> handleUserNotFoundException(UserNotFoundException ex) {
        return ResponseEntity.status(HttpStatus.NOT_FOUND)
                .body(new ApiResponse<>("User not found", ex.getMessage()));
    }

    @ExceptionHandler(InvalidRequestException.class)
    public ResponseEntity<ApiResponse<String>> handleInvalidRequestException(InvalidRequestException ex) {
        return ResponseEntity.status(HttpStatus.BAD_REQUEST)
                .body(new ApiResponse<>("Invalid request", ex.getMessage()));
    }
}
```

#### ✔ 전역 예외 처리를 사용하면 얻을 수 있는 장점

* ✅ **개별 컨트롤러에서 예외를 직접 처리할 필요가 없음**
* ✅ **모든 예외를 일관된 형식으로 클라이언트에게 응답 가능**
* ✅ **예외 처리 로직을 중앙 집중화하여 유지보수가 편리함**

***

### 🎯 결론: 좋은 메서드 분리 원칙

✅ **컨트롤러는 서비스만 호출하며, HTTP 요청을 다루는 역할만 수행해야 한다.**\
✅ **공통적인 응답 처리는 컨트롤러 내부의 private 메서드로 분리할 수 있다.**\
✅ **서비스 계층에서는 데이터 조회, 검증, 변환 등을 private 메서드로 분리하여 관리해야 한다.**\
✅ **전역 예외 처리 (`@ControllerAdvice`)를 활용하여 예외 처리 로직을 깔끔하게 유지한다.**
