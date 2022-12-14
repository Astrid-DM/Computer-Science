## 1. Cookie
### 1-1. 정의
- <key, value> 형태의 문자열로 브라우저에 저장되어 사용자를 인식하거나 일부 데이터를 저장
- 서버가 클라이언트에 정보를 전달할 때 저장하고자 하는 정보를 응답 헤더(Cookie)에 저장하여 전달

### 1-2. 등장 배경
- 서버에 무언가를 요청할 때 마다 사용자가 ID, PW를 통해 로그인을 해야하는 불편함이 존재했음
- Cookie는 사용자가 한 번 로그인을 하면 쿠키를 생성하여 저장하고, 이후 요청은 로그인 없이 진행 가능

### 1-3. 문제점
- Cookie가 노출되었을 때, ID, PW와 같은 중요 정보들이 쉽게 노출됨
- 웹 브라우저마다 Cookie에 대한 지원 형태가 다르기 때문에 브라우저간 공유가 불가능
- Cookie의 사이즈는 4KB로 제한되어있어 많은 양의 데이터를 담지 못함

## 2. Cookie + Session
### 2-1. 등장 배경
- Cookie의 문제점을 해결하기 위해 나온 개념
    - Cookie에 ID, PW와 같은 중요 정보를 담는게 아니라, 중요 정보가 아닌 인증을 위한 별개의 정보를 세션 저장소에 저장하고, 클라이언트는 이 정보를 쿠키에 대신 담아서 요청하고 서버는 세션 저장소에 있는 정보랑 일치하는지 확인하는 방식

### 2-2. 동작 과정
1. 클라이언트가 ID/PW로 서버에 로그인을 요청
2. ID/PW로 인증 후, 사용자를 식별할 유니크한 세션 ID를 만들어 마치 자물쇠처럼 서버의 세션 저장소에 저장
3. 세션 ID를 특정한 형태(Cookie or Json)로 클라이언트에 다시 반환
4. 이후 사용자 인증이 필요한 정보를 요청할 때마다 이 세션 ID를 쿠키에 담아 서버에 함께 전달
5. 인증이 필요한 API일때, 서버는 세션 ID가 세션 저장소에 있는지 확인
6. 있다면 인증 완료 후 API처리, 없다면 401 에러 반환

### 2-3. 문제점
- 세션 ID, Cookie 등이 탈취된다면 세션 저장소를 전부 지워 해결 가능하지만, 탈취당하지 않은 정상적인 유저도 모두 재인증을 해야하는 상황이 발생
- HTTP의 특성 중 하나인 `stateless`를 위반
    - `stateless` : 서버는 클라이언트의 상태를 저장하지 않음
- 세션 저장소에 클라이언트의 상태를 저장하는 특성상, `stateful`하게 가게됨
- `stateful`하게 갈 경우, 서버를 스케일 아웃할 때, 1번 서버에 로그인한 사용자가 2번 서버에 요청을 보낸다면 2번 서버에는 로그인 상태가 남아있지 않기 때문에 다시 로그인 해야하는 상황 발생

