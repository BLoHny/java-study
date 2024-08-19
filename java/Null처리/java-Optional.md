# Optional

Optional 이란 Java 8에 도입된 기능으로 기존의 Null처리, try-catch를 대체할 방법으로 고안되었습니다. <br>
Optional은 null 아닌 값을 포함할 수도 있고 포함하지 않을 수도 있는 컨테이너 객체라 표현되어있습니다. <br>


## 컨테이너 객체란?
- 다른 객체를 포함하거나 관리하는 객체
- 데이터 구조를 구성 
- 객체 그룹화
- 객체에 접근, 조작 메커니즘 제공

> EX) Java Collection. (List, Set.. )

## Optional이 NPE를 방지 하는 방법
```java
public final class Optional<T> {
    /**
     * If non-null, the value; if null, indicates no value is present
     */
    private final T value;
```

Optional은 Wrapper Class로, value 변수에 값을 저장하기 때문에 참조시 NPE를 방지 해준다.

## Optional의 주요 Method

1. **of()** : 
Optional 객체를 생성합니다. 값이 null이면 NullPointerException을 던집니다.

    ```java
    Optional<String> optional = Optional.of("Hello"); //예시에서 모두 이 변수를 사용합니다.
    // Optional<String> optional = Optional.of(null); // Throws NullPointerException
    ```
2. **ofNullable()** : 
값이 null일 수 있는 경우 Optional 객체를 생성합니다. 값이 null이면 빈 Optional 객체를 반환합니다.
    ```java
    Optional<String> optional = Optional.ofNullable("Hello");
    Optional<String> emptyOptional = Optional.ofNullable(null);
    ```

3. **empty()** :
빈 Optional 객체를 반환합니다.
    ```java
    Optional<String> emptyOptional = Optional.empty();
    ```

4. **isPresent()** :
값이 존재하면 true, 존재하지 않으면 false를 반환합니다.
    ```java
    if (optional.isPresent()) {
        System.out.println("Value is present");
    }
    ```

5. **ifPresent()** :
값이 존재하면 주어진 동작을 수행합니다.
    ```java
    optional.ifPresent(value -> System.out.println("Value: " + value));
    ```

6. **get()** : 
값이 존재하면 반환하고, 존재하지 않으면 NoSuchElementException을 던집니다.
    ```java
    String value = optional.get();
    ```

7. **orElse()** : 
값이 존재하면 반환하고, 존재하지 않으면 대체 값을 반환합니다.
    ```java
    String value = optional.orElse("Default Value");
    ```

8. **orElseGet()** : 
값이 존재하면 반환하고, 존재하지 않으면 대체 값을 제공하는 공급자를 호출하여 반환합니다.
    ```java
    String value = optional.orElseGet(() -> "Default Value from Supplier");
    ```

9. **orElseThrow()** : 
값이 존재하면 반환하고, 존재하지 않으면 주어진 예외를 던집니다.
    ```java
    String value = optional.orElseThrow(() -> new IllegalArgumentException("No value present"));
    ```

10. **map(), flatMap(), filter()** : 다양한 Stream Api 제공
    ```java
    //map() : 값이 존재하면 주어진 함수를 적용하여 반환하고, 존재하지 않으면 빈 Optional을 반환합니다.
    Optional<Integer> length = optional.map(String::length);

    //flatMap() : 값이 존재하면 주어진 함수에 의해 생성된 Optional을 반환하고, 존재하지 않으면 빈 Optional을 반환합니다.
    Optional<String> name = Optional.of("Hello");
    Optional<String> upperName = name.flatMap(n -> Optional.of(n.toUpperCase()));

    //filter() : 값이 존재하고 주어진 조건을 만족하면 그 값을 반환하고, 그렇지 않으면 빈 Optional을 반환합니다.
    Optional<String> longName = optional.filter(name -> name.length() > 5);
    ```

<br>

## Optional의 장점과 단점

다음 두 메서드가 있다고 가정을 하고 설명하겠습니다
```java
// UserRepository
User findById(Long id);
Optional<User> findById(Long id);
```

### 장점

- Return Type
    ```java
    public User execute() {
        User user = userRepository.findById(1L);
        if(account == null) {
            throw new UserNotFoundException();
        }
        return user;
    }

    public User execute() {
        return userRepository.findById(1L)
            .orElseThrow(UserNotFoundException::new);
    }
    ```

    다음과 같이 Repository에서 User 조회코드에서 불필요한 if문을 제거하고 사용자 정의 Exception을 쉽게 발생시킬 수 있습니다.
    <br>또한 추가적으로 다른 조건 검사가 필요한경우 Optional에서 제공하는 Stream을 사용하여 쉽게 코드를 작성할 수 있습니다.
        
    ```java
    public User execute() {
        User user = userRepository.findById(1L);
        if(account == null || user.getName() == null) {
            throw new UserNotFoundException();
        }
        return user;
    }

    public User execute() {
        return userRepository.findById(1L)
            .flatMap(User::getNameOptional)
            .orElseThrow(UserNotFoundException::new);
    }
    ```
    불필요한 If 조건을 줄이는데 용이합니다.


### 단점, 주의사항

- 오버헤드<br>
    Optional 자체도 결국은 객체를 생성하는 것이므로 간단한 field에 대한 optional wrapping 은 메모리적으로도 primitive type unwrap에 대한 오버헤드를 유발합니다

    ex)
    ```java
    int age = 0;
    Optional<Integer> test = Optional.of(age);
    ```

- 생성자, 메서드 등의 파라미터로 Optional을 사용하지 않아야 합니다.<br>
    ```java
    public void process(Optional<String> optionalParam) {
    // Method implementation
    }

    
    process(Optional.of("Hello"));
    process(Optional.empty());
    ```
        
    예시로 다음과 같은 method가 존재할때
    호출시 Optional을 명시적으로 호출해야합니다. <br>
    이는 의미없는 메모리 사용입니다. Optional을 사용한다 하여 null을 그대로 전달하면 NPE의 발생은 똑같으므로
    매개변수에서는 if를 사용하는게 좀 더 효율적입니다.