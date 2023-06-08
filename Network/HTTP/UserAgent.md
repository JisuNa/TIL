# USER-AGENT

## Overview

User-Agent 요청 헤더는 서버와 네트워크 피어가 응용 프로그램, 운영 체제, 공급업체 및/또는 요청하는 사용자 에이전트의 버전을 식별할 수 있게 해주는 특성 문자열입니다.

## 사용법

```kotlin
@GetMapping("/user-agent")
fun userAgent(request: HttpServletRequest) {
    println(request.getHeader("user-agent"))
}

// Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_7) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/113.0.0.0 Safari/537.36
```

이 코드로 user-agent를 확인할 수 있습니다.

## Conclusion

user-agent 헤더로 요청자의 식별 정보를 확인할 수 있습니다.