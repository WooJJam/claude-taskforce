# QA 참조 — Spring Boot (Java)

QA 시나리오 검증을 Spring Boot 스택에서 수행할 때의 구체 방법.

---

## API 시나리오 검증 — MockMvc

통합 테스트 컨텍스트에서 엔드포인트를 호출해 응답을 단언한다.

```java
mockMvc.perform(get("/api/users/" + id))
        .andExpect(status().isOk())
        .andExpect(jsonPath("$.code").value("SUCCESS"))
        .andExpect(jsonPath("$.data.email").value("user@example.com"));
```

예외/경계 시나리오:

```java
mockMvc.perform(get("/api/users/999999"))
        .andExpect(status().isNotFound())
        .andExpect(jsonPath("$.code").value("USER-001"));
```

## 검증 실행 명령

```bash
./gradlew test    # 시나리오에 대응하는 테스트 실행
```

## 참고: 실제 E2E가 필요한 경우

MockMvc는 실제 HTTP 소켓을 타지 않는 통합 테스트다. 진짜 E2E가 필요하면:

- `@SpringBootTest(webEnvironment = RANDOM_PORT)` + `TestRestTemplate`/`WebTestClient`로 실제 포트에 요청
- 롤백 없이 실제 DB 상태 검증 (Testcontainers 등)
