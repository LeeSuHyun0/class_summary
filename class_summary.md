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


'''java
import com.example.noticeboard.admin.login.dto.AdminLoginRequest;
import com.example.noticeboard.admin.login.service.AdminLoginService;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.security.auth.login.AccountException;

@RestController
@RequiredArgsConstructor
@RequestMapping("/admin")
public class AdminLoginController {

    private final AdminLoginService adminLoginService;

    @PostMapping("/login")
    public ResponseEntity adminLogin(@RequestBody AdminLoginRequest request, HttpServletResponse response) throws AccountException {

        return ResponseEntity.ok().body(adminLoginService.adminLogin(request, response));
    }

}
```

`AdminLoginController` 클래스는 관리자 로그인 요청을 처리하는 Spring REST 컨트롤러입니다.

---

## 클래스 및 메서드 기능

### 1. 클래스 역할

* `/admin` 경로 하위 요청을 처리하는 REST API 컨트롤러
* `AdminLoginService`를 통해 실제 로그인 로직을 수행
* 관리자 로그인 전용 엔드포인트를 제공합니다.

---

### 2. `adminLogin` 메서드

* HTTP POST 요청을 `/admin/login` 경로에서 받음
* 요청 본문에 담긴 `AdminLoginRequest` (관리자 로그인 정보 DTO)를 파라미터로 받음
* `HttpServletResponse` 객체도 받아서, 로그인 처리 시 필요한 응답 헤더나 쿠키 설정에 사용 가능
* `adminLoginService.adminLogin()` 메서드를 호출해 로그인 처리 후 반환값을 HTTP 200(OK) 응답 본문으로 전달
* 로그인 실패 시 `AccountException` 예외가 발생할 수 있음

---

## 요약

| 기능          | 설명                                  |
| ----------- | ----------------------------------- |
| 관리자 로그인 API | `/admin/login` POST 요청 처리           |
| 로그인 요청 처리   | 전달받은 로그인 정보로 `adminLoginService` 호출 |
| 로그인 결과 반환   | 로그인 성공 시 결과를 200 OK 응답으로 반환         |
| 예외 처리       | 로그인 실패 시 `AccountException` 던짐 가능   |


'''java
import com.example.noticeboard.oauth.service.CustomOAuth2UserService;
import com.example.noticeboard.oauth.support.CustomAuthenticationFailureHandler;
import com.example.noticeboard.oauth.support.OAuth2AuthenticationSuccessHandler;
import com.example.noticeboard.security.jwt.support.JwtAuthenticationFilter;
import com.example.noticeboard.account.user.constant.UserRole;
import com.example.noticeboard.admin.visitant.util.SingleVisitInterceptor;
import com.example.noticeboard.common.exception.FilterExceptionHandler;
import lombok.RequiredArgsConstructor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.http.HttpMethod;
import org.springframework.security.config.annotation.web.HttpSecurityBuilder;
import org.springframework.security.config.annotation.web.builders.HttpSecurity;
import org.springframework.security.config.annotation.web.configuration.EnableWebSecurity;
import org.springframework.security.config.annotation.web.configurers.AbstractAuthenticationFilterConfigurer;
import org.springframework.security.config.annotation.web.configurers.oauth2.client.OAuth2LoginConfigurer;
import org.springframework.security.config.http.SessionCreationPolicy;
import org.springframework.security.crypto.bcrypt.BCryptPasswordEncoder;
import org.springframework.security.oauth2.client.web.OAuth2LoginAuthenticationFilter;
import org.springframework.security.web.SecurityFilterChain;
import org.springframework.security.web.authentication.UsernamePasswordAuthenticationFilter;

@Configuration
@EnableWebSecurity
@RequiredArgsConstructor
public class SecurityConfig {

    private final JwtAuthenticationFilter authenticationFilter;
    private final CustomOAuth2UserService oauth2UserService;
    private final SingleVisitInterceptor singleVisitInterceptor;
    private final OAuth2AuthenticationSuccessHandler oauth2AuthenticationSuccessHandler;
    private final CustomAuthenticationFailureHandler customAuthenticationFailureHandler;

    @Bean
    public BCryptPasswordEncoder encodePassword() {
        return new BCryptPasswordEncoder();
    }

