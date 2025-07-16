```java
import com.example.noticeboard.account.profile.service.ProfileService;
import com.example.noticeboard.common.response.ResponseMessage;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.*;

import com.example.noticeboard.account.profile.dto.ProfileRequest;

@RestController
@RequestMapping("/profiles")
@RequiredArgsConstructor
public class ProfileController {

	private final ProfileService profileService;
	
	@GetMapping(value = "/statistics")
	public ResponseEntity<ResponseMessage> getStatistics(@CookieValue String accessToken) {
		return ResponseEntity.ok().body(profileService.getStatistics(accessToken));
	}
	
	@PutMapping
	public ResponseEntity<ResponseMessage> updateProfile(@RequestBody ProfileRequest profileRequest , @CookieValue String accessToken , HttpServletResponse response) {

		return ResponseEntity.ok().body(profileService.updateProfile(profileRequest , accessToken , response));
	}

	@GetMapping
	public ResponseEntity<ResponseMessage> getProfileData(@CookieValue String accessToken) {
		return ResponseEntity.ok().body(profileService.getProfileFromUser(accessToken));
	}
}
```

## 📌 핵심 기능 요약

`/profiles` 경로로 들어오는 요청을 처리하며, 다음과 같은 기능을 제공합니다:

1. **프로필 통계 조회** (`GET /profiles/statistics`)
2. **프로필 정보 수정** (`PUT /profiles`)
3. **프로필 정보 조회** (`GET /profiles`)

---

## 📂 주요 구성 설명

### ✅ 클래스 어노테이션

* `@RestController`: JSON 형태로 응답을 반환하는 REST 컨트롤러
* `@RequestMapping("/profiles")`: 모든 API 경로는 `/profiles`로 시작
* `@RequiredArgsConstructor`: `final`로 선언된 `profileService`를 자동 생성자 주입

---

## 🧩 각 메서드 설명

### 1. `getStatistics()`

```java
@GetMapping(value = "/statistics")
public ResponseEntity<ResponseMessage> getStatistics(@CookieValue String accessToken)
```

* **역할**: 현재 로그인한 사용자의 **프로필 관련 통계(예: 작성한 글 수, 댓글 수 등)** 를 조회
* **입력**: 쿠키에서 `accessToken`을 꺼내 인증
* **출력**: `ResponseMessage`로 감싼 통계 데이터

---

### 2. `updateProfile()`

```java
@PutMapping
public ResponseEntity<ResponseMessage> updateProfile(@RequestBody ProfileRequest profileRequest,
                                                     @CookieValue String accessToken,
                                                     HttpServletResponse response)
```

* **역할**: 사용자 프로필 정보(닉네임, 소개, 프로필 이미지 등)를 수정
* **입력**:

  * `ProfileRequest` (사용자가 보낸 수정 요청 정보)
  * `accessToken` (인증 토큰, 쿠키에서 꺼냄)
  * `HttpServletResponse` (응답에 쿠키 등을 조작하기 위함)
* **출력**: 업데이트 결과를 담은 `ResponseMessage`

---

### 3. `getProfileData()`

```java
@GetMapping
public ResponseEntity<ResponseMessage> getProfileData(@CookieValue String accessToken)
```

* **역할**: 현재 로그인한 사용자의 프로필 정보를 조회
* **입력**: 쿠키에서 꺼낸 `accessToken`
* **출력**: 사용자의 프로필 데이터를 담은 `ResponseMessage`

---

## 📦 사용하는 객체들

* `ProfileService`: 핵심 비즈니스 로직을 처리하는 서비스 클래스
* `ProfileRequest`: 사용자의 프로필 수정 요청을 담는 DTO
* `ResponseMessage`: 일관된 API 응답 형식을 위한 래퍼 클래스 (예: 상태 코드, 메시지, 데이터 포함)

---

## 🧠 정리하면

이 컨트롤러는 클라이언트(웹/앱 등)로부터 다음 요청을 받아 처리합니다:

| HTTP 메서드 | 경로                     | 설명          |
| -------- | ---------------------- | ----------- |
| GET      | `/profiles/statistics` | 내 프로필 통계 조회 |
| PUT      | `/profiles`            | 내 프로필 정보 수정 |
| GET      | `/profiles`            | 내 프로필 정보 조회 |

