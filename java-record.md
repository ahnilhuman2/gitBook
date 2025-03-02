# JAVA record 클래스

### **1. `record` 클래스란?**

Java 14에서 **프리뷰 기능**으로 도입되었으며, Java 16에서 정식 기능이 됨.\
이는 **불변(immutable) 데이터 객체**를 간결하게 정의하기 위한 기능

***

### **2. `record` 클래스의 특징**

* **불변 객체 (Immutable)** → 필드 값 변경 불가능
* **자동 생성되는 메서드**
  * 생성자(Constructor)
  * `getter` (필드명과 동일한 메서드)
  * `toString()`
  * `equals()` 및 `hashCode()`
* **`final` 클래스** → 상속 불가

***

### **3. `record` 클래스 선언 방법**

#### **(1) 일반적인 Java 클래스로 데이터 객체 만들기(lombok 사용안할시)**

```java
public class Person {
    private final String name;
    private final int age;

    public Person(String name, int age) {
        this.name = name;
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public int getAge() {
        return age;
    }

    @Override
    public String toString() {
        return "Person{name='" + name + "', age=" + age + "}";
    }

    @Override
    public boolean equals(Object obj) {
        if (this == obj) return true;
        if (obj == null || getClass() != obj.getClass()) return false;
        Person person = (Person) obj;
        return age == person.age && Objects.equals(name, person.name);
    }

    @Override
    public int hashCode() {
        return Objects.hash(name, age);
    }
}
```

***

#### **(2) `record` 클래스를 사용한 데이터 객체**

```java
public record Person(String name, int age) {}
```

위 코드만으로도 다음 기능이 자동으로 생성:

* `name`과 `age` 필드 (private `final`)
* 기본 생성자: `Person(String name, int age)`
* `getter`: `name()`과 `age()`
* `toString()`
* `equals()`
* `hashCode()`

***

### **4. `record` 클래스의 활용**

#### **(1) 기본 사용**

```java
public class RecordExample {
    public static void main(String[] args) {
        Person person1 = new Person("Alice", 25);
        Person person2 = new Person("Alice", 25);
        
        System.out.println(person1);  // Person[name=Alice, age=25]
        System.out.println(person1.name());  // Alice
        System.out.println(person1.age());   // 25
        
        System.out.println(person1.equals(person2));  // true
    }
}
```

***

#### **(2) 사용자 정의 생성자**

```java
public record Person(String name, int age) {
    public Person {
        if (age < 0) {
            throw new IllegalArgumentException("Age cannot be negative.");
        }
    }
}
```

***

#### **(3) 추가 메서드 작성**

```java
public record Person(String name, int age) {
    public String greet() {
        return "Hello, my name is " + name + " and I am " + age + " years old.";
    }
}
```

사용 예시:

```java
Person p = new Person("Alice", 25);
System.out.println(p.greet());
// 출력: Hello, my name is Alice and I am 25 years old.
```

***

### **5. `record` 클래스의 한계**

1.  **필드 값 변경 불가**

    ```java
    Person p = new Person("Alice", 25);
    // p.age = 30;  // 컴파일 오류!
    ```
2.  **상속 불가**

    ```java
    public class Student extends Person {}  // 오류!
    ```
3. **JavaBean 스타일이 아님** → `setName()` 같은 setter 메서드를 제공하지 않음.
4. **모든 필드는 반드시 선언과 동시에 초기화해야 함.**

***

### **6. `record` 클래스와 DTO (Data Transfer Object)**

`record` 클래스는 **DTO (데이터 전송 객체)** 로 자주 사용.

```java
public record UserDTO(String username, String email) {}
```

Spring Boot에서 `@RequestBody`를 통해 JSON 데이터를 받을 때:

```java
@PostMapping("/users")
public ResponseEntity<String> createUser(@RequestBody UserDTO userDTO) {
    return ResponseEntity.ok("User " + userDTO.username() + " created.");
}
```

***

### **7. `record` vs. `class` vs. `@Value` (Lombok)**

| 비교 항목                           | `record`   | `class` + `@Value` (Lombok) | `class` (일반) |
| ------------------------------- | ---------- | --------------------------- | ------------ |
| 코드 간결성                          | ✅ (가장 간결함) | ✅ (Lombok 사용 시 간결)          | ❌ (많은 코드 필요) |
| 불변성(Immutable)                  | ✅          | ✅ (`@Value` 사용 시)           | ❌            |
| `equals()` & `hashCode()` 자동 생성 | ✅          | ✅                           | ❌ (직접 구현 필요) |
| `toString()` 자동 생성              | ✅          | ✅                           | ❌ (직접 구현 필요) |
| 상속 가능 여부                        | ❌ (불가능)    | ❌ (`@Value`는 `final`)       | ✅ (가능)       |

***

### **8. 결론**

✅ `record` 클래스는 **불변 데이터 객체**를 간결하게 표현할 때 매우 유용.\
✅ Java 16부터 정식 지원되므로, DTO, Value Object, 간단한 데이터 저장용 클래스로 활용할 수 있음.\
⚠️ 하지만 **상속이 필요하거나, 필드 값을 변경해야 하는 경우에는 일반 `class`를 사용해야 함.**

✅ `record` 클래스는 별도의 의존성 없이 사용 가능하지만, Lombok의 `@Value`는 외부 라이브러리를 필요로 함\
✅ Lombok은 기존 클래스를 유지하면서 불변성을 부여할 수 있기 때문에, 기존 코드베이스와의 호환성이 중요할 경우 Lombok이 더 유용할 수 있다.