    @Bean
    public SecurityFilterChain filterChain(HttpSecurity http) throws Exception {
        http.csrf().disable()
                .httpBasic().disable()
                .sessionManagement().sessionCreationPolicy(SessionCreationPolicy.STATELESS)
                .and()
                .authorizeHttpRequests()
                .requestMatchers(HttpMethod.GET, "/admin/**").hasRole(UserRole.MANAGER.name())
                .requestMatchers(HttpMethod.GET, "/**").permitAll()
                .requestMatchers(HttpMethod.POST, "/admin/report/**" , "/admin/login", "/mail/**", "/admin/login", "/logins", "/registers", "/oauth/token", "/user/logout").permitAll()
                .requestMatchers( "/admin/**").hasRole(UserRole.MANAGER.name())
                .requestMatchers(HttpMethod.POST, "/**").hasAnyRole(UserRole.USER.name(), UserRole.MANAGER.name())
                .requestMatchers(HttpMethod.PATCH, "/posts/views/**").permitAll()
                .requestMatchers(HttpMethod.DELETE, "/**").hasAnyRole(UserRole.USER.name(), UserRole.MANAGER.name())
                .requestMatchers(HttpMethod.PATCH, "/**").permitAll()
                .requestMatchers(HttpMethod.PUT, "/**").hasAnyRole(UserRole.USER.name(), UserRole.MANAGER.name())
                .and()
                // Configures authentication support using an OAuth 2.0 and/or OpenID Connect 1.0 Provider. 
                .oauth2Login().loginPage("/authorization/denied")
                // loginPage가 리턴하는 OAuth2LoginConfigurer는 다음과 같음.
// public final class OAuth2LoginConfigurer<B extends HttpSecurityBuilder<B>>
// extends AbstractAuthenticationFilterConfigurer<B, org.springframework.security.config.annotation.web.configurers.oauth2.client.OAuth2LoginConfigurer<B>, OAuth2LoginAuthenticationFilter>
                // OAuth2LoginAuthenticationFilter
                .successHandler(oauth2AuthenticationSuccessHandler)
                .failureHandler(customAuthenticationFailureHandler)
                .userInfoEndpoint().userService(oauth2UserService);

        http.addFilterBefore(new FilterExceptionHandler(),
                UsernamePasswordAuthenticationFilter.class);

        http.addFilterBefore(singleVisitInterceptor,
                UsernamePasswordAuthenticationFilter.class
        );

        http.addFilterBefore(authenticationFilter,
                UsernamePasswordAuthenticationFilter.class);

        return http.build();
    }
```
---

### ✅ SecurityConfig 클래스란?

`SecurityConfig` 클래스는 Spring Security를 사용하여 **애플리케이션의 보안 설정을 담당**하는 핵심 구성 클래스입니다.
로그인, 인증, 인가, 필터 추가 등 다양한 보안 관련 기능을 이곳에서 설정하게 됩니다.

---

### 🔒 주요 기능 요약

1. **JWT 인증 방식 사용**

   * 세션을 사용하지 않고, JWT 토큰을 통해 사용자 인증을 처리합니다.
   * `STATELESS` 설정으로 서버는 로그인 상태를 저장하지 않습니다.

2. **소셜 로그인(OAuth2) 지원**

   * Google, Kakao 등 OAuth2 제공자를 통한 로그인 기능을 제공합니다.
   * 로그인 성공/실패에 따른 핸들러를 설정하여 추가적인 처리도 가능합니다.

3. **URL별 접근 권한 제어**

   * 어떤 경로는 로그인 없이 접근 가능하도록 허용하고,
   * 어떤 경로는 특정 역할(예: USER, MANAGER)만 접근할 수 있도록 제한합니다.

4. **보안 기능 설정**

   * CSRF 및 HTTP Basic 인증을 비활성화합니다 (REST API 환경에 맞게).
   * 세션을 사용하지 않고, 토큰 기반 인증을 사용하도록 설정합니다.

5. **커스텀 필터 추가**

   * JWT 인증 필터
   * 예외 처리 필터
   * 방문자 체크 필터 등 필요한 기능을 직접 필터로 추가할 수 있습니다.

---

### 🧩 설정 예시 설명

* `@Bean BCryptPasswordEncoder`
  → 비밀번호를 안전하게 암호화하여 저장하는 데 사용됩니다.

* `http.csrf().disable()`
  → CSRF 보호 기능을 끕니다. (세션을 사용하지 않는 REST API에서는 일반적입니다)

* `sessionCreationPolicy(SessionCreationPolicy.STATELESS)`
  → 서버가 세션을 생성하거나 저장하지 않도록 설정합니다.

* `authorizeHttpRequests()`
  → 경로별로 접근 권한을 설정합니다.

#### 예시 권한 설정

```plaintext
GET /admin/**          → MANAGER 역할만 접근 가능
GET /**                → 누구나 접근 가능
POST /admin/login 등   → 일부 경로는 로그인 없이 접근 가능
POST /**               → USER 또는 MANAGER 역할만 접근 가능
PATCH /posts/views/**  → 누구나 접근 가능
PATCH /**              → 모두 허용 (주의 필요)
DELETE /**             → USER 또는 MANAGER만 가능
```

* `oauth2Login()`
  → 소셜 로그인 기능 설정

  * 로그인 성공 시 실행할 핸들러
  * 실패 시 실행할 핸들러
  * 사용자 정보 조회 서비스 등을 설정할 수 있습니다.

---

### 🧷 필터 등록 (`addFilterBefore`)

Spring Security의 기본 필터 앞에 커스텀 필터들을 등록합니다. 실행 순서가 중요합니다.

```java
http.addFilterBefore(new FilterExceptionHandler(), UsernamePasswordAuthenticationFilter.class);
http.addFilterBefore(singleVisitInterceptor, UsernamePasswordAuthenticationFilter.class);
http.addFilterBefore(authenticationFilter, UsernamePasswordAuthenticationFilter.class);
```

* `FilterExceptionHandler`: 필터 단계에서 발생한 예외를 처리합니다.
* `singleVisitInterceptor`: 방문자 수 체크 또는 중복 방문 방지 등의 역할로 추정됩니다.
* `authenticationFilter`: JWT 토큰을 검사하여 인증을 수행합니다.

---

### 💡 주입되는 주요 의존성들

* `JwtAuthenticationFilter`: JWT 토큰을 검증하고 인증 처리
* `CustomOAuth2UserService`: 소셜 로그인 시 사용자 정보를 불러옴
* `OAuth2AuthenticationSuccessHandler`: 소셜 로그인 성공 시 실행되는 로직 처리
* `CustomAuthenticationFailureHandler`: 소셜 로그인 실패 시 처리
* `SingleVisitInterceptor`: 특정 요청의 방문 제어 로직 담당

---

### ✨ 정리

`SecurityConfig`는 다음과 같은 기능을 담당합니다:

* JWT 기반 로그인 처리
* 소셜 로그인(OAuth2) 연동
* URL 경로별로 접근 권한 세밀하게 제어
* 보안 필터를 추가하여 인증, 예외 처리 등 다양한 보안 로직 적용

이러한 설정을 통해 REST API 환경에서 보다 안전하고 효율적인 인증/인가 처리를 구성할 수 있습니다.


'''java
import com.example.noticeboard.common.response.ResponseCode;
import com.example.noticeboard.common.response.ResponseMessage;
import lombok.RequiredArgsConstructor;
import org.springframework.http.HttpStatus;
import org.springframework.http.ResponseEntity;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
@RequiredArgsConstructor
public class SecurityController {

    @GetMapping(value = "/authorization/denied")
    public ResponseEntity<ResponseMessage> informAuthorizationDenied() {
        ResponseMessage message = ResponseMessage.of(ResponseCode.AUTHORIZATION_ERROR , "로그인이 필요한 서비스입니다.");

        return ResponseEntity.status(HttpStatus.FORBIDDEN).body(message);
    }

}
```

`SecurityController` 클래스는 **Spring Security에서 인가(Authorization) 실패 시 사용자에게 알림을 제공하는 컨트롤러**입니다.

---

### ✅ 클래스 설명

```java
@RestController
@RequiredArgsConstructor
public class SecurityController { ... }
```

* `@RestController`: REST API 응답을 제공하는 컨트롤러입니다. (`@Controller + @ResponseBody`)
* `@RequiredArgsConstructor`: final 또는 @NonNull 필드에 대해 자동으로 생성자를 생성합니다.
  이 클래스에서는 필드가 없기 때문에 사실상 의미는 없습니다.

---

### ✅ 메서드: `informAuthorizationDenied()`

```java
@GetMapping(value = "/authorization/denied")
public ResponseEntity<ResponseMessage> informAuthorizationDenied() {
    ResponseMessage message = ResponseMessage.of(ResponseCode.AUTHORIZATION_ERROR , "로그인이 필요한 서비스입니다.");
    return ResponseEntity.status(HttpStatus.FORBIDDEN).body(message);
}
```

#### 기능 설명:

* **URL**: `/authorization/denied`로 GET 요청이 들어왔을 때 실행됩니다.
* **의도된 사용처**: 사용자가 로그인이 필요한 페이지에 접근했지만 **로그인하지 않았거나 권한이 없는 경우**, 이 URL로 리다이렉트되도록 설정하는 경우입니다.
* **ResponseMessage**: 커스텀 응답 객체를 사용하여 일관된 형식으로 응답을 줍니다.

  * 예: `ResponseCode.AUTHORIZATION_ERROR` (아마도 코드값이 `403` 또는 관련 메시지일 것으로 추정됩니다)
  * 메시지: `"로그인이 필요한 서비스입니다."`
* **HTTP 상태코드**: `403 Forbidden`
  → 인증은 되었지만, 해당 리소스를 사용할 권한이 없음을 의미합니다.

---

### 💬 쉽게 정리하면

이 컨트롤러는 로그인하지 않았거나 권한이 없을 때
👉 사용자에게 `"로그인이 필요한 서비스입니다."`라는 메시지를 `403` 상태코드와 함께 보내주는 역할을 합니다.
주로 **Spring Security의 OAuth2 실패 처리 또는 접근 거부 리다이렉트** 경로로 사용됩니다.


'''java
public class TokenForgeryException extends RuntimeException {
    private static final long serialVersionUID = 1L;

    public TokenForgeryException(String message) {
        super(message);
    }

}
```
`TokenForgeryException` 클래스는 **JWT 토큰 위조(변조)** 상황에서 사용되는 **사용자 정의 예외 클래스**입니다. 

---

### ✅ 클래스 설명

```java
public class TokenForgeryException extends RuntimeException {
    private static final long serialVersionUID = 1L;

    public TokenForgeryException(String message) {
        super(message);
    }
}
```

#### 🔹 기본 구조

* `RuntimeException`을 상속 → **Unchecked 예외**

  * 예외 처리를 반드시 try-catch로 하지 않아도 됩니다.
  * 보통 개발자가 의도한 예외 상황에 사용됩니다.

* `serialVersionUID`: 자바에서 직렬화(Serializable)를 사용하는 클래스가 변경되었을 때 버전 관리를 도와주는 값입니다. 필수는 아니지만, 명시하는 것이 좋습니다.

* 생성자:

  ```java
  public TokenForgeryException(String message) {
      super(message);
  }
  ```

  * 예외 발생 시 전달된 메시지를 부모 클래스에 넘겨서 예외 메시지로 사용할 수 있게 합니다.

---

### 💡 사용 목적

이 예외는 주로 **JWT 토큰을 검증**할 때,

* 유효하지 않은 서명
* 위조된 내용
* 토큰 값이 조작됨 등

의심스러운 JWT가 감지되었을 때 다음과 같이 사용될 수 있습니다:

```java
if (!isValid(token)) {
    throw new TokenForgeryException("토큰이 위조되었거나 유효하지 않습니다.");
}
```

---

### ✨ 요약

| 항목     | 설명                                  |
| ------ | ----------------------------------- |
| 클래스 이름 | `TokenForgeryException`             |
| 상속     | `RuntimeException` (Unchecked 예외)   |
| 용도     | JWT 등 **토큰이 위조되었을 때 예외를 던지기 위해 사용** |
| 예외 메시지 | 생성자에서 직접 설정 가능 (`super(message)`)   |


```java
import com.example.noticeboard.security.jwt.dto.Token;
import jakarta.persistence.*;
import lombok.AllArgsConstructor;
import lombok.Builder;
import lombok.Getter;
import lombok.NoArgsConstructor;

@Builder
@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class RefreshToken {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(nullable = false)
    private Long tokenId;

    @Column(nullable = false)
    private String token;

    @Column(nullable = false)
    private String keyEmail;

    public static RefreshToken defaultRefreshToken() {
        return RefreshToken.builder()
                .tokenId(1L)
                .token(" ")
                .keyEmail(" ")
                .build();
    }

    public static RefreshToken createRefreshToken(Token token) {
        return RefreshToken.builder()
                .keyEmail(token.getKey())
                .token(token.getRefreshToken())
                .build();
    }

}
```
`RefreshToken` 클래스는 **사용자의 리프레시 토큰을 데이터베이스에 저장하고 관리하기 위한 JPA 엔티티 클래스**입니다. 

---

### ✅ 클래스 설명

```java
@Builder
@Entity
@Getter
@NoArgsConstructor
@AllArgsConstructor
public class RefreshToken { ... }
```

* `@Entity`: 이 클래스는 JPA가 관리하는 테이블로 매핑됩니다. → 실제 DB에 `refresh_token` 같은 테이블로 저장됩니다.
* `@Builder`: 빌더 패턴을 사용하여 객체를 쉽게 생성할 수 있게 합니다.
* `@Getter`: 모든 필드에 대한 getter 메서드를 자동 생성합니다.
* `@NoArgsConstructor` / `@AllArgsConstructor`: 기본 생성자와 모든 필드를 인자로 받는 생성자를 생성합니다.

---

### ✅ 필드 설명

```java
@Id
@GeneratedValue(strategy = GenerationType.IDENTITY)
@Column(nullable = false)
private Long tokenId;
```

* 기본키(`PK`) 역할을 하는 ID입니다.
* `IDENTITY` 전략은 DB에서 자동 증가되는 값(AUTO\_INCREMENT)을 의미합니다.

```java
@Column(nullable = false)
private String token;
```

* 실제 저장되는 **리프레시 토큰 값**입니다.

```java
@Column(nullable = false)
private String keyEmail;
```

* 토큰과 연결된 사용자 식별 키(예: 이메일)를 저장합니다.

---

### ✅ 메서드 설명

#### 1. `defaultRefreshToken()`

```java
public static RefreshToken defaultRefreshToken() {
    return RefreshToken.builder()
            .tokenId(1L)
            .token(" ")
            .keyEmail(" ")
            .build();
}
```

* 기본값으로 채워진 "빈 리프레시 토큰 객체"를 생성합니다.
* 예외 상황 등에서 **기본 토큰 객체**가 필요할 때 사용합니다.

#### 2. `creareRefreshToken(Token token)`

```java
public static RefreshToken creareRefreshToken(Token token) {
    return RefreshToken.builder()
            .keyEmail(token.getKey())
            .token(token.getRefreshToken())
            .build();
}
```

* 전달받은 `Token` 객체로부터 필요한 정보를 꺼내서 `RefreshToken` 객체를 생성합니다.
* `Token` 클래스에는 아마도 `getKey()`와 `getRefreshToken()` 메서드가 정의되어 있을 것입니다.

---

### ✨ 요약

| 항목    | 설명                                                          |
| ----- | ----------------------------------------------------------- |
| 목적    | 리프레시 토큰을 DB에 저장하기 위한 JPA 엔티티                                |
| 주요 필드 | `tokenId`, `token`, `keyEmail`                              |
| 사용 예  | 로그인 시 발급한 리프레시 토큰을 DB에 저장하거나 갱신할 때 사용                       |
| 특징    | `@Builder` 패턴으로 객체 생성 편리, `default`와 `create` 메서드로 유틸 기능 제공 |


```java
import lombok.*;

@Builder
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Token {
    private String grantType;
    private String accessToken;
    private String refreshToken;
    private String key;
}
```
`Token` 클래스는 \*\*JWT 관련 정보(Access Token, Refresh Token 등)를 담는 DTO(Data Transfer Object)\*\*입니다. 이 DTO는 주로 **로그인 성공 시 클라이언트에게 토큰을 전달하거나**,
**DB에 RefreshToken을 저장할 때 정보를 옮기기 위해 사용됩니다.**

---

### ✅ 클래스 개요

```java
@Builder
@Getter
@Setter
@NoArgsConstructor
@AllArgsConstructor
public class Token { ... }
```

#### 📌 Lombok 애노테이션 설명

* `@Builder`: 빌더 패턴을 사용할 수 있게 해 줍니다 (예: `Token.builder().accessToken("...").build()`).
* `@Getter`, `@Setter`: 모든 필드에 대해 Getter/Setter 자동 생성.
* `@NoArgsConstructor`: 기본 생성자 생성.
* `@AllArgsConstructor`: 모든 필드를 받는 생성자 생성.

---

### ✅ 필드 설명

```java
private String grantType;
```

* 일반적으로 "Bearer" 문자열이 들어갑니다.
* 클라이언트가 인증 헤더에 `Authorization: Bearer [AccessToken]` 형식으로 전송할 때 사용됩니다.

```java
private String accessToken;
```

* 실제 로그인 후 발급되는 JWT **액세스 토큰**입니다.
* 클라이언트가 서버에 API 요청할 때 인증 수단으로 사용합니다.

```java
private String refreshToken;
```

* 액세스 토큰이 만료되었을 때, **새로운 토큰을 발급받기 위해 사용되는 토큰**입니다.
* 보통 DB나 Redis 등에 저장해두고, 일정 기간 내 재발급 요청이 오면 새로운 액세스 토큰을 만들어 줍니다.

```java
private String key;
```

* 이 토큰이 어떤 사용자에게 속하는지 식별하는 값입니다.
* 보통 **이메일 또는 사용자 ID** 역할을 합니다.
  → 이 값은 `RefreshToken` 엔티티의 `keyEmail` 필드로 매핑되어 저장됩니다.

---

### ✨ 정리

| 필드명            | 설명                      |
| -------------- | ----------------------- |
| `grantType`    | 토큰의 타입 ("Bearer" 등)     |
| `accessToken`  | 사용자 인증에 사용되는 JWT 액세스 토큰 |
| `refreshToken` | 재발급 요청 시 사용하는 리프레시 토큰   |
| `key`          | 사용자 식별용 키 (이메일, ID 등)   |

---

### 💡 사용 예시

```java
Token token = Token.builder()
    .grantType("Bearer")
    .accessToken("eyJhbGciOiJIUzI1...")
    .refreshToken("dskjlfsdfsdj...")
    .key("user@email.com")
    .build();
```

→ 이처럼 로그인 성공 후 `Token` 객체를 만들어서 클라이언트에 전달하거나,
→ `RefreshToken.createRefreshToken(token)` 으로 DB에 저장하는 데 활용합니다.


'''java
public class InvalidTokenException extends RuntimeException {
    private static final long serialVersionUID = 1L;

    public InvalidTokenException(String message) {
        super(message);
    }
}
'''

`InvalidTokenException` 클래스는 **잘못된(유효하지 않은) 토큰이 감지되었을 때 발생시키는 사용자 정의 예외 클래스**입니다.
---

### ✅ 클래스 설명

```java
public class InvalidTokenException extends RuntimeException {
    private static final long serialVersionUID = 1L;

    public InvalidTokenException(String message) {
        super(message);
    }
}
```

* `RuntimeException`을 상속한 **unchecked 예외**입니다.
  → 예외 처리(try-catch)가 강제되지 않으며, 토큰 검증 과정에서 문제가 발견되면 던집니다.

* `serialVersionUID`: 자바 직렬화 시 클래스 버전 관리용 필드로, 명시적으로 선언하여 안정성을 높입니다.

* 생성자에서 예외 메시지를 받아 부모 클래스인 `RuntimeException`에 전달합니다.

---

### 💡 사용 목적

이 예외는 JWT 또는 기타 토큰 검증 과정에서 다음과 같은 상황에 사용할 수 있습니다:

* 토큰이 아예 없거나 (null 또는 빈 문자열)
* 토큰 형식이 올바르지 않음
* 토큰이 만료되었거나 변조된 경우
* 기타 토큰 검증에서 실패한 경우

예를 들어,

```java
if (!isValid(token)) {
    throw new InvalidTokenException("유효하지 않은 토큰입니다.");
}
```

이렇게 토큰 검증 실패를 명확하게 알리고, 이후 예외 처리 핸들러에서 적절히 응답할 수 있도록 합니다.

---

### ✨ 요약

| 항목     | 내용                                |
| ------ | --------------------------------- |
| 클래스 이름 | `InvalidTokenException`           |
| 상속     | `RuntimeException` (unchecked 예외) |
| 용도     | 유효하지 않은 토큰 발견 시 던지는 예외            |
| 메시지 전달 | 생성자에서 예외 메시지 설정 가능                |


```java
import java.util.Optional;

import org.springframework.data.jpa.repository.JpaRepository;
import org.springframework.data.jpa.repository.Query;
import org.springframework.data.repository.query.Param;

import com.example.noticeboard.security.jwt.domain.RefreshToken;

public interface RefreshTokenRepository extends JpaRepository<RefreshToken, Long> {
	   Optional<RefreshToken> findByToken(String token);

	   @Query(value = "SELECT p from RefreshToken p where p.keyEmail = :userEmail")
	   Optional<RefreshToken> existsByKeyEmail(@Param("userEmail") String userEmail);

	   void deleteByKeyEmail(String userEmail);
}
```

`RefreshTokenRepository` 인터페이스는 **JPA를 이용해 `RefreshToken` 엔티티에 접근하는 데이터베이스 레포지토리** 역할을 합니다.

---

### ✅ 인터페이스 개요

```java
public interface RefreshTokenRepository extends JpaRepository<RefreshToken, Long> { ... }
```

* `JpaRepository<RefreshToken, Long>`를 상속하여 기본 CRUD 기능과 페이징, 정렬 기능을 사용할 수 있습니다.
* `RefreshToken` 엔티티를 `tokenId(Long)`를 기본 키로 관리합니다.

---

### ✅ 주요 메서드 기능

#### 1. `Optional<RefreshToken> findByToken(String token)`

* **기능**: DB에서 리프레시 토큰 값(`token` 필드)으로 해당 토큰 정보를 찾습니다.
* **반환**: 토큰이 존재하면 `Optional` 안에 `RefreshToken` 객체를, 없으면 빈 `Optional` 반환.

---

#### 2. `Optional<RefreshToken> existsByKeyEmail(@Param("userEmail") String userEmail)`

* **기능**: `keyEmail` 컬럼에 특정 이메일(`userEmail`)이 존재하는지 확인하는 커스텀 쿼리입니다.
* **JPQL 사용**: `"SELECT p from RefreshToken p where p.keyEmail = :userEmail"`

  * `p`는 `RefreshToken` 엔티티의 별칭입니다.
* **반환**: 해당 이메일에 연결된 리프레시 토큰이 있으면 `Optional`에 담아 반환, 없으면 빈 값 반환.

---

#### 3. `void deleteByKeyEmail(String userEmail)`

* **기능**: 특정 이메일(`keyEmail`)에 해당하는 리프레시 토큰을 삭제합니다.
* 이메일에 해당하는 토큰을 DB에서 제거할 때 사용합니다.

---

### ✨ 요약

| 메서드 이름                     | 기능 설명                               |
| -------------------------- | ----------------------------------- |
| `findByToken(String)`      | 토큰 값을 이용해 `RefreshToken` 조회         |
| `existsByKeyEmail(String)` | 이메일로 리프레시 토큰 존재 여부 확인 (Optional 반환) |
| `deleteByKeyEmail(String)` | 이메일로 리프레시 토큰 삭제                     |

---

### 💡 실제 사용 예시

```java
// 토큰 조회
Optional<RefreshToken> refreshToken = refreshTokenRepository.findByToken(tokenString);

// 이메일로 토큰 존재 여부 확인
Optional<RefreshToken> tokenByEmail = refreshTokenRepository.existsByKeyEmail(userEmail);

// 이메일로 토큰 삭제
refreshTokenRepository.deleteByKeyEmail(userEmail);


'''java
import com.example.noticeboard.common.response.ResponseCode;
import com.example.noticeboard.common.response.ResponseMessage;
import com.example.noticeboard.security.exception.TokenForgeryException;
import com.example.noticeboard.security.jwt.repository.RefreshTokenRepository;
import com.example.noticeboard.security.jwt.support.CookieSupport;
import com.example.noticeboard.security.jwt.support.JwtTokenProvider;
import com.example.noticeboard.security.jwt.domain.RefreshToken;
import com.example.noticeboard.security.jwt.dto.Token;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServletRequest;
import jakarta.servlet.http.HttpServletResponse;
import lombok.RequiredArgsConstructor;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

import java.util.Arrays;
import java.util.NoSuchElementException;

import static com.example.noticeboard.security.jwt.domain.RefreshToken.creareRefreshToken;

@Service
@RequiredArgsConstructor
public class JwtService {
    private final JwtTokenProvider jwtTokenProvider;
    private final RefreshTokenRepository refreshTokenRepository;

    @Transactional
    public void login(Token token) {
        RefreshToken refreshToken = creareRefreshToken(token);
        String loginUserEmail = refreshToken.getKeyEmail();

        refreshTokenRepository.existsByKeyEmail(loginUserEmail).ifPresent(a -> {
            refreshTokenRepository.deleteByKeyEmail(loginUserEmail);
        });

        refreshTokenRepository.save(refreshToken);
    }

    public RefreshToken getRefreshToken(HttpServletRequest request) {
        String refreshToken = getRefreshTokenFromHeader(request);

        return refreshTokenRepository.findByToken(refreshToken)
                .orElseThrow(() -> new TokenForgeryException("알 수 없는 RefreshToken 입니다."));
    }

    public ResponseMessage validateRefreshToken(HttpServletRequest request , HttpServletResponse response) {
        try {
            RefreshToken token = getRefreshToken(request);
            String accessToken = jwtTokenProvider.validateRefreshToken(token);

            response.addHeader("Set-Cookie" , CookieSupport.createAccessToken(accessToken).toString());

            return ResponseMessage.of(ResponseCode.CREATE_ACCESS_TOKEN);
        } catch (NoSuchElementException e) {
            CookieSupport.deleteJwtTokenInCookie(response);

            throw new TokenForgeryException("변조된 RefreshToken 입니다.");
        }
    }

    public String getRefreshTokenFromHeader(HttpServletRequest request) {
        Cookie cookies[] = request.getCookies();

        if (cookies != null && cookies.length != 0) {
            return Arrays.stream(cookies)
                    .filter(c -> c.getName().equals("refreshToken")).findFirst().map(Cookie::getValue)
                    .orElseThrow(() -> new SecurityException("인증되지 않은 사용자입니다."));
        }

        throw new SecurityException("인증되지 않은 사용자입니다.");
    }

}
```
`JwtService` 클래스는 JWT 인증에서 **리프레시 토큰 관리 및 검증**을 담당하는 서비스 클래스입니다.

---

# JwtService 클래스 기능 요약

* 리프레시 토큰 저장, 삭제, 검증을 수행합니다.
* 클라이언트의 요청에서 리프레시 토큰을 쿠키에서 읽어와 검증합니다.
* 리프레시 토큰 유효성 검사 후 새 액세스 토큰을 발급하고, 쿠키에 담아 응답합니다.
* 위조된 토큰에 대해선 예외를 발생시키고, 쿠키를 삭제합니다.

---

## 필드

```java
private final JwtTokenProvider jwtTokenProvider;
private final RefreshTokenRepository refreshTokenRepository;
```

* `JwtTokenProvider`: JWT 토큰 생성 및 검증을 담당하는 컴포넌트.
* `RefreshTokenRepository`: 리프레시 토큰 저장소(DB) 접근용 인터페이스.

---

## 주요 메서드

### 1. `login(Token token)`

* 로그인 성공 시 호출.
* `Token` DTO로부터 리프레시 토큰 정보를 만들어서(`creareRefreshToken`)
* 이미 DB에 존재하는 같은 이메일(`keyEmail`)의 리프레시 토큰이 있다면 삭제 후
* 새 리프레시 토큰을 저장합니다.

즉, 사용자별로 하나의 최신 리프레시 토큰만 유지하는 역할을 합니다.

---

### 2. `getRefreshToken(HttpServletRequest request)`

* HTTP 요청의 쿠키에서 리프레시 토큰을 추출(`getRefreshTokenFromHeader` 메서드 활용)
* 추출한 토큰을 DB에서 찾아서 반환
* DB에 없으면 `TokenForgeryException` (위조 토큰 예외)을 발생시킵니다.

---

### 3. `validateRefreshToken(HttpServletRequest request, HttpServletResponse response)`

* 클라이언트 요청의 리프레시 토큰을 검증합니다.
* 정상 토큰이면 `jwtTokenProvider`를 통해 새 액세스 토큰을 생성하고
* 새 액세스 토큰을 쿠키에 넣어 응답 헤더에 추가합니다.
* 처리 성공 시 `ResponseMessage`에 성공 코드(`CREATE_ACCESS_TOKEN`)를 담아 반환합니다.
* 예외 발생 시 (토큰 위조, DB 미존재 등)

  * 클라이언트 쿠키에서 JWT 토큰을 삭제하고
  * `TokenForgeryException` 예외를 던집니다.

---

### 4. `getRefreshTokenFromHeader(HttpServletRequest request)`

* HTTP 요청 쿠키에서 이름이 `"refreshToken"`인 쿠키 값을 찾아 반환합니다.
* 없으면 `SecurityException("인증되지 않은 사용자입니다.")`를 던집니다.

---

## 추가 설명

* `creareRefreshToken(Token token)` 메서드는 `RefreshToken` 엔티티를 생성하는 static 메서드로, 토큰 DTO에서 정보를 꺼내서 만듭니다.
* 쿠키 관리(`CookieSupport.createAccessToken()`, `CookieSupport.deleteJwtTokenInCookie()`)는 별도의 유틸 클래스로 처리하고 있습니다.
* `@Transactional`이 붙은 `login` 메서드는 DB에 저장과 삭제 작업을 트랜잭션 단위로 처리하여 안전성을 보장합니다.

---

## 전체적인 역할

* 로그인 시 리프레시 토큰을 DB에 저장/갱신
* 클라이언트 요청 시 쿠키에서 리프레시 토큰을 읽어와 검증
* 검증 완료 시 새 액세스 토큰을 생성해 쿠키에 넣어 응답
* 위조나 만료 등 문제 발생 시 예외 처리 및 쿠키 삭제


'''java
import com.example.noticeboard.security.jwt.dto.Token;
import jakarta.servlet.http.Cookie;
import jakarta.servlet.http.HttpServletResponse;
import lombok.NoArgsConstructor;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.ResponseCookie;

@NoArgsConstructor(access = lombok.AccessLevel.PRIVATE)
public class CookieSupport {

    @Value("${server.url}")
    private static String DOMAIN_URL;

    public static ResponseCookie createAccessToken(String access) {
        return ResponseCookie.from("accessToken" , access)
                .path("/")
                .maxAge(30 * 60 * 1000)
                .secure(true)
                .domain(DOMAIN_URL)
                .httpOnly(true)
                .sameSite("none")
                .build();
    }

    public static ResponseCookie createRefreshToken(String refresh) {
        return ResponseCookie.from("refreshToken" , refresh)
                .path("/")
                .maxAge(14 * 24 * 60 * 60 * 1000)
                .secure(true)
                .domain(DOMAIN_URL)
                .httpOnly(true)
                .sameSite("none")
                .build();
    }

    public static void setCookieFromJwt(Token token , HttpServletResponse response) {
        response.addHeader("Set-Cookie" , createAccessToken(token.getAccessToken()).toString());
        response.addHeader("Set-Cookie" , createRefreshToken(token.getRefreshToken()).toString());
    }

    public static void deleteJwtTokenInCookie(HttpServletResponse response) {
        Cookie accessToken = new Cookie("accessToken", null);
        accessToken.setPath("/");
        accessToken.setMaxAge(0);
        accessToken.setDomain(DOMAIN_URL);

        Cookie refreshToken = new Cookie("refreshToken", null);
        refreshToken.setPath("/");
        refreshToken.setMaxAge(0);
        refreshToken.setDomain(DOMAIN_URL);

        response.addCookie(accessToken);
        response.addCookie(refreshToken);
    }

}
```
`CookieSupport` 클래스는 JWT 토큰 관련 쿠키를 생성, 설정, 삭제하는 **쿠키 유틸리티 클래스**입니다.

---

## 클래스 개요

* 쿠키 관련 작업을 모아둔 헬퍼 클래스입니다.
* `@NoArgsConstructor(access = lombok.AccessLevel.PRIVATE)` 로 생성자를 private 처리하여 인스턴스화 방지(유틸 클래스 목적).
* `DOMAIN_URL`은 쿠키 도메인 설정에 사용되는데, `@Value`로 외부에서 주입하려 했지만 `static` 필드에는 적용되지 않아 실제 값 주입 안 될 가능성이 있습니다.(주석 참고)

---

## 주요 메서드

### 1. `createAccessToken(String access)`

```java
public static ResponseCookie createAccessToken(String access)
```

* 이름: `"accessToken"`
* 값: 액세스 토큰 문자열
* 경로: `/` (도메인 전체에서 접근 가능)
* 만료 시간: 30분 (`30 * 60 * 1000` 밀리초, 다만 `maxAge()`는 초 단위라 오작동 가능성 있음)
* 보안 설정: `secure=true` (HTTPS에서만 전송)
* 도메인: `DOMAIN_URL`
* HttpOnly: true (JS에서 쿠키 접근 불가)
* SameSite: `"none"` (크로스 사이트 요청 시에도 쿠키 전송 허용)
* 반환: Spring의 `ResponseCookie` 객체

---

### 2. `createRefreshToken(String refresh)`

* 이름: `"refreshToken"`
* 값: 리프레시 토큰 문자열
* 경로: `/`
* 만료 시간: 14일 (`14 * 24 * 60 * 60 * 1000` 밀리초, 역시 `maxAge()` 초 단위 주의)
* 기타 설정은 `createAccessToken`과 동일

---

### 3. `setCookieFromJwt(Token token, HttpServletResponse response)`

* 액세스 토큰과 리프레시 토큰을 각각 쿠키로 만들어
* 응답 헤더 `"Set-Cookie"`에 추가합니다.
* 즉, 로그인 등 토큰 발급 시 클라이언트에게 두 개의 쿠키를 전달하는 역할입니다.

---

### 4. `deleteJwtTokenInCookie(HttpServletResponse response)`

* `"accessToken"`과 `"refreshToken"` 쿠키를 삭제합니다.
* 쿠키의 값은 `null`로, `maxAge`를 `0`으로 설정하여 즉시 만료시킵니다.
* 도메인과 경로도 지정하여 삭제 대상 쿠키가 정확하게 매칭되도록 합니다.
* 주로 로그아웃 시 쿠키를 클라이언트에서 제거할 때 사용합니다.

---

## 주의 사항 및 개선점

* `@Value("${server.url}")` 는 `static` 필드에는 자동 주입이 안 됩니다.
  → 현재 `DOMAIN_URL` 값이 주입되지 않을 수 있으니, 다른 방법 (예: `@Component`로 바꾸고 인스턴스 필드로 사용하거나, 생성자 주입)으로 관리하는 것이 좋습니다.

* `maxAge` 메서드 인자는 초 단위인데, 현재는 밀리초 단위 값이 들어가 있어 쿠키 만료 시간이 비정상적일 수 있습니다.
  → `maxAge(30 * 60)` (30분 = 1800초) 또는 `maxAge(14 * 24 * 60 * 60)` (14일) 로 수정하는 것이 맞습니다.

---

## 요약

| 기능            | 설명                                     |
| ------------- | -------------------------------------- |
| 액세스 토큰 쿠키 생성  | `accessToken` 이름의 쿠키를 만들어 보안 설정과 함께 반환 |
| 리프레시 토큰 쿠키 생성 | `refreshToken` 이름의 쿠키를 만들어 반환          |
| 쿠키 응답 헤더 설정   | 로그인 시 두 토큰 쿠키를 HTTP 응답에 포함             |
| 쿠키 삭제         | 로그아웃 시 쿠키 삭제용으로 만료시킨 쿠키를 응답에 추가        |
