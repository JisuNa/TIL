# Reference

JDK 1.2 부터는 java.lang.ref 패키지를 추가해 제한적으로 사용자 코드와 GC가 상호작용할 수 있게 했습니다.

java.lang.ref 패키지는 전형적인 객체 참조인 strong reference 외에도 soft, weak, phantom 3가지의 새로운 참조 방식을 Reference 클래스로 제공합니다.

## GC와 Reachability

GC는 객체가 가비지인지 판별하기 위해 reachability 라는 개념을 사용했습니다.

객체에 유효한 참조가 있으면 'reachable' 없으면 'unreachable' 로 구분하여 GC를 수행합니다.

유효한 참조 여부는 최초의 참조가 있어야하는데 root set 이라고 합니다.