모든 요청은 **쿠키에 저장된 accessToken을 기반으로 로그인한 사용자**의 정보만 다룹니다.
로그인하지 않은 사용자는 접근할 수 없도록 설계되어 있을 가능성이 높습니다 (예: 필터나 인터셉터로 막음).


```java
import com.example.noticeboard.account.profile.dto.ProfileRequest;
import jakarta.persistence.*;
import lombok.Getter;
import lombok.Setter;

@Entity
@Getter
@Setter
public class Profile {
	
	@Id
	@GeneratedValue(strategy = GenerationType.IDENTITY)
	private long profileKey;

	private String phone;
	
	@Column(nullable = false)
	private int options;
	
	static public Profile createInitProfileSetting() {
		Profile profile = new Profile();
		profile.setOptions(1);
		return profile;
	}

	public void updateProfile(ProfileRequest profileRequest) {
		this.phone = profileRequest.getPhone();
		this.options = profileRequest.getOptions();
	}
	
}
```

## 📌 주요 역할 요약

| 기능                                             | 설명 |
| ---------------------------------------------- | -- |
| ✅ DB의 `Profile` 테이블과 매핑되는 엔티티                  |    |
| ✅ 초기 기본 프로필 설정 생성 (`createInitProfileSetting`) |    |
| ✅ 사용자 요청으로 프로필 정보 수정 (`updateProfile`)         |    |

---

## 📂 필드 설명

| 필드명          | 타입       | 설명                                   |
| ------------ | -------- | ------------------------------------ |
| `profileKey` | `long`   | PK (기본 키), 자동 생성 (`@GeneratedValue`) |
| `phone`      | `String` | 사용자의 전화번호 (nullable)                 |
| `options`    | `int`    | 사용자 설정 (ex: 알림 설정, 공개 여부 등. 필수값)     |

* `@Entity`: JPA 엔티티임을 명시
* `@Getter`, `@Setter`: Lombok으로 자동으로 Getter/Setter 생성
* `@Column(nullable = false)`: `options` 필드는 DB에 **NULL 불가**

---

## 🧩 주요 메서드 설명

### 1. `createInitProfileSetting()`

```java
static public Profile createInitProfileSetting()
```

* **역할**: 기본값이 설정된 `Profile` 객체를 생성 (주로 회원가입 시 초기값으로 사용)
* **설정값**:

  * `options` 필드를 1로 세팅
  * `phone`은 null (입력되지 않음)

---

### 2. `updateProfile(ProfileRequest profileRequest)`

```java
public void updateProfile(ProfileRequest profileRequest)
```

* **역할**: 사용자가 보낸 `ProfileRequest` DTO 기반으로 현재 프로필 정보를 수정
* **수정 대상**:

  * `phone` (전화번호)
  * `options` (사용자 설정값)

---

## 🧠 정리하면

이 클래스는 **회원 프로필을 나타내는 JPA 엔티티**로서:

* DB에 저장되는 사용자 프로필 데이터를 정의하고,
* 초기 프로필을 생성하거나,
* 사용자의 요청으로 프로필을 업데이트하는 기능을 제공합니다.


```java
import lombok.*;

@Builder
@Getter
@Setter
@AllArgsConstructor
@NoArgsConstructor
public class ProfileRequest {
    private String userId;
    private String phone;
    private Integer options;

} 
```

## 📌 핵심 역할

| 역할                                       | 설명 |
| ---------------------------------------- | -- |
| ✅ 사용자의 프로필 정보(전화번호, 설정 등)를 담는 **전송용 객체** |    |
| ✅ 컨트롤러에서 요청을 받을 때 매핑                     |    |
| ✅ `Profile` 엔티티에 전달하여 업데이트에 활용           |    |

---

## 📂 필드 설명

| 필드명       | 타입        | 설명                           |
| --------- | --------- | ---------------------------- |
| `userId`  | `String`  | 사용자 식별자 (로그인 ID 등)           |
| `phone`   | `String`  | 사용자 전화번호                     |
| `options` | `Integer` | 사용자 설정 값 (예: 알림 설정, 공개 범위 등) |

