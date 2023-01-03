# Inheritance and Composition (Is-a vs Has-a relationship) in Java

## 상속이란

상속이란, 기존의 클래스를 재사용하여 새로운 클래스를 작성하는 것이다.
재사용과 확장을 위해 상속을 사용한다.

## 상속의 단점

'캡슐화'를 깨트리는 치명적인 단점이 있다. 상위클래스의 변수와 메소드가 하위클래스에게 노출 되기 때문이다. 이에 따라 하위클래스는 상위클래스에게 강하게 결합되고, 의존하게 되며 상위클래스의 내부구현이 달라지면, 수정사항이 없는 하위클래스가 오동작 할 수 있다.

## Composition 이란

기존의 클래스를 확장(상속)하는 대신, 새로운 클래스를 만들고 private필드로 기존 클래스의 인스턴스를 참조하게 하는 방법을 통해 기능 확장을 할 수있으며, 이를 Composition(조합||구성)이라 한다.

컴포지션을 활용하면, 상속에서의 문제점을 보완할 수 있다. 캡슐화를 깨지게 하는 문제와, 상위클래스와 하위클래스간에 생기는 강한 결합도와 의존도역시 낮출수 있다.

## 합성의 예시

```java
interface Action {
    void action();
}

class DefaultAction implements Action {
    @Override
    public void action() {
        System.out.println("action");
    }
}

class AClass {
    private Action action;

    public AClass(Action action) {
        this.action = action;
    }

    public void action() {
        this.action.action();
    }
}

class BClass {
    private Action action;

    public BClass(Action action) {
        this.action = action;
    }

    public void action() {
        this.action.action();
    }
}
```
