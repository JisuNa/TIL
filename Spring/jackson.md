# Jackson

Jackson은 JAVA 진영에서 대표적인 json 라이브러리

## 스프링과 Jackson

org.springframework.boot:spring-boot-starter-web 에는 jackson을 기본적으로 포함하고 있다.

Spring에서 request body와 객체 간의 변환을 `HttpMessageConverter` 인터페이스를 통해서 제공한다.
`@RequestBody`와 `@ResponseBody`가 없으면 View Resolver가 사용된다.

## 직렬화와 역직렬화

Java Object를 Json으로 파싱하는 것을 직렬화(serialize)라고 하며, 반대는 역직렬화(deserialize)라고 합니다.

## 직렬화/역직렬화 대상

### A Public Filed
필드를 가장 간단하게 직렬화/역직렬화를 가능하게 하는 방법은 public으로 선언하는 것이다.

```java
public class Person {
    public String name;

    public Person() {
    }

    public Person(String name) {
        this.name = name;
    }
}
```
필드가 public 이라면 직렬화/역직렬화 대상이다.

### getter를 가진 non-public 필드
직렬화 역직렬화 모두 가능하다. getter가 있는 필드는 property로 간주한다.

```java
public class Person {
    private String name;

    public Person() {
    }
	
    public Person(String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```
getter를 통해서 직렬화가 된다.
역직렬화는 기본 생성자가 있어야 할 수 있다.

만약 기본 생성자 없이 역직렬화를 해야한다면 `@JsonCreator`와 `@JsonProperty`를 통해서 가능하다.

```java
public class Person {
    private String name;

    @JsonCreator
    public Person(@JsonProperty("name") String name) {
        this.name = name;
    }

    public String getName() {
        return name;
    }
}
```

### setter를 가진 non-public 필드
setter 메서드는 non-public 필드의 역직렬화를 가능하게 한다.

```java
public class Person {
    private String name;

    public void setName(String name) {
        this.name = name;
    }

    public String accessName() {
        return this.name;
    }
}
```

setter만으로는 직렬화는 할 수 없다.