---

## 🧵 Lombok 어노테이션 설명

| 어노테이션                 | 기능                            |
| --------------------- | ----------------------------- |
| `@Builder`            | 빌더 패턴으로 객체 생성 가능              |
| `@Getter` / `@Setter` | 모든 필드에 대해 getter/setter 자동 생성 |
| `@AllArgsConstructor` | 모든 필드 값을 받는 생성자 생성            |
| `@NoArgsConstructor`  | 파라미터 없는 기본 생성자 생성             |

---

## ✅ 정리

`ProfileRequest` 클래스는:

* **사용자 프로필 수정 요청** 데이터를 담는 DTO(Data Transfer Object)
* `Profile` 엔티티와 1:1로 매핑되는 구조는 아니지만, 수정에 필요한 핵심 필드를 포함
* Lombok을 이용해 코드 간결하게 작성
* 컨트롤러에서 `@RequestBody`로 자동 매핑되어 활용됩니다.


```java
import com.example.noticeboard.account.user.domain.User;
import com.fasterxml.jackson.annotation.JsonFormat;
import com.example.noticeboard.account.profile.domain.Profile;
import lombok.*;

import java.time.LocalDate;

@Builder
@Getter
@NoArgsConstructor(access = AccessLevel.PRIVATE)
@AllArgsConstructor(access = AccessLevel.PRIVATE)
public class ProfileResponse {
    private String email;

    private String phone;

    private int options;

    @JsonFormat(shape = JsonFormat.Shape.STRING , pattern = "yyyy-MM-dd" , timezone = "Asia/Seoul")
    private LocalDate joinDate;

    public static ProfileResponse createProfileResponse(Profile profile , User user) {
        return ProfileResponse.builder()
                .email(user.getEmail())
                .phone(profile.getPhone())
                .options(profile.getOptions())
                .joinDate(user.getJoinDate())
                .build();
    }
}
```

## 📌 핵심 역할

| 역할                                              | 설명 |
| ----------------------------------------------- | -- |
| ✅ 사용자 프로필 조회 시 응답 데이터 구조 정의                     |    |
| ✅ `User`와 `Profile` 엔티티 정보를 하나의 DTO로 묶어 반환      |    |
| ✅ 날짜 포맷 지정 (joinDate 필드에 대해 `yyyy-MM-dd` 형식 지정) |    |

---

## 📂 필드 설명

| 필드명        | 타입          | 설명                              |
| ---------- | ----------- | ------------------------------- |
| `email`    | `String`    | 사용자의 이메일 주소 (User 엔티티에서 가져옴)    |
| `phone`    | `String`    | 전화번호 (Profile 엔티티에서 가져옴)        |
| `options`  | `int`       | 사용자 설정 값 (Profile에서 가져옴)        |
| `joinDate` | `LocalDate` | 가입 날짜 (User에서 가져옴, JSON 포맷 지정됨) |

* `@JsonFormat`: 날짜를 `"yyyy-MM-dd"` 형식으로 JSON 변환 시 포맷 지정
  → `2025-07-16` 형태로 출력됨
  → 타임존은 `Asia/Seoul`

---

## 🛠 생성 메서드: `createProfileResponse`

```java
public static ProfileResponse createProfileResponse(Profile profile, User user)
```

* **역할**: `User`와 `Profile` 객체에서 필요한 정보를 추출해 `ProfileResponse` 객체 생성
* **장점**: 서비스 계층에서 불필요한 로직 없이 깔끔하게 객체를 만들 수 있음

## 🧵 Lombok 어노테이션 정리

| 어노테이션                                   | 설명                   |
| --------------------------------------- | -------------------- |
| `@Builder`                              | 빌더 패턴으로 객체 생성 가능     |
| `@Getter`                               | 모든 필드에 대해 getter 생성  |
| `@NoArgsConstructor(access = PRIVATE)`  | 기본 생성자를 외부에서 못 쓰게 제한 |
| `@AllArgsConstructor(access = PRIVATE)` | 전체 필드 생성자도 외부에서 못 씀  |

---

## ✅ 정리

`ProfileResponse` 클래스는:

