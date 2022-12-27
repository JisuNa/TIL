# Dependency

## 의존성이란

객체가 협력을 한다는 것은 객체 간의 의존성이 존재한다는 뜻이다.

파라미터나 리턴값 또는 지역변수 등으로 다른 객체를 참조

## 컴파일타임 의존성

코드를 컴파일하는 시점에 결정되는 의존성이며, 클래스 사이의 의존성에 해당한다.

추상화된 클래스나 인터페이스가 아닌 구체 클래스에 의존하면 컴파일타임 의존성을 갖게된다.

```java
@Component
public class SHA256PasswordEncoder {
    public String encryptPassword(String pw)  {
        //...
    }
}

public class MemberService {

    private final SHA256PasswordEncoder passwordEncoder;

    @Transactional
    public void signUp(String email, String pw) {
        String encryptedPassword = passwordEncoder.encryptPassword(pw);

        //...
    }
}
```

결합도가 높다.

## 런타임 의존성

코드를 실행하는 시점에 결정되는 의존성이며, 객체 사이의 의존성에 해당한다.

추상화된 클래스나 인터페이스에 의존할 때 런타임 의존성을 갖게 되고 결합도가 낮게된다.

```java
public interface PasswordEncoder {
    String encryptPassword(final String pw);
}

@Component
public class SHA256PasswordEncoder implements PasswordEncoder {
    @Override
    public String encryptPassword(final String pw)  {
		//...
    }
}

public class MemberService {

    private final PasswordEncoder passwordEncoder;

    @Transactional
    public void signUp(String email, String pw) {
        String encryptedPassword = passwordEncoder.encryptPassword(pw);

        //...
    }
}
```

이렇게 런타임 의존성을 갖게될 경우 결합도가 낮다.

만약 `SHA256PasswordEncoder`의 취약점이 발견되거나 새로 구현해야할 경우 `MemberService`는 수정이 필요하지 않다.

```java
@Component
public class BCryptPasswordEncoder implements PasswordEncoder { 
    @Override
    public String encryptPassword(final String pw)  {
        //...
    }
}
```

## Conclusion

결합도가 높은 컴파일타임 의존성보다는 결합도가 낮은 런타임 의존성을 사용하자.