# Serializable

## Overview

이 문서에서는 Java 의 Serializable 의 동작방식을 알아봅니다.

## Serializable 의 기능과 동작방식

Serializable 은 객체를 직렬화할 때 사용합니다.

```java
public interface Serializable {
}
```

어떤 기능이 있는지 확인해보면 아무것도 없습니다.

그럼 어떻게 동작하는 걸까요?

먼저 객체를 파일로 저장하는 코드를 작성해보겠습니다.

```kotlin
@Test
fun test() {
    val file = File("text.txt")
    val output = ObjectOutputStream(FileOutputStream(file))

    output.writeObject(SomeObject("flame", "flame@google.com"))
}

class SomeObject(
    private val name: String,
    private val email: String
)
```

해당 코드를 테스트하면 아래와 같은 에러가 발생합니다.

```
at java.base/java.io.ObjectOutputStream.writeObject0(ObjectOutputStream.java:1187)
at java.base/java.io.ObjectOutputStream.writeObject(ObjectOutputStream.java:350)
```

에러가 발생한 곳을 확인하면 아래와 같은 코드를 확인할 수 있습니다.

```java
// remaining cases
if (obj instanceof String) {
    writeString((String) obj, unshared);
} else if (cl.isArray()) {
    writeArray(obj, desc, unshared);
} else if (obj instanceof Enum) {
    writeEnum((Enum<?>) obj, desc, unshared);
} else if (obj instanceof Serializable) {
    writeOrdinaryObject(obj, desc, unshared);
} else {
    if (extendedDebugInfo) {
        throw new NotSerializableException(
            cl.getName() + "\n" + debugInfoStack.toString());
    } else {
        throw new NotSerializableException(cl.getName());
    }
}
```

```java
} else if (obj instanceof Serializable) {
```

그 중에 해당 객체가 Serializable 을 확인하는 조건문을 확인할 수 있습니다.

그렇다면 아래와 같이 Serializable 을 구현하면 직렬화가 제대로 동작된다고 예상할 수 있습니다.

```kotlin
class SomeObject(
    private val name: String,
    private val email: String
): Serializable
```

## Conclusion

이렇게 인터페이스 내부에 상수도, 메소드도 없는 인터페이스를 마커 인터페이스 (Marker Interface)라고 합니다.

마커 인터페이스는 대표적인 예시로 Cloneable, Serializable 이 있습니다.