* 클라이언트에 반환할 **프로필 데이터 응답용 DTO**
* `User`와 `Profile` 정보를 조합해 필요한 필드만 전달
* 날짜 형식 지정 (`yyyy-MM-dd`)
* 생성자 접근 제한 + 정적 팩토리 메서드 사용 → **안정적이고 일관된 응답 객체 생성**


```java
import lombok.Builder;
import lombok.Getter;

@Builder
@Getter
public class StatisticsResponse {
    private long totalPost;
    private long totalComment;
    private long totalPostView;
    private String joinData;
}
```

## 📌 핵심 역할

| 역할                             | 설명 |
| ------------------------------ | -- |
| ✅ 사용자 활동 통계를 담는 응답 전용 객체 (DTO) |    |
| ✅ API 응답에서 통계 데이터를 JSON 형태로 반환 |    |
| ✅ `@Builder`로 객체 생성 간결화        |    |

---

## 📂 필드 설명

| 필드명             | 타입       | 설명                   |
| --------------- | -------- | -------------------- |
| `totalPost`     | `long`   | 사용자가 작성한 총 게시글 수     |
| `totalComment`  | `long`   | 사용자가 작성한 총 댓글 수      |
| `totalPostView` | `long`   | 사용자가 작성한 게시글의 총 조회 수 |
| `joinData`      | `String` | 사용자 가입일 (날짜 문자열)     |

---

## 🧵 Lombok 어노테이션

| 어노테이션      | 설명                         |
| ---------- | -------------------------- |
| `@Builder` | 빌더 패턴으로 객체 생성 (가독성 향상)     |
| `@Getter`  | 모든 필드에 대해 getter 메서드 자동 생성 |

---

## ✅ 정리

`StatisticsResponse`는:

* **사용자의 게시판 활동 통계**를 담는 **API 응답 DTO**
* `@Builder`로 편하게 생성 가능
* 주로 `GET /profiles/statistics` 같은 요청에서 응답 데이터로 사용됨

> 💡 실제 통계 데이터는 서비스 계층에서 계산한 후, 이 DTO로 감싸서 클라이언트에 전달합니다.


```java
import com.example.noticeboard.account.profile.dto.StatisticsResponse;

public interface CustomProfileRepository {

    StatisticsResponse getStatisticsOfUser(String userId);

}
```

## 📌 핵심 역할

| 역할                                                           | 설명 |
| ------------------------------------------------------------ | -- |
| ✅ 사용자 ID를 기반으로 통계 데이터를 조회                                    |    |
| ✅ 통계 결과를 `StatisticsResponse` DTO 형태로 반환                     |    |
| ✅ Spring Data JPA의 기본 기능 외에 **복잡한 커스텀 쿼리**를 작성하기 위한 확장 인터페이스 |    |

---

## 📂 메서드 설명

```java
StatisticsResponse getStatisticsOfUser(String userId);
```

* **기능**: 전달된 `userId`에 해당하는 사용자가

  * 작성한 게시글 수,
  * 작성한 댓글 수,
  * 게시글 총 조회 수,
  * 가입 날짜
    등을 통계로 계산하여 `StatisticsResponse` 객체로 반환

* **반환 타입**: `StatisticsResponse` (응답 DTO, JSON 변환 가능)

---

## ✅ 정리

`CustomProfileRepository`는:

* 특정 사용자의 **활동 통계를 계산**하는 메서드를 정의한 인터페이스
* JPA의 기본 CRUD 기능으로는 처리하기 어려운 **맞춤형 쿼리 작업**을 처리할 때 사용
* 구현 클래스(`CustomProfileRepositoryImpl`)에서 실제 통계 계산 쿼리를 작성하여 사용


```java
import com.example.noticeboard.account.profile.domain.Profile;
import org.springframework.data.jpa.repository.JpaRepository;

public interface ProfileRepository extends JpaRepository<Profile, Long> , CustomProfileRepository {

}
```

## 📌 핵심 기능 요약

| 기능                                            | 설명 |
| --------------------------------------------- | -- |
| ✅ `Profile` 엔티티에 대한 기본 CRUD 기능 제공             |    |
| ✅ 커스텀 통계 조회 기능(`getStatisticsOfUser`) 포함      |    |
| ✅ Spring이 자동으로 구현체를 만들어줌 (`JpaRepository` 덕분) |    |