💡 `stateful` vs `stateless` : [링크 참고](https://github.com/Astrid-DM/Computer-Science/blob/main/WEB/Stateful_VS_Stateless.md)

### 2-4. 문제점 해결
##### 세션 클러스터링 
- 세션 클러스터링으로 서버간 로그인 정보가 담긴 세션을 공유하는 방법이 있지만, 실제 서비스와 관련없는 인프라적인 작업으로 서버 리소스를 많이 쓰게되는 단점 존재
- 전체적인 서버 규모가 크지 않다면 나쁘지 않은 방법이지만, MSA로 잘게 쪼개져 수십, 수백개의 서버로 이루어진다면 단점이 극명하게 드러남

##### 스티키 세션
- 스케일 아웃 시 여러 서버에 세션 정보를 복사할 필요가 없도록 특정 세션을 처음 처리한 서버에게 같은 세션의 요청을 처리하도록 하는 방식
    - A 유저가 최초로 A 서버에 요청을 했다면 이후 A의 모든 요청은 A서버가 처리하는 방식
- 로드 밸런싱이 균일하게 보장되지 않음 (특정 서버에 요청이 몰릴수도 있음)
- 클라이언트의 상태를 어디선가 들고있어야 한다는 문제점을 해결하지 못함

#
> 이러한 문제점을 해결하기 위해 나온 개념이 JWT
#

## 3. JWT
### 3-1. JWT 정의
- JSON Web Token(JWT)은 웹 표준(`RFC 7519`)으로서 두 개체에서 JSON 객체를 사용하고 가볍가 자가 수용적인(Self-contained) 방식으로 정보를 안정성 있게 전달
    - 자가 수용적 : JWT 안에 인증에 필요한 모든 정보를 자체적으로 지니고 있음

### 3-2. 인증과정
![JWT Process](https://velog.velcdn.com/images/znftm97/post/f3062577-90ac-4c7c-9f8e-d0e0babcdcde/image.png)

### 3-3. JWT 구조
![JWT Structure](https://velog.velcdn.com/images/znftm97/post/aff45d8d-8f18-436f-9dc5-2c31cf41e9d4/image.png)
![JWT Example](https://velog.velcdn.com/images/znftm97/post/f2f7bd79-8475-4cb9-81cc-242e1f975421/image.png)
- 각각 구분을 위해 `.`구분자가 들어감
#
#

##### 헤더 (Header)
- 두 가지 정보를 가짐
    - typ - 토큰의 타임 (JWT)
    - alg - 해싱 알고리즘
        - Signature를 해싱하기 위한 안고리즘 지정

##### 내용 (Payload)
- 토큰에 담을 정보들이 존재하고 여기에 담는 정보의 한 조각을 `Claim`이라고 함
- 클레임은 키값 형태로 존재
    - 클레임의 종류는 등록된(registered) 클레임, 공개(public) 클레임, 비공개 클레임 등이 있음
- Payload ex )
``` json
    {
        "iss": "jh.com", // 등록된 클레임
        "exp": "1485270000000", // 등록된 클레임
        "<https://xxx.com/jwt_claims/is_admin>": true, // 공개 클레임
        "userId": "11028373727102", // 비공개 클레임
        "username": "jh" // 비공개 클레임
    }
```

### 3-4. JWT 등장배경
- 인증에 필요한 정보가 토큰에 들이었어서 별도의 저장소가 필요 없음
    - 하지만 보안성을 높이기 위해 `Refresh Token`을 사용하는 경우, 별도의 저장소에 저장하면서 사용하는 경우에 해당하지 않음
- Cookie와 Session 사용시 문제점이었던 Stateful 특성을 JWT 사용시 Stateless하게 가져갈 수 있음
- 다양한 언어에서 지원
- HTTP 헤더에 넣어서 쉽게 전달 가능
- MSA 환경에서 유용

### 3-5. JWT 단점
- 거의 모든 요청에 토큰이 포함되므로 트래픽 크기에 영향을 미칠 수 있음
- 토큰에 정보가 많아져 토큰의 크기가 커지면 네트워크에 부하를 줄 수 있음
- 페이로드는 암호화 된게 아니라 `BASE64`로 인코딩 된 것이므로 중간에 토큰을 탈취하면 페이로드의 데이터를 모두 볼 수 있음
    - 따라서 페이로드에 중요한 정보는 담으면 안됨

## 4. Refresh Token
### 4-1. 등장배경
- 보안에 100%는 없음. 따라서 토큰이 노출되어 탈취당할 경우를 대비해야 하는데, 이런 경우를 대비해 사용하는게 `Refresh Token`
- `Access Token`은 언제든지 탈취될 수 있다고 가정하기 때문에 `Access Token`에는 중요 정보를 담으면 안됨
- 따라서 `Access Token`은 유효기간을 짧게 설정하고, `Refresh Token`의 유효기간은 길게 설정.
- 물론 `Access Token`의 유효기간 동안에는 공격에 노출될 수 있지만, 피해를 최소화 하기 위해 유효기간을 짧게 설정함

### 4-2. 공격자와 일반 유저를 구분하는 방법
- 공격자는 탈취한 `Refresh Token`으로 계속 `Access Token`을 생성해서 정상적인 사용자처럼 서버에 계속 요청을 보낼 수 있음
- 이를 대비해서 서버에서 추가 검증 로직으로 방어해야 함
    - DB에 사용자와 `Access Token`, `Refresh Token`들을 매핑하여 저장
    - 정상적인 유저의 `Access Token`이 만료된 경우
        - `Access Token`과 `Refresh Token`을 서버로 보내서 새 `Access Token`을 요청 -> 서버에서는 DB에 저장된 `Access Token`, `Refresh Token` 쌍과 클라이언트에서 보낸 쌍을 비교 -> 일치하면 새 `Access Token`을 발급
    - 공격자가 `Refresh Token`을 탈취한 경우
        - 공격자가 탈취한 `Refresh Token`으로 새 `Access Token` 생성 요청 -> `Access Token` 없이 요청하면 공격으로 간주 -> 서버에서 `Access Token`, `Refresh Token` 폐기

## 5. JWT vs Session, Cookie
### 5-1. 누가 더 안전한가?
- 명학하게는 결론을 내릴 수 없음. `Session`, `Cookie`, `JWT`든 뭐든 모두 HTTP 프로토콜 위에 동작하는 text로, 네트워크 레이어에 공격자가 접근하여 탈취할 수만 있다면 모두 노출될 수 밖에 없음

### 6-2. 차이점?
- `stateful`, `stateless`의 차이점 존재. 이는 토큰이 탈취됐을 때 서버에서 능동적으로 이에 대응하여 토큰을 폐기처리할 수 있냐 없냐에 직결됨
- JWT의 등장 배경을 살펴보면 보안이 뛰어나서가 아니라 `MSA(MicroService Architecture)`가 도입되면서 주목받기 시작
![amazon vs netflix](https://velog.velcdn.com/images/znftm97/post/51ab5358-60fa-4ed6-8b5d-7654b7ab1904/image.png)
- 위 사진처럼 수천, 수만가지의 서버 to 서버 통신이 이루어지는 아키텍쳐에서 중앙화된 사용자 식별 저장소를 통해 각 API 요청을 인증처리 해야한다면, 인증 서버만 수백대가 필요할 것
- 그렇다고 아무리 내부서버끼리의 통신이라고 인증을 제외할 순 없으니, JWT를 통해 인증을 진행
- 추가로 앱과 웹을 모두 서비스하는 서버의 경우, 웹에서 Session을 이용하고 앱에서 Token을 이용하는 방식으로 별개의 인증 방식을 가져가는게 아니라, 두 환경 모두 토큰을 기반으로 인증하여 환경에 구애받지 않고 동일한 API를 이용할 수 있음


## 7. 요약
### 쿠키
- 적은 용량과 보안 문제점 존재

### 쿠키 + 세션
- 쿠키의 문제점 해결
- 문제점
    - 상태를 저장해야 하므로, 즉, 세션 정보를 저장하기 위한 추가 저장소가 필요
    - `Http Stateless` 특성을 위반
        - 즉, 로그인 할때마다 세션 스토리지(추가 저장소)에 접근해서 확인해야 함
    - 스케일 아웃시 여러 서버에 세션 정보를 복사해줘야하는 작업이 필요
    - 이를 해결하기 위한 개념인 스티키 세션, 세션 클러스터링이 등장했지만 여전히 `stateful`하다는 문제점은 해결 못함

### JWT
- 클라이언트가 상태를 들고있을 필요 없이 토큰만으로 인증처리 가능. 즉, `Stateless`를 보장
- MSA에서 중앙화된 인증방식에 비해 유리
- 그러나 보안 문제로 `Refresh Token`을 도입하면 결국 이를 저장하기 위한 별도의 저장소가 필요한 것은 마찬가지
    - `Stateless`를 완벽하게 보장하지 못함
- 하지만 세션은 로그인을 할때마다 저장소에 접근하고, JWT는 토큰이 만료되었을때만 저장소에 접근하므로 접근하는 횟수 자체는 상대적으로 적다는 장점이 있음
- `Access Token`을 사용하는 기간 동안은 `Stateless`하지만, 만료되었을때는 `Stateless`가 깨지게 됨
- MSA 환경에서 유용

  
  
**결국 JWT도 완벽한 기술은 아니나, 차선책은 될 수 있다.**  
#

> 출처 : [LTH님의 Velog - JWT란?](https://velog.io/@znftm97/JWT-Session-Cookie-비교-sphsi9yh) 
