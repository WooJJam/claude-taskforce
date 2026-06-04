# 회원가입 시스템 설계

**날짜:** 2026-06-04
**범위:** 이메일+비밀번호 기반 회원가입 API (보안 제외, 최소 구현)

---

## 아키텍처

3계층 구조: Controller → Service → Repository → H2 파일 DB

```
POST /api/users/signup
        ↓
UserController
        ↓
UserService  ← 이메일 중복 체크
        ↓
UserRepository (JpaRepository)
        ↓
H2 파일 DB
```

---

## 파일 구조

```
src/main/java/com/woojjam/claude/user/
├── User.java              # @Entity
├── UserRepository.java    # JpaRepository<User, Long>
├── UserService.java       # signup() 비즈니스 로직
└── UserController.java    # REST 엔드포인트
```

---

## 데이터 모델

**User 엔티티**

| 필드 | 타입 | 제약 |
|------|------|------|
| id | Long | PK, auto-increment |
| email | String | unique, not null |
| password | String | not null (평문 저장) |
| createdAt | LocalDateTime | not null |

---

## API 명세

### POST /api/users/signup

**Request Body**
```json
{
  "email": "user@example.com",
  "password": "1234"
}
```

**Response**

| 상황 | 상태코드 | 응답 |
|------|---------|------|
| 성공 | 201 Created | `{ "id": 1, "email": "user@example.com" }` |
| 이메일 중복 | 409 Conflict | `{ "message": "이미 사용 중인 이메일입니다." }` |
| 빈값 | 400 Bad Request | validation error |

---

## 설정

**H2 파일 DB** (`application.yml`)
- datasource url: `jdbc:h2:file:./data/claude-dev`
- H2 콘솔: `/h2-console` (데이터 직접 확인용)
- DDL auto: `create` (최초 실행 시 테이블 자동 생성)

**의존성 추가**
- `spring-boot-starter-data-jpa`
- `h2`
- `spring-boot-starter-validation`

---

## 유효성 검사

- `email`: `@NotBlank`, `@Email`
- `password`: `@NotBlank`

---

## 결정 사항

- 비밀번호 평문 저장 (보안 제외 요건)
- Spring Security 미사용
- 로그인 기능 미포함 (회원가입만)