---

## 📂 코드 분석

```java
public interface ProfileRepository 
       extends JpaRepository<Profile, Long>, CustomProfileRepository
```

### 🔹 `JpaRepository<Profile, Long>`

* Spring Data JPA가 제공하는 인터페이스
* 기본적인 CRUD 메서드가 자동으로 구현됨

| 메서드                    | 설명       |
| ---------------------- | -------- |
| `findById(Long id)`    | ID로 조회   |
| `findAll()`            | 전체 조회    |
| `save(Profile entity)` | 저장 또는 수정 |
| `deleteById(Long id)`  | 삭제       |
| `count()`              | 개수 세기    |

### 🔹 `CustomProfileRepository`

* 사용자가 직접 만든 **커스텀 쿼리용 인터페이스**
* 예: `getStatisticsOfUser(String userId)` 같은 맞춤형 기능 추가
* 실제 구현체는 `CustomProfileRepositoryImpl` 클래스에서 작성해야 함

---

## ✅ 정리

`ProfileRepository`는:

* `Profile` 엔티티를 위한 표준 JPA 리포지토리이며,
* Spring Data JPA가 제공하는 기본 CRUD 기능을 자동으로 사용하고,
* 직접 정의한 통계 조회 같은 커스텀 기능도 함께 사용할 수 있는 확장형 리포지토리입니다.

> 즉, **기본 + 커스텀 기능을 모두 포함한 Profile 데이터 전용 DAO**입니다.


```java
import com.example.noticeboard.account.user.domain.QUser;
import com.example.noticeboard.post.comment.domain.QComment;
import com.example.noticeboard.post.post.domain.QPost;
import com.querydsl.jpa.impl.JPAQueryFactory;
import com.example.noticeboard.account.profile.dto.StatisticsResponse;
import lombok.RequiredArgsConstructor;

import java.time.LocalDate;

@RequiredArgsConstructor
public class ProfileRepositoryImpl implements CustomProfileRepository {

    private final JPAQueryFactory queryFactory;

    @Override
    public StatisticsResponse getStatisticsOfUser(String userId) {

        long totalPost = queryFactory.select(QPost.post.count())
                .from(QPost.post)
                .innerJoin(QPost.post.writer , QUser.user).on(QUser.user.id.eq(userId))
                .fetchOne();

        Integer totalView = queryFactory.select(QPost.post.views.sum())
                .from(QPost.post)
                .innerJoin(QPost.post.writer , QUser.user).on(QUser.user.id.eq(userId)).fetchOne();

        if(totalView == null) {
            totalView = 0;
        }

        long totalComment = queryFactory.select(QComment.comment.count())
                .from(QComment.comment)
                .innerJoin(QComment.comment.writer , QUser.user).on(QUser.user.id.eq(userId)).fetchOne();

        LocalDate joinDate = queryFactory.select(QUser.user.joinDate)
                .from(QUser.user)
                .where(QUser.user.id.eq(userId)).fetchOne();

        return StatisticsResponse.builder()
                .totalPost(totalPost)
                .totalPostView(totalView)
                .totalComment(totalComment)
                .joinData(joinDate.toString())
                .build();
    }
}
```

## 📌 핵심 기능

| 기능                                  | 설명 |
| ----------------------------------- | -- |
| ✅ 특정 사용자 ID에 대해 직접 쿼리를 날려 **통계 계산** |    |
| ✅ 게시글 수, 댓글 수, 게시글 조회수 총합, 가입일을 계산  |    |
| ✅ `StatisticsResponse` 객체로 감싸서 반환   |    |

---

### 🔍 `getStatisticsOfUser(String userId)` 메서드

#### 1. 📌 **게시글 수 조회**

```java
long totalPost = queryFactory.select(QPost.post.count())
    .from(QPost.post)
    .innerJoin(QPost.post.writer, QUser.user)
    .on(QUser.user.id.eq(userId))
    .fetchOne();
```

* 해당 유저가 작성한 **게시글 수**를 count
* `Post.writer` 필드와 `User.id`를 조인하여 해당 사용자의 글만 집계

---

