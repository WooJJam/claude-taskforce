# Implementer 참조 — Spring Boot (Java)

Implementer 스킬의 일반 TDD 원칙을 Spring Boot(Java + JUnit5 + Mockito) 스택에 적용할 때의 구체 컨벤션.

---

## AAA 패턴 예시

```java
@Test
@DisplayName("중복 이메일 회원가입시 예외 발생")
void 중복_이메일로_회원가입하면_예외가_발생한다() {
    // Arrange
    given(userRepository.existsByEmail("dup@example.com")).willReturn(true);

    // Act & Assert
    assertThatThrownBy(() -> userService.signup(new SignupRequest("dup@example.com", "pw")))
            .isInstanceOf(IllegalArgumentException.class);
}
```

## 테스트 네이밍

- 메서드명은 한글로 행동을 서술한다.
- `@DisplayName`을 반드시 붙인다.

```java
@DisplayName("이메일 회원가입")
void 새_이메일로_회원가입하면_저장된다();

@DisplayName("중복 이메일 회원가입시 예외 발생")
void 중복_이메일로_회원가입하면_예외가_발생한다();
```

---

## 단위 테스트 — 서비스/도메인 로직

Mockito로 의존성을 격리한다.

```java
@ExtendWith(MockitoExtension.class)
class UserServiceTest {
    @Mock UserRepository userRepository;
    @InjectMocks UserService userService;
}
```

## 통합 테스트 — API 엔드포인트

실제 컨텍스트를 띄워 HTTP 동작을 검증한다.

```java
@SpringBootTest
@AutoConfigureMockMvc
@Transactional   // 테스트 종료 후 롤백
class UserControllerTest {
    @Autowired MockMvc mockMvc;
}
```

---

## 최종 검증 명령

```bash
./gradlew test    # 전체 테스트
./gradlew build   # 빌드(컴파일 + 테스트 + 패키징)
```

> Maven 프로젝트면 `./mvnw test`, `./mvnw verify`.
