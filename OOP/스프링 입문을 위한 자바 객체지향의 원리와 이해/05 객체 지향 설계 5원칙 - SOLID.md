# 05 객체 지향 설계 5원칙 - SOLID

SOLID는 결합도는 낮추고 응집도는 높이기 위해 객체 지향의 관점에서 재정립한 것이다.

**왜 결합도는 낮추고 응집도는 높여야 하는가?**

- 리팩터링과 유지보수가 쉬워진다
- 확장성이 좋아진다.
- 재사용성이 상승한다.
- 자연적인 모델링이 가능해진다.
- 클래스 단위로 모듈화 해서 대형 프로젝트 개발이 용이해진다.

## SRP 단일 책임 원칙

Single Responsibility Principle

각 객체나 클래스는 오직 하나의 책임만을 지게 해야 한다.

모듈이 변경되어야할 이유는 한가지여야한다.

같은 이유로 변하는 것들을 모아라. 다른 이유로 변경되는 항목을 분리해라.

**SRP 위배**
```java
public class UserInfoEmailSender {
    private String userName;
    private String email;

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public void setEmail(String email) {
        this.email = email;
    }

    public void sendEmail() {
        // 이메일 전송 코드
    }
}
```

**SRP로 개선**
```java
public class UserInfo {
    private String userName;
    private String email;

    public void setUserName(String userName) {
        this.userName = userName;
    }

    public void setEmail(String email) {
        this.email = email;
    }
}

public class EmailSender {
    public void sendEmail(String email) {
        // 이메일 전송 코드
    }
}
```

## OCP 개방 폐쇄 원칙

Open Closed Principle

자신의 확장에는 열려 있고, 주변의 변화에 대해서는 닫혀 있어야 한다.

새로운 기능을 추가할 수 있도록 설계되어야 하지만, 기존 기능을 수정하지 않도록 설계되어야 한다.

인터페이스를 구현하거나, 추상 클래스를 상속받아 기능을 확장하는 방식을 사용하는 것이 좋다.

**OCP 위배**
```java
class PaymentProcessor {
    public void processPayment(Payment payment) {
        if (payment.getType() == PaymentType.CREDIT_CARD) {
            // process credit card payment
        } else if (payment.getType() == PaymentType.PAYPAL) {
            // process PayPal payment
        } else if (payment.getType() == PaymentType.BITCOIN) {
            // process bitcoin payment
        }
    }
}
```

**OCP로 개선**
```java
interface PaymentProcessor {
    void processPayment(Payment payment);
}

class CreditCardProcessor implements PaymentProcessor {
    public void processPayment(Payment payment) {
        // process credit card payment
    }
}

class PayPalProcessor implements PaymentProcessor {
    public void processPayment(Payment payment) {
        // process PayPal payment
    }
}

class BitcoinProcessor implements PaymentProcessor {
    public void processPayment(Payment payment) {
        // process bitcoin payment
    }
}
```
## LSP 리스코프 치환 원칙

서브 타입은 언제나 자신의 기반 타입으로 교체할 수 있어야 한다.
- 하위 클래스 is a kind of 상위 클래스 - 하위 분류는 상위 분류의 한 종류다.
- 구현 클래스 is able to 인터페이스 - 구현 분류는 인터페이스할 수 있어야 한다.

상위 타입의 객체가 하위 타입의 객체로 치환될 수 있어야 한다는 의미이다.

즉, 상위 타입의 객체가 하위 타입의 객체로 치환될 수 있으면 그 객체는 자신의 인터페이스를 충실히 준수한다고 할 수 있다.

## ISP 인터페이스 분리 원칙

범용 인터페이스 보다는 특정 인터페이스로 세분화하는게 좋다.

**ISP 위배**
```java
public interface AllInOneDevice {
    void print();
    void copy();
    void fax();
}

public class SmartMachine implements AllInOneDevice {
    @Override
    public void print() {
        System.out.println("print");
    }

    @Override
    public void copy() {
        System.out.println("copy");
    }

    @Override
    public void fax() {
        System.out.println("fax");
    }
}

public class PrinterMachine implements AllInOneDevice {
    @Override
    public void print() {
        System.out.println("print");
    }

    @Override
    public void copy() {
        throw new UnsupportedOperationException();
    }

    @Override
    public void fax() {
        throw new UnsupportedOperationException();
    }
}
```
위 코드를 보면 `PrinterMachine`은 `AllInOneDevice` 인터페이스의 구현을 강제하기 때문에 사용할 수 없는 `copy`와 `fax`를 못하게 막고 있다.

인터페이스를 더 세분화 한다면 불필요한 코드를 줄일 수 있다.

**ISP로 개선**
```java
public interface PrinterDevice {
    void print();
}

public interface CopyDevice {
    void copy();
}

public interface FaxDevice {
    void fax();
}

public class SmartMachine implements PrinterDevice, CopyDevice, FaxDevice {
    @Override
    public void print() {
        System.out.println("print");
    }

    @Override
    public void copy() {
        System.out.println("copy");
    }

    @Override
    public void fax() {
        System.out.println("fax");
    }
}

public class PrinterMachine implements PrinterDevice {
    @Override
    public void print() {
        System.out.println("print");
    }
}
```

## DIP 의존관계 역전 원칙

상위 레벨은 하위 레벨의 추상화에 의존해야지, 구체화에 의존하면 안된다.