#### 2. 📌 **게시글 총 조회수 조회**

```java
Integer totalView = queryFactory.select(QPost.post.views.sum())
    .from(QPost.post)
    .innerJoin(QPost.post.writer, QUser.user)
    .on(QUser.user.id.eq(userId))
    .fetchOne();
```

* 게시글의 `views` 필드를 모두 더함 (sum)
* 결과가 null일 경우 `0`으로 처리 (게시글이 없을 수도 있기 때문)

```java
if(totalView == null) {
    totalView = 0;
}
```

---

#### 3. 📌 **댓글 수 조회**

```java
long totalComment = queryFactory.select(QComment.comment.count())
    .from(QComment.comment)
    .innerJoin(QComment.comment.writer, QUser.user)
    .on(QUser.user.id.eq(userId))
    .fetchOne();
```

* 해당 유저가 작성한 **댓글 수**를 count

---

#### 4. 📌 **가입일 조회**

```java
LocalDate joinDate = queryFactory.select(QUser.user.joinDate)
    .from(QUser.user)
    .where(QUser.user.id.eq(userId))
    .fetchOne();
```

* `User` 테이블에서 해당 사용자의 **가입일(joinDate)** 을 조회

---

#### 5. 📌 통계 DTO로 반환

```java
return StatisticsResponse.builder()
    .totalPost(totalPost)
    .totalPostView(totalView)
    .totalComment(totalComment)
    .joinData(joinDate.toString())
    .build();
```

* 모든 통계를 `StatisticsResponse` 객체로 묶어서 반환
* `joinData`는 `String` 형태로 변환 (예: `"2025-07-16"`)

---

## ✅ 정리

`ProfileRepositoryImpl`은:

| 항목            | 설명                                     |
| ------------- | -------------------------------------- |
| 📌 클래스 이름     | `ProfileRepositoryImpl`                |
| 📌 implements | `CustomProfileRepository`              |
| 📌 주요 기능      | `userId`를 기준으로 **직접 쿼리를 날려 통계 정보를 계산** |
| 📌 기술 스택      | QueryDSL 사용 (타입 안전한 쿼리)                |
| 📌 반환 형태      | `StatisticsResponse` DTO               |

### 🔸 결과적으로 이 코드는

➡️ `GET /profiles/statistics` API에서
➡️ 특정 유저의 활동 통계를 응답하기 위한 **백엔드 핵심 구현 로직**입니다.


```java
import com.example.noticeboard.account.user.domain.User;
import com.example.noticeboard.account.user.service.LoginService;
import com.example.noticeboard.common.response.ResponseCode;
import com.example.noticeboard.common.response.ResponseMessage;
import com.example.noticeboard.common.response.message.AccountMessage;
import com.example.noticeboard.security.jwt.support.CookieSupport;
import com.example.noticeboard.security.jwt.support.JwtTokenProvider;
import com.example.noticeboard.account.profile.dto.StatisticsResponse;
import com.example.noticeboard.account.profile.dto.ProfileRequest;
import com.example.noticeboard.account.user.exception.LoginException;
import com.example.noticeboard.account.user.repository.LoginRepository;
import com.example.noticeboard.account.profile.repository.ProfileRepository;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import static com.example.noticeboard.account.profile.dto.ProfileResponse.createProfileResponse;

@Service
@RequiredArgsConstructor
public class ProfileService {

    private final ProfileRepository profileRepository;
	private final LoginRepository loginRepository;
	private final JwtTokenProvider jwtTokenProvider;
	private final LoginService loginService;
	
	public ResponseMessage<StatisticsResponse> getStatistics(String accessToken) {
		String userId = jwtTokenProvider.getUserPk(accessToken);

		return ResponseMessage.of(ResponseCode.REQUEST_SUCCESS, profileRepository.getStatisticsOfUser(userId));
	}

	@Transactional
	public ResponseMessage updateProfile(ProfileRequest profileRequest , String token , HttpServletResponse response) {
		User user = loginService.findUserByAccessToken(token);

		if(!profileRequest.getUserId().equals(user.getId())) {
			if(loginRepository.findById(profileRequest.getUserId()).isPresent()) {
				throw new LoginException(AccountMessage.EXISTS_ID);
			}

			user.updateId(profileRequest.getUserId());

			CookieSupport.deleteJwtTokenInCookie(response);
		}

		user.getProfile().updateProfile(profileRequest);

		return ResponseMessage.of(ResponseCode.REQUEST_SUCCESS);
	}

	public ResponseMessage getProfileFromUser(String token) {
		User result = loginService.findUserByAccessToken(token);

		return ResponseMessage.of(ResponseCode.REQUEST_SUCCESS, createProfileResponse(result.getProfile() , result));
	}
}
```

## 📌 클래스의 핵심 역할

| 기능                                     | 설명 |
| -------------------------------------- | -- |
| ✅ 사용자 활동 통계 조회 (`getStatistics`)       |    |
| ✅ 사용자 프로필 수정 (`updateProfile`)         |    |
| ✅ 사용자 프로필 정보 조회 (`getProfileFromUser`) |    |

---

## 📂 필드 설명

```java
private final ProfileRepository profileRepository;
private final LoginRepository loginRepository;
private final JwtTokenProvider jwtTokenProvider;
private final LoginService loginService;
```

* `ProfileRepository`: 사용자 프로필 저장소 (통계 등 커스텀 쿼리 포함)
* `LoginRepository`: 사용자 정보 확인용 (ID 중복 확인 등)
* `JwtTokenProvider`: JWT 토큰에서 사용자 ID 추출
* `LoginService`: 토큰 기반 사용자 정보 조회

---

## 🧩 메서드별 상세 설명

---

### 1. `getStatistics(String accessToken)`

```java
public ResponseMessage<StatisticsResponse> getStatistics(String accessToken)
```

#### 🧭 기능

* JWT 액세스 토큰에서 사용자 ID를 추출
* 해당 사용자의 통계 정보를 `ProfileRepository`에서 가져옴
* `StatisticsResponse` DTO로 래핑하여 반환

#### 📤 반환

* 게시글 수, 댓글 수, 조회 수, 가입일 등을 포함한 통계 정보

---

### 2. `updateProfile(ProfileRequest profileRequest, String token, HttpServletResponse response)`

```java
@Transactional
public ResponseMessage updateProfile(...)
```

#### 🧭 기능

* JWT 토큰을 통해 현재 로그인한 사용자 조회
* 클라이언트에서 보낸 `ProfileRequest.userId`가 기존 ID와 다르면:

  * ID 중복 체크
  * 중복이 없으면 ID 변경
  * 쿠키에서 기존 JWT 삭제 (`CookieSupport.deleteJwtTokenInCookie`)
* 전화번호 및 옵션 업데이트

#### ✅ 보안 포인트

* **현재 로그인한 사용자 ID와 요청된 ID가 다를 경우** 신중히 처리
* ID 변경 시, 중복 체크 및 재인증을 유도하는 처리 포함

#### 📤 반환

* 성공 여부를 담은 `ResponseMessage`

---

### 3. `getProfileFromUser(String token)`

```java
public ResponseMessage getProfileFromUser(String token)
```

#### 🧭 기능

* JWT 토큰을 통해 현재 로그인한 사용자 정보 조회
* `ProfileResponse`로 변환하여 반환

#### 📤 반환

```json
{
  "email": "user@example.com",
  "phone": "010-1234-5678",
  "options": 1,
  "joinDate": "2025-07-16"
}
```

---

## ✅ 정리

| 메서드                    | 기능                              |
| ---------------------- | ------------------------------- |
| `getStatistics()`      | 사용자의 게시글/댓글/조회수/가입일 통계 조회       |
| `updateProfile()`      | 사용자 프로필 수정 (전화번호, 설정, ID 변경 포함) |
| `getProfileFromUser()` | 현재 로그인한 사용자의 프로필 정보 조회          |

### 🎯 전체적으로

이 `ProfileService`는 로그인된 사용자의 프로필 데이터를 안전하게 **읽고, 수정하고, 통계로 분석하는 핵심 서비스 계층**입니다.

> 💡 보안상 중요한 부분(예: ID 변경, 쿠키 제거, 인증 확인)이 잘 처리되어 있어 실제 서비스에서 유용하게 쓰일 수 있는 구조입니다.
