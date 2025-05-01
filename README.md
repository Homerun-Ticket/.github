![image](https://github.com/user-attachments/assets/b1413c0f-6219-4e3f-88aa-6ab8005b767d)

## 🚀 서비스 소개

### ✨ 서비스 개요

<aside>

요즘 다시 야구 열기가 뜨거워졌습니다.

하지만 아직 까지 프로야구 티켓 예매는 **티켓 링크, 인터파크 등 사이트 별로 분산** 되어 있어

팬 입장에서 **예매 환경이 불편하고 번거롭다는 불만**이 많습니다.

또한, 경기 당일 갑작스러운 일정 변경이나 예기치 못한 사정으로 인해

티켓을 사용하지 못하게 되는 경우, **취소 수수료가 부담**되고,

반대로 인기 경기의 경우 **웃돈을 주고서라도 티켓을 구하고 싶은 팬들**도 많습니다.

> 이러한 니즈를 바탕으로
> 
> 
> "**티켓 예매와 티켓 경매가 함께 가능한 통합 프로야구 티켓 플랫폼**"을 기획했습니다.
>

</aside>

---

## **🧑🏻‍💻** 팀원 소개

|이름|역할|
|------|----------------------------------------------------------------------------------------------|
|이규정| 경매 CRUD 기능 구현 / Redis 기반 캐싱 전략 및 이벤트 연동 설계 / 동시성 제어 |
|석연걸| 멤버, 포인트 CRUD / Spring Security를 통한 인증 인가 구현 / 결제, API Gateway / 서버 모니터링 |
|류성현| 티켓, 예매 CRUD / 대기열 기능을 통한 트래픽 제어 / 동시성 제어 |
|윤현호| 경기, 경기장 기능 구현 / 서버 간 통신 기반 남은 좌석 수 조회 구조 설계 및 최적화 / Redis 기반 캐싱 전략 및 이벤트 연동 설계 |
|이인학| CI/CD 배포 |

---

## 👤 사용자 이용 흐름

- 티켓 예매
  
    ![image](https://github.com/user-attachments/assets/d748a6d9-4e5d-4517-bb8b-5edf9b74e0b3)
    
- 티켓 경매
  
    ![image](https://github.com/user-attachments/assets/987f36c7-d8b4-405b-84e5-6318e8d761e3)

## 🔄 서비스 작동 흐름

- 좌석 선점 및 티켓 예매
    
   ![image](https://github.com/user-attachments/assets/3291a526-5fd8-4653-8428-2b3225a80b52)
    
- 경매
    
    ![image](https://github.com/user-attachments/assets/a322c68f-3eb5-44ea-b0b2-ab605e4f1304)

---

## 🔑 핵심 기능

<aside>

🎫 **티켓팅 기능**

- 대기열
- 남은 좌석 수 조회
- 좌석 점유 및 티켓 예매

</aside>

<br>

<aside>

🔨 **경매**

- 경매 등록 / 입찰 / 상위 입찰 / 낙찰

</aside>

<br>

<aside>

**🔖 공통**

- API Gateway
- 인증/인가 서버
- 포인트 결제
- google reCAPTCHA 및 Rate Limiter

</aside>

---

## 🏗️ System Architecture

![image](https://github.com/user-attachments/assets/0545802f-8b25-4b54-88a6-19bdd080a3b6)

---

## 📘 설계 문서

- **API 명세서**
    
    [ticketable](https://documenter.getpostman.com/view/27492121/2sB2j4eAeL)
    
- **ERD**
    
    ![image](https://github.com/user-attachments/assets/804a6aaa-3b8a-4bab-8531-76938a6ee100)
  
---

## 🛠️ 기술 스택

![image](https://github.com/user-attachments/assets/48466192-35c2-4f9c-8d29-648aed5ce582)

---


## 🏦 기술적 의사결정 및 성능 개선

<aside>

- **모놀리식에서 MSA로: 기술적 분리와 통합의 결정 과정 (내용 수정 예정) ← 다같이 쓰는게 좋을듯**
    
    ---
    
    ### 1. 문제 정의
    
    현재 서비스의 회원가입 및 로그인 절차가 너무 간단하여, 누구나 쉽게 가입하고 로그인할 수 있습니다.
    
    이는 애플리케이션 성능에도 직접적인 영향을 미칠 수 있습니다.
    
    한 가지 예시를 들자면, 서비스에서 제공하는 이메일 알림 기능은 비동기 **`AsyncAwait`** 메커니즘을 활용하여 진행됩니다.
    
    가입된 모든 사용자에게 알림을 보내면 ‘비동기 + 블로킹 i/o (네트워크 i/o)’ 작업 때문에 가용 스레드가 고갈될 수 있습니다.
    
    또한, 서비스에서 사용자의 나이와 경력처럼 민감한 정보를 다루기 때문에, 무분별한 아이디-비밀번호 입력 시도를 방지하는 보안 조치도 필요합니다.
    
    ---
    
    ### 2. 기능 설계, 구현
    
    1. 조건 없는 이메일 알림 발송 → 이메일 알림 인증을 받은 사용자에게만 발송하도록 개선
        - 유효하지 않은 이메일로 발송하는 경우를 차단
    2. 회원가입/로그인이 너무 쉬운 구조 → 회원가입 시 이메일 인증을 받도록 개선
        - 이메일 서비스 남용 방지 위해 한 이메일 당 하루 최대 3번으로 제한
        - 인증코드 실패 횟수도 체크하고, 3회 이상 실패 시 코드 무효화
    3. 해커의 무분별한 아이디/비밀번호 입력 시도 → 아이피 밴, 이메일 밴 도입
        - 아이피 밴
            - 한 IP에서 4개 이상의 계정에 로그인 시도 시 밴, 시도 실패 정보는 Redis에 저장
            - IPv6을 우선적으로 수집
            - 공유 네트워크를 고려하여 30초만 Redis에 밴 정보 저장
            - AOP 코드
                
                ```java
                @Slf4j
                @Aspect
                @Component
                @RequiredArgsConstructor
                public class IpBlockingAspect {
                
                    private final KeyValueCommandRepository keyValueCommandRepository;
                    private final KeyValueQueryRepository keyValueQueryRepository;
                
                    private static final String BLOCK_PREFIX = "block:ip:";
                    private static final String LOGIN_ATTEMPT_PREFIX = "attempt:ip:";
                    private static final long IP_BLOCK_DURATION = 30;  //ip 30초 차단
                    private static final long ATTEMPT_TTL = 15;        //15분 동안 시도 기록 유지
                    private static final int MAX_DIFFERENT_EMAILS = 3; //서로 다른 이메일 4개 이상 감지되면 차단(3개까지 허용)
                
                    /**
                     * 서로 다른 이메일 4개 이상 로그인 시도 시 해당 사용자의 ip 를 차단합니다.
                     * 로그인 성공 시에도 시도 기록이 남아있습니다.(로그인 후 재로그인 악용 방지)
                     */
                    @Around("execution(* com.project.cheerha.domain.auth.controller.AuthController.login(..))")
                    public Object blockAbnormalIp(ProceedingJoinPoint joinPoint) throws Throwable {
                        HttpServletRequest request = getRequest();
                        if (request == null) {
                            return joinPoint.proceed();
                        }
                
                        Object[] args = joinPoint.getArgs();
                        CreateLoginRequestDto dto = (CreateLoginRequestDto) args[0];
                        String email = dto.email();
                
                        String ip = IpUtil.getClientIp(request);
                        String redisBlockKey = BLOCK_PREFIX + ip;
                        String redisAttemptKey = LOGIN_ATTEMPT_PREFIX + ip;
                
                        try {
                            return joinPoint.proceed();
                        } catch (Exception e) {
                            //해당 ip 에서 로그인 시도한 이메일 리스트 가져오기
                            List<String> attemptedEmails = keyValueQueryRepository.getListRange(redisAttemptKey, 0, -1);
                            if (attemptedEmails == null || !attemptedEmails.contains(email)) {
                                keyValueCommandRepository.pushToListLeft(redisAttemptKey, email);
                            }
                
                            //추가 시 ttl 설정
                            keyValueCommandRepository.expireValue(redisAttemptKey, ATTEMPT_TTL, TimeUnit.MINUTES);
                
                            //서로 다른 이메일이 3개 이상이면 차단
                            if (!Objects.requireNonNull(attemptedEmails).contains(email) && attemptedEmails.size() >= MAX_DIFFERENT_EMAILS) {
                                keyValueCommandRepository.setValue(redisBlockKey, "blocked", IP_BLOCK_DURATION, TimeUnit.SECONDS);
                                log.warn("IP {} 차단됨 (서로 다른 {}개 이메일 감지됨)", ip, MAX_DIFFERENT_EMAILS + 1);
                                keyValueCommandRepository.removeValue(redisAttemptKey);
                            }
                            throw e;
                        }
                    }
                
                    private HttpServletRequest getRequest() {
                        ServletRequestAttributes attributes = (ServletRequestAttributes) RequestContextHolder.getRequestAttributes();
                        return attributes != null ? attributes.getRequest() : null;
                    }
                
                }
                
                ```
                
        - 이메일 밴
            - 3일 내에 로그인 5회 이상 실패하면 해당 이메일 밴
            - 실패 정보는 Redis, 밴 정보는 DB에 저장
            - AOP 코드
                
                ```java
                @Slf4j
                @Aspect
                @Component
                @RequiredArgsConstructor
                public class EmailBlockingAspect {
                
                    private final KeyValueCommandRepository keyValueCommandRepository;
                    private final BannedEmailRepository bannedEmailRepository;
                
                    private static final String FAIL_PREFIX = "fail:email:";
                    private static final long EMAIL_FAIL_DURATION = 3;    //로그인 실패 시 실패데이터 3일간 유지
                    private static final int MAX_FAILED_COUNT = 5;  //5회 실패 시 차단
                    private static final String BAN_MASSAGE = "비밀번호 입력 5회 실패";    //db에 저장되고, 로그에 출력되는 메세지
                
                    /**
                     * 같은 이메일로 5회 이상 로그인 실패한 경우 밴하는 메서드입니다.
                     */
                    @Around("execution(* com.project.cheerha.domain.auth.controller.AuthController.login(..))")
                    public Object blockAbnormalEmail(ProceedingJoinPoint joinPoint) throws Throwable {
                        Object[] args = joinPoint.getArgs();
                        CreateLoginRequestDto dto = (CreateLoginRequestDto) args[0];
                        String email = dto.email();
                        String failCountKey = FAIL_PREFIX + email;
                
                        //차단된 이메일인지 확인
                        if (bannedEmailRepository.existsByEmail(email)) {
                            log.warn("임시차단된 사용자의 로그인 요청: {}", email);
                            throw new UnAuthorizedException(AuthErrorCode.BANNED_EMAIL);
                        }
                
                        try {
                            Object result = joinPoint.proceed(args);
                            //로그인 성공 시 failCount 삭제
                            keyValueCommandRepository.removeValue(failCountKey);
                            return result;
                        } catch (Exception e) {
                            if(Objects.equals(e.getMessage(), "패스워드가 잘못되었습니다.")){
                                //잘못된 비밀번호 입력 시 count 1회 추가, 첫 추가 시 ttl 설정
                                long failedAttempts = keyValueCommandRepository.incrementValue(failCountKey);
                                if (failedAttempts == 1) {
                                    keyValueCommandRepository.expireValue(failCountKey, EMAIL_FAIL_DURATION, TimeUnit.DAYS);
                                }
                
                                //잘못된 시도 5회 시 이메일 차단
                                if (failedAttempts >= MAX_FAILED_COUNT) {
                                    String message = BAN_MASSAGE;
                                    BannedEmail bannedEmail = BannedEmail.toEntity(
                                            email,
                                            message
                                    );
                                    log.warn("email {} 차단 완료 : {}", email, message);
                                    bannedEmailRepository.save(bannedEmail);
                                    keyValueCommandRepository.removeValue(failCountKey);
                                }
                            }
                            throw e;
                        }
                    }
                }
                
                ```
                
    4. 이메일 밴을 당한 사용자는 어떻게 재접근할 수 있을까? → 비밀번호 리셋 기능 도입
        - 이메일 인증을 통해 비밀번호 리셋 가능
    5. 밴 당한 IP는 모든 API를 사용할 수 없게 전체 차단
    
    ---
    
    ### 3. 기능 구현 시 트러블슈팅
    
    **[문제 발생]** IP 전역 밴을 **`Interceptor`**로 구현하다가 문제가 발생함
    
    ![image.png](attachment:279264e9-9a7d-4674-946a-6fcd3c05b438:image.png)
    
    ![image.png](attachment:301ca667-903f-4e8d-a499-fff758b81a89:image.png)
    
    - JWT토큰 관련 필터가 걸린 API에서는 토큰 Exception이 먼저 출력됨
        
        -. JWT Filter가 인터셉터보다 앞단에 있음을 의미함
        
    
    ![image.png](attachment:0cf0049e-0a00-4ce5-90b9-05b700a3f64a:image.png)
    
    **[해결 방법]** IP Blocking은 필터체인 사용
    
    - 해당 문제는 **`filter`**는 웹MVC에 존재하고 스프링MVC에 존재하는 **`Interceptor`**보다 전방에 위치했기 때문에 발생함
    
     → IP Blocking을 필터체인에서 가장 우선순위에 두어서 문제를 해결함 
    
    - IP Blocking Filter
        
        ```java
        @Slf4j
        @Component
        @RequiredArgsConstructor
        public class IpBlockingFilter implements Filter {
        
            private final FilterExceptionHandler filterExceptionHandler;
            private final KeyValueQueryRepository keyValueQueryRepository;
        
            private static final String BLOCK_PREFIX = "block:ip:";
        
            @Override
            public void doFilter(ServletRequest request, ServletResponse response, FilterChain chain)
                    throws IOException, ServletException {
        
                HttpServletRequest httpRequest = (HttpServletRequest) request;
                HttpServletResponse httpResponse = (HttpServletResponse) response;
                String ip = IpUtil.getClientIp(httpRequest);
                String redisBlockKey = BLOCK_PREFIX + ip;
        
                if (Boolean.TRUE.equals(keyValueQueryRepository.hasKey(redisBlockKey))) {
                    log.warn("차단된 IP 로그인 시도: {}", ip);
                    filterExceptionHandler.sendErrorResponse(httpResponse, HttpStatus.FORBIDDEN, "30초간 차단된 IP입니다.");
                    return;
                }
        
                chain.doFilter(request, response);
            }
        }
        
        ```
        
    
    ---
    
    ### 4. 구현 완료
    
    1. 이메일 인증으로 회원가입, 이메일 알림 구독, 비밀번호 리셋 가능
    
    ![스크린샷 2025-03-12 오후 11.16.03.png](attachment:b875d3e7-e074-4d25-bf97-eeaf76148129:%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2025-03-12_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_11.16.03.png)
    
    1. Kibana의 로그 대시보드로 이상 사용자의 IP를 확인하고, 지속적으로 올바르지 않은 요청을 보낸 IP는 WAF(Web Application Firewall)에서 차단 가능
    
    ![스크린샷 2025-03-12 오후 10.35.06.png](attachment:7dafc2f1-5593-47fe-8f2f-57aed736633c:%E1%84%89%E1%85%B3%E1%84%8F%E1%85%B3%E1%84%85%E1%85%B5%E1%86%AB%E1%84%89%E1%85%A3%E1%86%BA_2025-03-12_%E1%84%8B%E1%85%A9%E1%84%92%E1%85%AE_10.35.06.png)
    
    ---
    
    ### 5. 영향
    
    - **긍정적 영향**
        - **무분별한 공격에 대응 가능 →** 완벽하지는 않으나, 해커의 무분별한 공격을 막을 수 있음
        - **SendGrid API 횟수 조절 가능 →** SendGrid API 사용 횟수를 합리적으로 조절할 수 있음
    - **부정적 영향**
        - **단체 IP 밴 가능성 존재 →** 공용 IP에서 동시에 4명 이상 접속 시 단체로 IP 밴에 걸릴 수 있음
    
    ### **6. 향후 고려 사항**
    
    - **추가 대책 마련:** IP 밴은 30초밖에 안 되므로, 해커가 공용 IP를 사용할 때 대응책 마련 필요
</aside>

<aside>

- **MSA 분리 후 서버 간의 통신 방법 선택**
    
    ---
    
    ### 1. 배경
    
    초기에는 모놀리식 구조에서 개발을 시작했으며, **도메인 책임 분리**, **장애 격리**를 위해 MSA 구조로 전환했습니다. 이를 위해 **도메인 중심 수직 분리(Vertical Slice Architecture)** 와 모듈 기반 개발을 적용했고, 이후 각 도메인을 독립 서비스로 분리하여 통신이 필요한 구조로 변경되었습니다.
    
    이에 따라 서버 간 통신이 필수 요소가 되었습니다
    
    ---
    
    ### 2. 요구사항
    
    - 티켓팅 및 경매 시스템은 대규모 동시 트래픽이 집중되는 실시간 서비스 입니다. 특히 예매 오픈 시간에는 단 시간내에 수십만 건의 조회와 수천건의 예매 요청이 발생합니다.
    - 따라서 조금이라도 대용량의 트래픽에서 성능을 유지하기 위해서 서버 간의 통신에서도 성능을 높힐 필요가 있습니다!
    
    ---
    
    ### 3. 고려한 대안
    
    MSA 전환 이후 서버 간 데이터 전달을 위해 다음 세 가지 방식을 도입 및 테스트했습니다.
    
    | 항목 | Feign Client | gRPC | RSocket |
    | --- | --- | --- | --- |
    | 통신 방식 | HTTP/1.1 + JSON | HTTP/2 + Protobuf | TCP/WebSocket + Binary |
    | 구조 | 선언형 HTTP 인터페이스 | 고성능 RPC 프레임워크 | Reactive 비동기 메시징 |
    | 직렬화 | JSON (텍스트 기반) | Protobuf (이진) | 이진 |
    | 특징 | 설정 간단, Spring 친화적 | 빠르고 경량, 안정적 | 양방향 스트리밍, Backpressure 지원 |
    | 성능 | 낮음 (부하 시 병목 발생) | 우수 (낮은 지연 + 안정적) | 우수 (지연 최소화) |
    | 응답 처리 | 동기 처리 | 동기 or 비동기 | 기본 비동기 (Mono/Flux) |
    | 캐시 연동 | 쉬움 | 쉬움 | ⚠️ `.block()` 필요 → 병목 발생 가능 |
    | 사용 적합성 | 간단한 내부 호출 | 고성능 서버 간 통신 | 실시간 데이터 스트리밍 등 |
    
    ---
    
    ### 4. 실험 방법 및 결과 요약
    
    **공통 실험 환경**
    
    - 5분간 좌석 조회 약 20만 건 / 좌석 선점 약 5,000건
    - 초반 과부하 → 점차 감소하는 실제 예매 패턴 시뮬레이션
    - 매 테스트 전 캐시 삭제, 조회 쿼리 3종 랜덤 호출
    
    **결과 요약**
    
    | 통신 방식 | 평균 응답(ms) | P90 | P95 | 실패율 | 특징 |
    | --- | --- | --- | --- | --- | --- |
    | Feign | 10.78 | 13.41 | 14.99 | 0% |  |
    | Feign | 4.75 | 10.23 | 15.97 | 0% |  |
    | gRPC | 5.06 | 3.16 | 7.93 | 0% |  |
    | gRPC | 4.86 | 3.65 | 6.0 | 0% |  |
    | RSocket | 11.19 | 3.93 | 7.82 | 0% | 첫 캐시 gRPC 생성 |
    | RSocket | 3.46 | 3.99 | 5.99 | 0% | 첫 캐시 gRPC 생성 |
    
    ![image.png](attachment:441b0a34-79c5-445a-a9ad-9e536b0a4bc7:image.png)
    
    ---
    
    ### 5. 결정 및 근거
    
    위 테스트를 기반, RSocket과 gRPC 모두 Feign 대비 20~50% 정도의 빠른 응답 시간을 보여줌
    
    - RSocket은 기본적으로 **비동기 Mono/Flux 기반 응답 구조**
    - 그러나 Spring의 `@Cacheable`은 **동기 방식만 지원**
    - RSocket의 응답을 `.block()`으로 처리 시:
        - 비동기 처리 무력화
        - 이벤트 수신 지연 및 장애 가능성 (실제 Redis Pub/Sub 수신 실패 발생)
    
    → 결과적으로 RSocket은 **캐시 초기화 또는 갱신 로직과의 궁합이 좋지 않음**
    
    > gRPC를 서버 간 통신 방식으로 채택
    > 
    
    **결정 이유:**
    
    - Feign보다 우수한 성능 (P90/P95 기준 50% 이상 개선)
    - RSocket 수준의 응답 시간 확보
    - Spring 기반 구조와의 높은 호환성
    - 동기 캐시 로직과도 충돌 없이 통합 가능
    
    ---
    
    ### 6. 코드
    
    - 추가 예정
    
    ---
    
    ### 7. 향후 고려 사항
    
    - 실시간 스트리밍 또는 사용자 간 인터랙션 통신에는 향후 RSocket 별도 채택 가능
</aside>

<aside>

- **MSA 분리 후 캐시 전환 (내용 수정 예정)**
    
    ---
    
    ### 1. 배경
    
    기존에는 단일 서버 내에서 Caffeine 캐시를 사용하여 조회 응답 성능을 개선해 왔습니다. 하지만 MSA 구조로 전환되며, **서버 간 도메인이 분리되고 DB도 독립적으로 구성**됨에 따라 Caffeine처럼 인메모리 기반의 로컬 캐시는 **공유가 불가능한 구조**가 되었습니다.
    
    ---
    
    ### 2. 요구사항
    
    - MSA 환경에서는 **여러 인스턴스/서비스 간 캐시 동기화**가 필수
    - 실시간 티켓팅 시스템 특성상 조회 성능은 사용자 경험과 직결
    - 데이터 변경이 발생하는 서비스와 조회 서비스가 분리되어 있는 구조
    
    ---
    
    ### 3. 고려한 대안
    
    | 항목 | Caffeine (기존) | Redis (도입) |
    | --- | --- | --- |
    | 구조 | 단일 인스턴스 메모리 기반 | 네트워크 기반 글로벌 캐시 |
    | 공유 가능성 | ❌ 불가 | ✅ 가능 |
    | MSA 적합도 | 낮음 | 높음 |
    | 속도 | 빠름 (로컬) | 빠름 (네트워크 기준) |
    | 확장성 | 낮음 | 높음 |
    
    ---
    
    ### 4. 결정 및 근거
    
    - 좌석 예약/취소는 Ticket 서비스에서 발생
    - 하지만 캐시는 Game 서비스에 존재 → 데이터 변경이 발생해도 캐시 갱신 X
    - 결과적으로 캐시 불일치 발생
        - → Redis Pub/Sub 기반 이벤트 처리 방식 도입
            - Ticket 서비스에서 예매/취소 시 이벤트 발행
            - Game 서비스는 해당 이벤트 수신 후 캐시 갱신 수행
    
    개선 효과
    
    - 캐시 갱신 시점을 명확히 컨트롤 → 불일치 문제 해결
    - 서비스 간 느슨한 결합 구조 유지
    - 정상적으로 캐시 기능 구현 완료를 통한 성능 개선
    
    ---
    
    ### 5. 코드
    
    - 
    
    ---
    
    ### 6. 향후 고려 사항
    
    - Redis Pub/Sub의 메시지 영속성 한계 고려
    - 추후 Kafka 기반 메시징 시스템 전환 검토
        - 메시지 유실 방지
        - 멀티 구독자 처리 용이
        - 이벤트 리플레이 기능 등 추가 기능 확보
    
    ---
    
</aside>

<aside>

- **서버 모니터링 방법 선택**
    
    ---
    
    ### 1. 배경
    
    - 티켓팅을 수행하는 과정에서 많은 양의 트래픽이 몰리도록 테스트를 해봤는데, 서버가 버티지 못하고 다운되는 경우가 발생했습니다.
    - 이 상황을 보고 서비스 서버의 상태를 체크 해주는 모니터링 서버의 필요성을 느꼈습니다.
    
    ---
    
    ### 2. 요구사항
    
    - **실시간 모니터링** : 실시간으로 서버의 상태를 모니터링 할 수 있어야 한다.
    - **알림 기능** : 서버에 문제가 발생했을 때 알림을 받을 수 있어야 한다.
    
    ---
    
    ### 3. 고려한 대안
    
    | **비교 항목** | **장점** | **단점** |
    | --- | --- | --- |
    | **Prometheus** | 1. **실시간 서버 모니터링과 데이터 시각화에 뛰어나다.**
    2. **Grafana와 함께 사용하면 데이터 시각화가 가능해 실시간 성능 상태를 한 눈에 알아보기 쉽다.** | **주로 단기적인 모니터링에 최적화 되어 있기 때문에 장기적인 저장을 위해선 외부 로컬 스토리지가 필요하다.** |
    | **Zabbix** | **강력한 사용자 커뮤니티를 가지고 있으며, 높은 수준의 커스터마이징을 가지고 있어 서버 관리에 최적화 돼있다.** | **Spring boot Actuator 직접 연동 지원이 약하며, UI 반응성이 안좋고 초기 설정이 복잡하다.** |
    | **New Relic** | **서버의 성능 데이터 뿐만이 아닌 애플리케이션 데이터도 함께 제공해 다양한 관점에서 성능을 관리할 수 있다.** | **사용량과 데이터 양이 증가함에 따라 비용이 많이 들며, 기능이 너무 다양해 신규 사용자에게는 어려울 수 있다.** |
    
    ---
    
    ### 4. 결정 및 근거
    
    **✅ Prometheus + Grafana**
    
    - 오픈소스가 무료
    - 초기 설정이 굉장히 간편함
    - 요구 사항인 데이터 시각화가 뛰어나며, AlertManager로 알림 관리도 가능
    - 하드웨어의 상태와 커널 관련 메트릭을 수집하는 Node Exporter도 사용 가능
    - 하드웨어의 상태와 커널 관련 메트릭을 수집하는 Node Exporter도 사용 가능
    - **적용 이미지**
        - 서비스 서버 상태
            
            ![image.png](attachment:29c9169b-e007-4799-bdba-4442bfd95084:image.png)
            
        
        []()
        
        - 모니터링 서버 상태
        
        []()
        
        - 모니터링 서버 상태
        
        ![image.png](attachment:1546125c-148c-4b81-a810-179b3266b722:image.png)
        
    
    ---
    
    ### 5. 코드
    
    - **docker-compose.yml**
        
        ```java
        version: '3.8'
        
        services:
          prometheus:
            image: prom/prometheus:latest
            container_name: prometheus
            ports:
              - "9091:9090"
            volumes:
              - ./prometheus.yml:/etc/prometheus/prometheus.yml:ro
        
          grafana:
            image: grafana/grafana:latest
            container_name: grafana
            user: "admin"
            ports:
              - "3030:3000"
            volumes:
              - ./grafana-data:/var/lib/grafana
            depends_on:
              - prometheus
        
          node-exporter:
            image: prom/node-exporter:latest
            container_name: node-exporter
            restart: always
            network_mode: host
        
        volumes:
          grafana-data:
        ```
        
    - **prometheus.yml**
        
        ```java
        global:
          scrape_interval: 10s
        
        scrape_configs:
          - job_name: "spring-boot-server"
            metrics_path: /actuator/prometheus
            static_configs:
              - targets: ["3.39.237.240:8080"]
        
          - job_name: "node-exporter"
            static_configs:
              - targets: ["3.37.207.64:9100"]
        ```
        
    
    ---
    
    ### 6. 향후 고려 사항
    
    - **만약 예산이 충분하고 APM 심화 기능이 필요하다면 New Relic 도입을 검토 해보는게 좋을 것 같다.**
</aside>

<aside>

- **트래픽 제어를 위한 대기열 기능 도입**
    
    ### 1. 배경
    
    현재 시스템은 특정 기능(예: 티켓 예매)을 중심으로 **단시간에 많은 사용자의 요청이 집중될 가능성**이 존재합니다.
    
    이러한 요청 집중은 서버의 일시적인 과부하, DB 연결 수 초과, 주요 기능의 지연 또는 실패로 이어질 수 있으며,
    
    사용자 경험 저하와 서비스 신뢰도 하락의 원인이 될 수 있습니다.
    
    특히 이벤트성 기능은 특성상 **동시다발적인 요청 흐름이 발생하기 쉬워**, 이를 사전에 제어할 수 있는 기술적 장치가 필요하다고 판단했습니다.
    
    이에 따라, 요청을 일정량씩 순차적으로 처리하여 시스템 안정성을 확보하기 위한 **대기열(Queue) 기능** 도입을 검토하게 되었습니다.
    
    ---
    
    ### 2. 요구사항
    
    - **서버 보호**: 과도한 동시 요청으로부터 DB 및 서버 리소스를 보호해야 함
    - **요청 순서 보장**: 먼저 온 요청이 먼저 처리되는 **FIFO(First-In-First-Out)** 방식 필요
    
    ---
    
    ### 3. 고려한 대안
    
    | **비교 항목** | **캐싱 기반 응답 속도 향상** | **대기열 기반 순차 처리** |
    | --- | --- | --- |
    | **적용 목적** | 읽기 요청을 빠르게 응답 (부하 완화) | 쓰기 요청을 제어하며 순차 처리 |
    | **적용 대상** | 동일한 요청이 반복되는 **조회성 API** | 단건 처리 비용이 높은 **등록/수정/처리성 API** |
    | **시간 복잡도** | 매우 낮음 (O(1) 조회 가능) | O(1) 삽입 + 일정한 소비 주기 필요 |
    | **트래픽 제어 방식** | 요청 자체를 줄임 (캐시 적중 시) | 요청을 큐에 담아 **지연 처리** |
    | **사용자 경험** | ✅ 빠른 응답 |  "대기 중" 경험 유도 필요 |
    | **과부하 방지 효과** | ✅ Redis 적중률 높을수록 효과 큼 | ✅ 요청량을 분산시켜 DB 보호 |
    | **적용 난이도** | ✅ 상대적으로 간단 | ⚠️ 동시성, 스케줄링 등 고려사항 많음 |
    | **단점** | ❌ 최신성이 떨어질 수 있음 | ❌ 대기 시간이 길어지면 UX 저하 |
    
    ---
    
    ### 4. 결정 및 근거 - 대기열 도입
    
    **✅ 예매는 입력 작업이 동반**
    
    - 캐시는 트래픽 부하를 줄이기 위한 효과적인 수단이지만, 그 대상은 주로 **읽기(Read)** 요청에 국한됨
    - 그러나 **좌석 예매는 단순 조회를 넘어 서버 상태를 변경하는 입력(Write) 작업**이 포함되며, 이 입력 요청 또한 **과도하게 몰릴 경우 서버 과부하를 유발**
    - 특히 예매 흐름은 단순히 “좌석 조회 → 예매 요청”이 아니라, 많은 사용자가 동시에 조회를 시작하면서 **입력 이전 단계에서 이미 리소스가 고갈되기 시작함**
    
    따라서 다음과 같은 흐름으로 트래픽을 제어함:
    
    - 대기열 → 좌석 조회 → 예매 요청5. 대기열 설계 및 구현
    
    ---
    
    ### 5. 대기열 구현 방식
    
    | 비교 항목 | Java 내부 큐 (BlockingQueue 등) | Redis 기반 큐 | 메시지 큐 시스템 (Kafka, RabbitMQ) |
    | --- | --- | --- | --- |
    | 적용 난이도 | 간단 | 비교적 간단 | 설정 복잡, 외부 시스템 필요 |
    | 동시성 처리 | 단일 인스턴스 내에서만 가능 | 분산 환경에서도 가능 | 높은 수준의 병렬/비동기 처리 |
    | 확장성 | 인스턴스 로컬에 한정 | Redis 클러스터로 확장 가능 | 수평 확장 최적화 |
    | 지속성 | 서버 재시작 시 초기화 | Redis 설정에 따라 가능 | 내구성 보장 (디스크 기반 저장)결정 |
    
    | **비교 항목** | **Java 내부 큐 (BlockingQueue 등)** | **Redis List 기반 큐** | **메시지 큐 시스템 (Kafka, RabbitMQ)** |
    | --- | --- | --- | --- |
    
    | **적용 난이도** | ✅ 매우 쉬움 (의존성 없음) | ✅ 비교적 간단 | ⚠️ 설정 복잡, 외부 시스템 필요 |
    | --- | --- | --- | --- |
    
    | **동시성 처리** | ⚠️ 단일 인스턴스 내에서만 가능 | ✅ 분산 환경에서도 가능 | ✅ 높은 수준의 병렬/비동기 처리 |
    | --- | --- | --- | --- |
    
    | **확장성** | ❌ 인스턴스 로컬에 한정 | ✅ Redis 클러스터로 확장 가능 | ✅ 수평 확장 최적화 |
    | --- | --- | --- | --- |
    
    | **지속성** | ❌ 서버 재시작 시 초기화 | ⚠️ Redis 설정에 따라 가능 | ✅ 내구성 보장 (디스크 기반 저장) |
    | --- | --- | --- | --- |
    
    | **성능** | ✅ 빠름 (메모리 기반) | ✅ 빠름 + 외부 공유 가능 | ⚠️ 지연 가능 (복잡한 브로커 처리) |
    | --- | --- | --- | --- |
    
    | **운영 및 모니터링** | ✅ 별도 도구 불필요 | ⚠️ Redis 모니터링 필요 | ⚠️ 별도 대시보드 및 운영 필요 |
    | --- | --- | --- | --- |
    
    | **적합한 상황** | 단일 서버 기반 간단한 큐 | MSA 또는 분산 서비스에서 요청 제어 | 고신뢰 메시지 전달/이벤트 스트리밍 |
    | --- | --- | --- | --- |
    
    | **비교 항목** | **Java 내부 큐 (BlockingQueue 등)** | **Redis List 기반 큐** | **메시지 큐 시스템 (Kafka, RabbitMQ)** |
    | --- | --- | --- | --- |
    
    | **적용 난이도** | ✅ 매우 쉬움 (의존성 없음) | ✅ 비교적 간단 | ⚠️ 설정 복잡, 외부 시스템 필요 |
    | --- | --- | --- | --- |
    
    | **동시성 처리** | ⚠️ 단일 인스턴스 내에서만 가능 | ✅ 분산 환경에서도 가능 | ✅ 높은 수준의 병렬/비동기 처리 |
    | --- | --- | --- | --- |
    
    | **확장성** | ❌ 인스턴스 로컬에 한정 | ✅ Redis 클러스터로 확장 가능 | ✅ 수평 확장 최적화 |
    | --- | --- | --- | --- |
    
    | **지속성** | ❌ 서버 재시작 시 초기화 | ⚠️ Redis 설정에 따라 가능 | ✅ 내구성 보장 (디스크 기반 저장) |
    | --- | --- | --- | --- |
    
    | **성능** | ✅ 빠름 (메모리 기반) | ✅ 빠름 + 외부 공유 가능 | ⚠️ 지연 가능 (복잡한 브로커 처리) |
    | --- | --- | --- | --- |
    
    | **운영 및 모니터링** | ✅ 별도 도구 불필요 | ⚠️ Redis 모니터링 필요 | ⚠️ 별도 대시보드 및 운영 필요 |
    | --- | --- | --- | --- |
    
    | **적합한 상황** | 단일 서버 기반 간단한 큐 | MSA 또는 분산 서비스에서 요청 제어 | 고신뢰 메시지 전달/이벤트 스트리밍 |
    | --- | --- | --- | --- |
    
    ### **Redis기반 큐**
    
    대기열 구현 방식으로 **Redis 기반 큐**를 선택.
    
    Redis는 Java 내부 큐에 비해 **분산 환경에 더 적합하며**,
    
    Spring Boot와의 연동도 간단해 실제 서비스 환경에 빠르게 적용할 수 있습니다.
    
    또한 대기열의 순번 정보는 **결제나 트랜잭션 처리처럼 고신뢰 로그 보관이 필요한 데이터가 아닌**,
    
    **일시적인 트래픽 흐름을 제어하기 위한 용도**이므로,
    
    Kafka나 RabbitMQ 같은 메시지 브로커를 도입하는 것은 **복잡도와 비용 측면에서 오버엔지니어링에 해당한다고 판단**하였습니다.
    
    ---
    
    ### 6. 대기열 설계 및 구현
    
    - 1.  대기열 설계와 레디스 자료구조 선택
        
        ![image.png](attachment:6a17c7f6-9d07-40aa-b4bd-17d22c067946:image.png)
        
        **대기열(WaitingQueue)** 과 **작업열(ProceedQueue)** 두 단계로 나누어 설계
        
        ### 설계 개요
        
        - 사용자 요청 시, HTTP 헤더에 대기열 토큰(UUID)이 포함되어 있는지 확인
        - 토큰이 없는 경우, Redis ZSet을 기반으로 대기열에 새 토큰을 등록
        - 사용자에게는 **대기열 순번** 응답
        - 백그라운드 스케줄러가 Redis ZSet에서 앞쪽 토큰들을 일정 간격으로 꺼내 작업열로 이동
        - 작업열에 존재하는 토큰만 요청 처리 허용
        - 사용자는 polling을 통해 작업열 진입 여부 확인
        
        ### 대기열 (**WaitingQueue**)
        
        - Sorted Set(ZSet) 자료구조사용
        - **정렬된 순서를 유지하면서도 삽입/삭제/조회가 빠름**
        
        ### **작업열(ProceedQueue)**
        
        - String 자료구조 사용
        - String Key 존재 여부로 처리 가능 여부를 빠르게 판단 O(1)
        - 각각의 토큰에 TTL 설정을 통해 **시간 초과 자동 제거**가능
        
    - 2.  대기열(Waiting Queue)
        
        ```java
                @Service
        public class RedisWaitingQueueService implements WaitingQueueService {
        	private final ZSetOperations<String, String> waitingQueue;
        
        	public RedisWaitingQueueService(RedisTemplate<String, String> redisTemplate) {
        		waitingQueue = redisTemplate.opsForZSet();
        	}
        
        	//대기열 입장 후 토큰 반환
        	@Override
        	public String enterWaitingQueue() {
        		String token = UUID.randomUUID().toString();
        		long now = System.currentTimeMillis();
        		waitingQueue.addIfAbsent(QueueSystemConstants.WAITING_QUEUE_KEY, token, now);
        		return token;
        	}
        
        	//대기열 순서 조회
        	@Override
        	public long getOrder(String token) {
        		Long rank = waitingQueue.rank(QueueSystemConstants.WAITING_QUEUE_KEY, token);
        		return rank == null ? -1 : rank;
        	}
        
        	@Override
        	public void removeToken(String token) {
        		waitingQueue.remove(QueueSystemConstants.WAITING_QUEUE_KEY, token);
        	}
        }
        ```
        
    - 3.  작업열(ProceedQueue)
        
        ```java
                @Component
        public class RedisProceedQueueServiceV2 implements ProceedQueueService {
        
        	private final ValueOperations<String, String> proceedList;
        	private final RedisTemplate<String, String> scriptRunner;
        	private final DefaultRedisScript<Long> moveWaitingToProceedV2;
        
        	public RedisProceedQueueServiceV2(RedisTemplate<String, String> redisTemplate, DefaultRedisScript<Long> moveWaitingToProceedScriptV2) {
        		this.proceedList = redisTemplate.opsForValue();
        		this.scriptRunner = redisTemplate;
        		this.moveWaitingToProceedV2 = moveWaitingToProceedScriptV2;
        	}
        	
        	@Override
        	public boolean isContains(String token) {
        		return proceedList.get(QueueSystemConstants.PROCEED_QUEUE_KEY+":"+ token) != null;
        	}
        
        	@Override
        	public void removeToken(String token) {
        		proceedList.getOperations().delete(QueueSystemConstants.PROCEED_QUEUE_KEY+":"+ token);
        	}
        
        	@Override
        	public void pullFromWaitingQueue(Long targetSize) {
        		scriptRunner.execute(moveWaitingToProceedV2,
        			List.of(QueueSystemConstants.WAITING_QUEUE_KEY, QueueSystemConstants.PROCEED_QUEUE_KEY),
        			String.valueOf(targetSize));
        	}
        
        }
        ```
        
    - 4.  대기열과 작업열의 기능을 통합해서 흐름 제어
        
        ```java
        @Component
        public class QueueManager {
        	private final WaitingQueueService waitingQueueService;
        	private final ProceedQueueService proceedQueueService;
        
        	public QueueManager(WaitingQueueService waitingQueueService, @Qualifier("redisProceedQueueServiceV2") ProceedQueueService proceedQueueService) {
        		this.waitingQueueService = waitingQueueService;
        		this.proceedQueueService = proceedQueueService;
        	}
        	//대기열 입장
        	public String enterWaitingQueue() {
        		return waitingQueueService.enterWaitingQueue();
        	}
        
        	//대기순번 조회
        	public long getWaitingOrder(String token) {
        		long waitingOrder = waitingQueueService.getOrder(token);
        		boolean isProceed = proceedQueueService.isContains(token);
        		//대기열, 작업열 에 존재하지않으면 잘못된 토큰
        		if(waitingOrder == -1 && !isProceed) {
        			log.warn(" 토큰이 대기열 / 작업열에 모두 존재하지않습니다 : {}", token);
        			throw new ServerException(INVALID_WAITING_TOKEN);
        		}
        		return waitingOrder;
        	}
        
        	//해당 토큰이 작업이 가능한 상태인지 조회
        	public boolean isAllowed(String token) {
        		return proceedQueueService.isContains(token);
        	}
        
        	public void removeTokenFromProceedQueue(String token) {
        		proceedQueueService.removeToken(token);
        	}
        
        	public void deleteTokenFromWaitingAndProceedQueue(String token) {
        		waitingQueueService.removeToken(token);
        		proceedQueueService.removeToken(token);
        	}
        
        	public long getExpectedWaitingOrder(long currentWaitingOrder) {
        		double v = QueueSystemConstants.PROCEED_QUEUE_TARGET_SIZE * 0.8;
        		return (long) Math.floor(currentWaitingOrder / v) + 1;
        	}
        
        	@Scheduled(fixedRate = 1000)
        	public void moveWaitingToProceedAtomicScheduled(){
        		proceedQueueService.pullFromWaitingQueue(QueueSystemConstants.PROCEED_QUEUE_TARGET_SIZE);
        	}
        }
        ```
        
    
    ---
    
    ### 7.  도입 후 테스트
    
    - 테스트 개요
        - Tool : k6
        - vuser : 1000, duration : 300s
        - 대상 : /api/v1/games/${GAME_ID} ( 예매하기 전에 반드시 거쳐야하는 api )
        - polling 방식 / 간격 1s
        - 스케줄링 사용여부에관한 테스트
            - 대기열토큰을 작업열로 이동시키는 메서드를 스케줄링을 통해 이동시키는것과
            - polling 요청마다 이동시키는 방식 비교
        - 테스트는 항목별로 3번씩 진행
        
        ```java
        import http from 'k6/http';
        import { sleep } from 'k6';
        import { Counter } from 'k6/metrics';
        
        export const successCount = new Counter('success_count');
        
        const BASE_URL = 'http://localhost:8080';
        const GAME_ID = 1;
        
        const tokenMap = {};
        
        export const options = {
          vus: 1000,
          duration: '300s',
        };
        
        export default function () {
          const vu = __VU;
          let token = tokenMap[vu];
        
          const headers = token ? { 'waiting-token': token } : {};
          const res = http.get(`${BASE_URL}/api/v1/games/${GAME_ID}`, { headers });
        
          if (res.status === 200) {
            successCount.add(1);
            console.log(`[VU ${vu}] ✅ 입장 성공`);
            tokenMap[vu] = null;
          } else if (res.status === 202) {
            try {
              const body = JSON.parse(res.body);
              if (body.token && !token) {
                tokenMap[vu] = body.token;
                console.log(`[VU ${vu}] 신규 token 발급: ${body.token}`);
              } else if (body.order !== undefined) {
                console.log(`[VU ${vu}] 대기 중... 순번: ${body.order}`);
              }
            } catch (_) {
              console.error(`[VU ${vu}] 응답 파싱 실패`);
            }
          } else {
            console.error(`[VU ${vu}] ❌ 실패 응답: ${res.status}`);
          }
        
          sleep(1);
        }
        ```
        
    - 대기열X
        
        ```java
               /\      Grafana   /‾‾/
            /\  /  \     |\  __   /  /
           /  \/    \    | |/ /  /   ‾‾\
          /          \   |   (  |  (‾)  |
         / __________ \  |_|\_\  \_____/
        
             execution: local
                script: stress.js
                output: -
        
             scenarios: (100.00%) 1 scenario, 1000 max VUs, 5m30s max duration (incl. graceful stop):
                      * default: 1000 looping VUs for 5m0s (gracefulStop: 30s
        
          █ TOTAL RESULTS
        
            CUSTOM
            success_count...........................................................: 28184  90.576499/s
        
            HTTP
            http_req_duration.......................................................: avg=9.83s  min=121.39ms med=10.04s max=12.48s p(90)=10.2s p(95)=10.25s
              { expected_response:true }............................................: avg=9.83s  min=121.39ms med=10.04s max=12.48s p(90)=10.2s p(95)=10.25s
            http_req_failed.........................................................: 0.00%  0 out of 28184
            http_reqs...............................................................: 28184  90.576499/s
        
            EXECUTION
            iteration_duration......................................................: avg=10.84s min=1.14s    med=11.04s max=13.48s p(90)=11.2s p(95)=11.25s
            iterations..............................................................: 28184  90.576499/s
            vus.....................................................................: 19     min=19         max=1000
            vus_max.................................................................: 1000   min=1000       max=1000
        
            NETWORK
            data_received...........................................................: 15 MB  48 kB/s
            data_sent...............................................................: 2.8 MB 9.0 kB/s
        ```
        
        | 지표 | 값 |
        | --- | --- |
        | 총 요청 수 | 28,184 |
        | 성공 응답 수 (`success_count`) | 28,184 |
        | 요청 실패율 | **0.00% (0/28,184)** |
        | 평균 응답 시간 | **9.83초** |
        | 최소 응답 시간 | 121.39 ms |
        | 중간값 (median) | 10.04초 |
        | 최대 | 12.48초 |
        | 90th Percentile | 10.2초 |
        | 95th Percentile | 10.25초 |
        | 평균 요청 속도 | 90.57 req/s |
    - 대기열O, 작업열 토큰 이동 수 50
        
        ```java
                /\      Grafana   /‾‾/
            /\  /  \     |\  __   /  /
           /  \/    \    | |/ /  /   ‾‾\
          /          \   |   (  |  (‾)  |
         / __________ \  |_|\_\  \_____/
        
             execution: local
                script: stress.js
                output: -
        
             scenarios: (100.00%) 1 scenario, 1000 max VUs, 5m30s max duration (incl. graceful stop):
                      * default: 1000 looping VUs for 5m0s (gracefulStop: 30s)
        
          █ TOTAL RESULTS
        
            CUSTOM
            success_count...........................................................: 15646  51.96693/s
        
            HTTP
            http_req_duration.......................................................: avg=15.09ms min=1.05ms med=6.34ms max=758.58ms p(90)=17.43ms p(95)=81.94m
              { expected_response:true }............................................: avg=15.09ms min=1.05ms med=6.34ms max=758.58ms p(90)=17.43ms p(95)=81.94m
            http_req_failed.........................................................: 0.00%  0 out of 295217
            http_reqs...............................................................: 295217 980.539513/s
        
            EXECUTION
            iteration_duration......................................................: avg=1.01s   min=1s     med=1s     max=2.52s    p(90)=1.01s   p(95)=1.08s
            iterations..............................................................: 295217 980.539513/s
            vus.....................................................................: 15     min=15          max=1000
            vus_max.................................................................: 1000   min=1000        max=1000
        
            NETWORK
            data_received...........................................................: 137 MB 454 kB/s
            data_sent...............................................................: 44 MB  146 kB/s
        
        running (5m01.1s), 0000/1000 VUs, 295217 complete and 0 interrupted iterations
        default ✓ [======================================] 1000 VUs  5m0s
        ```
        
        | 항목 | 값 |
        | --- | --- |
        | **총 요청 수** | 295,217 |
        | 성공 응답 수 (`success_count`) | 15646 |
        | **요청 실패율** | 0.00% (0/295,217) |
        | **평균 응답 시간** | 15.09 ms |
        | **최소 응답 시간** | 1.05 ms |
        | **중간값 (median)** | 6.34 ms |
        | **최대 응답 시간** | 758.58 ms |
        | **90th percentile** | 17.43 ms |
        | **95th percentile** | 81.94 ms |
        | 평균 요청 속도 | 980.54 req/s |
    - 대기열O, 작업열 토큰 이동 수 100
        
        ```java
        
                 /\      Grafana   /‾‾/
            /\  /  \     |\  __   /  /
           /  \/    \    | |/ /  /   ‾‾\
          /          \   |   (  |  (‾)  |
         / __________ \  |_|\_\  \_____/
        
             execution: local
                script: stress.js
                output: -
        
             scenarios: (100.00%) 1 scenario, 1000 max VUs, 5m30s max duration (incl. graceful stop):
                      * default: 1000 looping VUs for 5m0s (gracefulStop: 30s)
        
          █ TOTAL RESULTS
        
            CUSTOM
            success_count...........................................................: 27291  90.596587/s
        
            HTTP
            http_req_duration.......................................................: avg=41.95ms min=1.49ms med=6.33ms max=2.18s p(90)=34.51ms p(95)=354.91ms
              { expected_response:true }............................................: avg=41.95ms min=1.49ms med=6.33ms max=2.18s p(90)=34.51ms p(95)=354.91ms
            http_req_failed.........................................................: 0.00%  0 out of 287962
            http_reqs...............................................................: 287962 955.933255/s
        
            EXECUTION
            iteration_duration......................................................: avg=1.04s   min=1s     med=1s     max=3.18s p(90)=1.04s   p(95)=1.36s
            iterations..............................................................: 287962 955.933255/s
            vus.....................................................................: 69     min=69          max=1000
            vus_max.................................................................: 1000   min=1000        max=1000
        
            NETWORK
            data_received...........................................................: 134 MB 446 kB/s
            data_sent...............................................................: 42 MB  140 kB/s
        
        running (5m01.2s), 0000/1000 VUs, 287962 complete and 0 interrupted iterations
        default ✓ [======================================] 1000 VUs  5m0s
        ```
        
        | 지표 | 값 |
        | --- | --- |
        | 총 요청 수 | 287,962 |
        | 성공 응답 수 (`success_count`) | **27,291** |
        | 요청 실패율 | 0.00% (0/287,962) |
        | 평균 응답 시간 | 41.95 ms |
        | 최소 응답 시간 | 1.49 ms |
        | 중간값 (median) | 6.33 ms |
        | 최대 | 2.18 s |
        | 90th Percentile | 34.51 ms |
        | 95th Percentile | 354.91 ms |
        | 평균 요청 속도 | 955.93 req/s |
    - 대기열O,  polling마다 토큰 이동 방식
        
        ```java
        
                 /\      Grafana   /‾‾/
            /\  /  \     |\  __   /  /
           /  \/    \    | |/ /  /   ‾‾\
          /          \   |   (  |  (‾)  |
         / __________ \  |_|\_\  \_____/
        
             execution: local
                script: stress.js
                output: -
        
             scenarios: (100.00%) 1 scenario, 1000 max VUs, 5m30s max duration (incl. graceful stop):
                      * default: 1000 looping VUs for 5m0s (gracefulStop: 30s)
        
          █ TOTAL RESULTS
        
            CUSTOM
            success_count...........................................................: 27709  91.785119/s
        
            HTTP
            http_req_duration.......................................................: avg=77.6ms min=3.21ms med=10.82ms max=2.26s p(90)=97.18ms p(95)=660.47ms
              { expected_response:true }............................................: avg=77.6ms min=3.21ms med=10.82ms max=2.26s p(90)=97.18ms p(95)=660.47ms
            http_req_failed.........................................................: 0.00%  0 out of 278199
            http_reqs...............................................................: 278199 921.524712/s
        
            EXECUTION
            iteration_duration......................................................: avg=1.08s  min=1s     med=1.01s   max=3.26s p(90)=1.1s    p(95)=1.67s
            iterations..............................................................: 278199 921.524712/s
            vus.....................................................................: 150    min=150         max=1000
            vus_max.................................................................: 1000   min=1000        max=1000
        
            NETWORK
            data_received...........................................................: 130 MB 430 kB/s
            data_sent...............................................................: 41 MB  135 kB/s
        ```
        
        | 지표 | 값 |
        | --- | --- |
        | 총 요청 수 | 278,199 |
        | 성공 응답 수 (`success_count`) | 27,709 |
        | 요청 실패율 | **0.00%** (0/278,199) |
        | 평균 | 77.6 ms |
        | 최소 | 3.21 ms |
        | 중간값 (median) | 10.82 ms |
        | 최대 | 2.26 s |
        | 90th Percentile | 97.18 ms |
        | 95th Percentile | 660.47 ms |
        | 평균 요청 속도 | 921.52 req/s |
        
    - 최종 결론
        1. redis cpu 점유율
            1. polling 마다 - 19.13 ~ 24.60 %
            2. 스케줄링 (500ms) :  11 ~13.7%    —  44% 효율 개선
        2. 대기열 적용 여부
            1. rps
                1. 대기열 적용 : 920 ~ 980 
                2. 대기열 X : 98
                3. 대기번호 발급에 의한 rps 뻥튀기  
                4. 다만, 매요청마다 빠른응답을 통한 사용자경험 개선 9초 → 0.3초 이내 97%이상 개선
        3. 다른 서비스 영향 여부
            1. 부하테스트 도중 다른 api를 호출했을때 응답시간 (Insomnia 사용)
                1. 대기열 적용 : 대부분의 요청이 300ms 이내에 응답
                2. 대기열 X : 9초 이상 지연
        4. 성공 응답 수(대기 순번X, api의 원래응답)
            1. 대기열을 적용하지 않은 경우에 성공 응답 수가 가장 큼
            2. 대기열을 적용했을경우 작업열의 용량과 성공 응답 수 비례하여 증가
                1. 다만, 용량이 클수록 일부 응답이 튀는 현상 발생
                2. pending 상태의 db connection 또한 크게 증가
    
    ---
    
    ### 8. 향후 고려 사항
    
    **복구 전략 수립**
    
    - Redis 다운 시 대기열 시스템 전체 마비 가능 → 클러스터 구성 or 대체 전략 필요
    - 토큰을 파일/DB 백업해 일시 복구 가능하도록 처리 고려
</aside>

<aside>

- **악성 유저와 봇 대처 방법 선택**
    
    ### 1. 배경
    
    - 모니터링을 구현하면서 만약 한 유저가 악의적으로 과도한 요청을 보내 공격을 하는 경우와(ex. DDos), 악성 봇 들이 계정을 마구잡이로 생성하는 경우에 대해 생각해봤다.
    - 과도한 요청을 막는 Rate Limiter 알고리즘과 악성 봇을 막는 google reCAPTCHA를 도입하도록 결정했다.
    
    ---
    
    ### 2. 요구사항
    
    - **많은 반복 요청 차단** : 한 유저가 한 요청을 빠르게 많이 요청할 때 차단
    - **악성 봇 차단** : 봇에 의한 로그인, 회원가입 차단
    
    ---
    
    ### 3. 주요 기능
    
    - **Rate Limiter**
        - **과도한 요청을 제한하고 제어하기 때문에 DDos 같은 공격을 막아준다.**
        - **최대 호출 수를 제한하기 때문에 총 호출에 대한 예상이 가능하며, 서버에 무리가 가지 않는 선으로 제한 기준을 설정하기 때문에 서버의 안정적인 관리가 가능하다.**
        - **요청에서 금액적인 부담이 커질 수 있는데, Rate Limit을 설정함으로써 비용에 대한 예상이 가능하고, 비용을 절감할 수 있다.**
    - **google reCAPTCHA**
        - **다양한 요소를 분석하여 사용자가 봇일 확률을 평가하고 이에 따라 응답을 조정한다.**
        - **정교한 기술을 사용해 사람과 봇을 구분하여 정확하게 식별한다.**
        - **머신러닝을 활용하여 봇 감지 정확도를 지속적으로 개선하고 새로운 위협에 적응한다.**
        - **조정 가능한 위험 분석 엔진을 제공하여 자동화 된 소프트웨어로부터 보호하고 조직의 웹사이트 및 모바일 앱 내에서 악의적인 활동을 저지한다.**
    
    ---
    
    ### 4. 결과 이미지
    
    - **Rate Limiter**
        - 로그인 요청 (1~2)
            
            ![image.png](attachment:8c44d1d6-305b-4e4b-934b-c7a14c038074:image.png)
            
        - 로그인 요청 (3~5)
        
        ![image.png](attachment:fcc937b3-128e-4b97-8d8e-18a82645daff:image.png)
        
    - **google reCAPTCHA**
        - 로봇이 아닙니다 버튼 안 누를 시
            
            ![image.png](attachment:0ce99666-bc53-4f7c-b787-e2b79461cbe9:image.png)
            
        - 버튼 누를 시
    
    ---
    
    ### 5. 코드
    
    - **Rate Limiter**
        - **KeyResolver로 유저의 Address를 가져옴**
            
            ```java
            @Bean
            	public KeyResolver ipKeyResolver() {
            		return exchange ->
            			Mono.just(Objects.requireNonNull(exchange.getRequest()
            					.getRemoteAddress())
            				.getAddress()
            				.getHostAddress());
            	}
            ```
            
        - **Gateway의 yml에서 redis token으로 해당 유저의 요청 수 확인 (일부분)**
            
            ```java
                      filters:
                        - name: RequestRateLimiter
                          args:
                            keyResolver: "#{@ipKeyResolver}"
                            redis-rate-limiter.replenishRate: 1   # 초당 최대 1 토큰 생성
                            redis-rate-limiter.burstCapacity: 2   # 2 토큰시 버스트
            ```
            
    - **google reCAPTCHA**
        - **reCAPTCHA 토큰이 있는지 검증 (버튼을 눌렀는지)**
            
            ```java
            @Service
            public class ReCaptchaService {
            	
            	private final WebClient webClient;
            	private final String secretKey;
            	
            	public ReCaptchaService(
            		WebClient.Builder builder,
            		@Value("${recaptcha.verify.url}") String verifyUrl,
            		@Value("${recaptcha.secret.key}") String secretKey
            	) {
            		this.webClient = builder.baseUrl(verifyUrl).build();
            		this.secretKey = secretKey;
            	}
            	
            	public boolean isValid(String token) {
            		if (token == null || token.isEmpty()) {
            			return false;
            		}
            		
            		RecaptchaResponse response = webClient.get()
            			.uri(uriBuilder -> uriBuilder
            				.queryParam("secret", secretKey)
            				.queryParam("response", token)
            				.build())
            			.retrieve()
            			.bodyToMono(RecaptchaResponse.class)
            			.block();
            		
            		return response != null && response.isSuccess();
            	}
            }
            ```
            
    
    ---
    
    ### 6. 향후 고려 사항
    
    - **reCAPTCHA v3 버전 사용**
    - **Rate Limiter 걸어줄 API들 고려**
</aside>

<aside>

- **데이터 조회에서의 방식 선택**
    
    ### 1. 배경
    
    경매 도메인에서는 하나의 경매가 티켓, 좌석, 결제 정보 등 여러 테이블과 조인을 통해 상세 정보를 구성해야 하며, 정렬 조건(예: 생성일 기준 최신순)에 따라 성능 이슈가 발생할 수 있습니다.
    
    특히 **MSA 환경에서는 단일 모놀리식 서버 구조보다 요청/응답 과정에서 네트워크 오버헤드가 추가되고**, 서비스 간 통신 및 데이터 호출 시에도 더 많은 리소스가 소모됩니다. 이러한 구조에서는 동일한 기능이라도 더 높은 효율성과 최적화가 필수입니다.
    
    이를 해결하기 위해 **반정규화(AuctionTicketInfo 테이블 분리)** 기법을 도입하여, 조회용 데이터를 사전에 구성해놓고 조인 없이 빠르게 조회할 수 있도록 구조를 변경하였습니다.
    
    ---
    
    ### 2. 요구사항
    
    - 경매 상세정보 조회는 페이지별로 대량 호출될 수 있으므로, 다음 조건을 만족하는 구조가 필요합니다:
        - 최신 생성일 기준 정렬
        - 좌석 수, 연석 여부, 경기 팀 등을 기준으로 조건 검색
        - 대량 트래픽 시 빠른 조회 보장
    
    ---
    
    ### 3. 실험 개요 및 비교 기준
    
    | **항목** | **정규화 조회 방식** | **반정규화 조회 방식** |
    | --- | --- | --- |
    | **조회 방식** | 각 테이블을 조인 후 실시간 계산 | AuctionTicketInfo 테이블로 캐싱된 정보만 조회 |
    | **조인 수** | 최대 5개 (Game, Ticket, Seat, Section, Payment 등) | 2개 (Auction, AuctionTicketInfo) |
    | **적용 기술** | JPQL + N+1 패치 방지용 fetch join | QueryDSL 단일 조인 |
    
    **측정 항목:**
    
    - 전체 조회 시간
    - 실행된 SQL 쿼리 수
    - SQL 최대 실행 시간
    - 페이지 사이즈 및 전체 데이터 수 증가에 따른 영향도
    
    ---
    
    ### 4. 실험 환경
    
    - 테스트 데이터: 1,000 ~ 100,000건 (Auction 기준)
    - 페이지 사이즈: 10, 100, 1000
    - 테스트 방식: `@SpringBootTest` 기반 통합 테스트
    - Hibernate Statistics 사용
    - 각 테스트는 `flush`/`clear`로 DB 상태 초기화 후 수행
    
    ---
    
    ### 5. 테스트 결과 요약
    
    | 데이터 수 | 페이지 사이즈 | 반정규화 실행시간 (ms) | 정규화 실행시간 (ms) | 속도 개선율 |
    | --- | --- | --- | --- | --- |
    | 1,000 | 10 | 30.3 | 88.4 | **65.7%** |
    | 1,000 | 100 | 55.1 | 387.6 | **85.8%** |
    | 1,000 | 1000 | 147.3 | 5286.4 | **97.2%** |
    | 10,000 | 10 | 131.1 | 881.9 | **85.1%** |
    | 10,000 | 100 | 155.2 | 1337.3 | **88.4%** |
    | 10,000 | 1000 | 252.0 | 7802.8 | **96.8%** |
    | 100,000 | 10 | 610.3 | 2110.9 | **71.1%** |
    | 100,000 | 100 | 655.8 | 2323.4 | **71.8%** |
    | 100,000 | 1000 | 728.9 | 6843.7 | **89.3%** |
    
    ---
    
    ### 6. 시각화 결과 요약
    
    ![반정규화 vs 정규화 조회속도기준 절감비율.png](attachment:6298eb00-88e3-4b9a-bfe4-2e34cacef48c:반정규화_vs_정규화_조회속도기준_절감비율.png)
    
    - **[반정규화 vs 정규화 조회 속도 그래프]**: 페이지가 커질수록 차이가 심화됨
    - **[조회 성능 절감 비율]**: 최대 97%까지 성능 개선
    - **[총 SQL 실행 시간]**: 정규화 방식에서 쿼리 건수가 많음에도 실행시간은 짧음 (단, 총합은 높음)
    - **[쿼리 수]**: 반정규화는 2~3개로 고정, 정규화는 최대 2000건 이상
    - **[쿼리당 평균 실행 시간]**: 정규화는 짧지만 총 수가 많음
    
    ---
    
    ### 7. 분석
    
    - **반정규화 방식의 장점**
        - 전체 조회 시간과 처리량에서 탁월한 성능
        - 조인이 거의 없고 QueryDSL에서 조건 검색만으로 해결됨
        - 실시간 트래픽이 집중되는 조회 API에 적합함
    - **단점 및 주의사항**
        - 데이터 일관성을 보장하려면 생성/수정 타이밍에 주의가 필요
        - 캐시와 유사하게 **데이터 싱크 이슈**가 발생할 수 있음
        - 하나의 쿼리로 많은 필드를 조회하므로 **쿼리 튜닝 포인트가 명확함**
        - 다만, 경매에 도달한 티켓은 필드의 변경이 크지 않을 것으로 예상.
    
    ---
    
    ### 8. 결론
    
    - **실시간으로 많은 조회가 발생하는 도메인에서는 반정규화 적용이 효과적**
    - **정규화 기반 조회 대비 평균 70~90% 이상 빠른 조회 속도를 보임**
    - 다만, 반정규화된 테이블을 유지보수 할 수 있도록 트리거 혹은 서비스 레이어에서 정확한 동기화 전략이 필요함
    
    ---
    
    ### 9. 향후 개선 및 고려 사항
    
    - CQRS 패턴 도입하여 조회 전용 테이블로 완전히 분리 가능
    - Kafka나 Event Stream으로 AuctionTicketInfo 자동 생성 가능
    - 정규화 방식은 관리자나 통계 시스템에서 보완적 조회에 사용

---

### 

---

</aside>

---

## 🚨 트러블 슈팅

<aside>

- 트러블 슈팅
    
    <aside>
    
    - @Cacheable 동작하지 않던 문제: AOP 프록시 한계와 서비스 분리로 해결
        
        ---
        
        ## 문제 : 캐시 미적용 이슈
        
        ## 문제 발생 배경
        
        티켓팅 시스템에서 사용자들이 좌석 예매를 시도할 때,
        
        **특정 경기의 좌석별 예매 여부 데이터**는 매우 짧은 시간 안에 대량으로 반복 조회됩니다.
        
        - 예: 경기 오픈 직후 수천 명이 동시에 "예매 가능한 좌석"을 조회하는 상황
        - 이러한 고부하 트래픽은 시스템의 성능을 단숨에 저하시킬 수 있습니다.
        
        따라서 우리는 **조회 성능을 극대화**하기 위해 쿼리 튜닝, 인덱스, 캐싱 도입 등 최적화 전략을 적용했습니다.
        
        하지만 @Cacheable을 통해 캐시를 적용했음에도 성능이 전혀 개선되지 않았습니다.
        
        API 응답 속도 역시 캐시 미적용 시와 동일했고 쿼리 로그를 확인하니 모든 요청마다 쿼리가 반복 실행되고 있었습니다.
        
        ![image.png](attachment:e2f739f0-5b0e-4dc9-b20c-6b72f8b0abb3:image.png)
        
        ---
        
        ## 문제 증상 요약
        
        - `@Cacheable`을 적용한 메서드가 **매번 DB 쿼리를 실행**
        - 캐시 저장 로그/조회 로그 없음
        - 캐시 개선 전 테스트와 평균 응답 시간 동일
        
        ---
        
        ## 원인 분석: Spring AOP 프록시 구조의 한계
        
        ### Spring AOP란?
        
        Spring AOP(Apect-Oriented Programming)는
        
        **핵심 비즈니스 로직과 공통 기능(트랜잭션, 캐싱, 로깅 등)을 분리**해
        
        애플리케이션을 더 모듈화할 수 있도록 도와주는 프로그래밍 기법입니다.
        
        Spring은 내부적으로 **프록시 객체**를 생성해 이런 기능을 끼워 넣습니다.
        
        > 예를 들어, @Cacheable이 붙은 메서드가 호출되면
        > 
        > 
        > → Spring은 먼저 "이전에 동일한 파라미터로 호출된 결과가 있나?"를 캐시에서 조회한 뒤
        > 
        > → 있다면 DB 대신 캐시 응답을 반환하고,
        > 
        > → 없다면 메서드를 실행한 뒤 결과를 캐시에 저장합니다.
        > 
        
        이 모든 과정은 **프록시(proxy)** 라는 중간 객체가 개입해서 자동으로 수행합니다.
        
        ### 문제의 본질: "자기 자신 호출 시 프록시를 거치지 않음"
        
        Spring AOP는 기본적으로 **프록시 객체를 통해 외부 호출이 이루어져야 동작**합니다.
        
        그러나 다음과 같은 코드처럼 **동일 클래스 내부에서 자신이 가진 메서드를 직접 호출**하면 프록시가 개입하지 못하게 됩니다.
        
        - 예제
            
            ```java
            @Service
            public class GameService {
            
                // 캐싱을 시도한 메서드
                @Cacheable(value = "seatCountsBySectionType", key = "#gameId")
                public List<SectionTypeSeatCountResponse> getSectionSeatCountsCached(Long gameId) {
                    // 호출될 때마다 DB 쿼리가 실행됨
                    return gameRepository.findUnBookedSeatsCountInSectionTypeByGameIdV2(gameId);
                }
            
                // 다른 비즈니스 로직에서 캐싱 대상 메서드 호출
                public StadiumGetResponse getStadiumAndSectionSeatCountsV2(Long gameId) {
                    List<SectionTypeSeatCountResponse> sectionBookedSeatCounts = getSectionSeatCountsCached(gameId); // ❌ 프록시 거치지 않음!
                    ...
                }
            }
            ```
            
        - 위 예제에서 `getSectionSeatCountsCached()`는 **@Cacheable이 붙어 있지만**,
            
            **같은 클래스 내부에서 직접 호출되었기 때문에 프록시를 거치지 않고**,
            
            결국 **AOP도 동작하지 않게 됩니다**.
            
        - 그래서 **캐시를 조회하거나 저장하는 로직이 아예 수행되지 않게 되는 것**입니다.
        
        해결 방안: **캐시 대상 메서드를 외부 서비스로 분리**
        
        ---
        
        ## 해결 방안: **캐시 대상 메서드를 외부 서비스로 분리**
        
        ### 전략: **프록시 객체가 개입할 수 있도록 구조 분리**
        
        - `@Cacheable`이 붙은 메서드를 **GameCacheService라는 별도 클래스**로 분리
        - `GameService`에서 해당 메서드를 호출할 때, 이제는 Spring이 생성한 **프록시 객체를 통해 호출**하게 되어
        **AOP 기능(@Cacheable)이 정상적으로 동작**
        - 코드
            
            ```java
            @Service
            public class GameCacheService {
            	@Cacheable(value = "seatCountsBySectionType", key = "#gameId")
            	public List<SectionTypeSeatCountResponse> getSectionSeatCountsCached(Long gameId) {
            		return gameRepository.findUnBookedSeatsCountInSectionTypeByGameIdV2(gameId);
            	}
            }
            
            @Service
            public class GameService {
                private final GameCacheService gameCacheService;
            
                public StadiumGetResponse getStadiumAndSectionSeatCountsV2(Long gameId) {
                    // 프록시 객체를 통해 캐시가 개입됨
                    List<SectionTypeSeatCountResponse> sectionBookedSeatCounts = gameCacheService.getSectionSeatCountsCached(gameId);
                    ...
                }
            }
            
            ```
            
        
        ---
        
        ## 테스트
        
        - k6를 활용하여 실제 티켓팅 환경과 유사하게 좌석 조회와 좌석 예매 로직을 동시에 실행
        - 최대 초당 30명의 좌석 예매와 초당 5000명의 좌석 정보 조회가 발생
        - 테스트 시나리오 코드
            
            ```java
            export let options = {
                scenarios: {
                    viewers: {
                        executor: 'ramping-arrival-rate',
                        startRate: 1000,
                        timeUnit: '1s',
                        preAllocatedVUs: 500,
                        maxVUs: 1000,
                        stages: [
                            { target: 5000, duration: '30s' },  // 티켓 오픈 직후
                            { target: 3000, duration: '30s' },  // 살짝 감소
                            { target: 1500, duration: '1m' },   // 안정화
                            { target: 800, duration: '1m' },    // 점차 줄어듦
                            { target: 200, duration: '2m' },    // 소강 상태
                            { target: 50, duration: '30s' },     // 마무리
                        ],
                        exec: 'seatView',
                    },
            
                    // 🎫 예매 시나리오 (reserveSeat)
                    bookers: {
                        executor: 'ramping-arrival-rate',
                        startRate: 1,
                        timeUnit: '1s',
                        preAllocatedVUs: 200,
                        maxVUs: 500,
                        stages: [
                            { target: 30, duration: '30s' },  // 예매자 몰림
                            { target: 15, duration: '30s' },  // 감소
                            { target: 10, duration: '1m' },    // 안정
                            { target: 3, duration: '1m' },    // 소강
                            { target: 2, duration: '2m' },    // 거의 없음
                            { target: 1, duration: '30s' },   // 종료
                        ],
                        exec: 'reserveSeat',
                    },
                },
            };
            ```
            
        - 테스트 결과
        - 캐시 적용 전
            
            ![image.png](attachment:556e84de-014b-4237-bc55-c229d3acb1d5:image.png)
            
        - 캐시 적용 후
            
            ![image.png](attachment:d738c4c0-bc79-4ad3-b132-ed3b8e44a993:83a0996b-f6bd-4fd1-817f-95eca6169dd7.png)
            
        
        ### 테스트 요약
        
        | 항목 | **캐시 미적용** | **캐시 적용 후** |
        | --- | --- | --- |
        | 🕒 **평균 응답 시간** | **6.05초** | **284ms** |
        | 🕒 **P90 응답 시간** | **~7.48초** | **~662ms** |
        | 🕒 **P95 응답 시간** | ~ 7.8초 | ~737ms |
        | ⚡ **처리 요청 수 (iterations)** | 55,562 | 약 120,000 이상 |
        | 📉 **부하 실패율 (http_req_failed)** | 0.56% | 0.11% 이하 |
        
        ![image.png](attachment:b18dca66-0056-4e69-822a-cbbd3d2a7df4:image.png)
        
        - 캐시 적용 전후를 비교한 결과, 평균 응답 시간은 약 **95% 단축** 되었고,
        - P90과 P95 응답 시간도 각각 **91% 90%** 개선되었습니다.
        - **전체적으로 90~95%의 성능 향상**을 확인할 수 있었습니다.
        - 또한 처리 요청 수는 2배 이상 증가했고 부하 실패율 역시 5배 감소했습니다!
        
        ---
        
        ## 배운점
        
        - Spring의 AOP 기반 어노테이션 (`@Cacheable`, `@Transactional`, `@Async` 등)은 **동일 클래스 내부 호출 시 동작하지 않는다.**
        - 이러한 동작 방식은 **프록시 패턴의 구조적 특성**에 기인하므로,
        → **설계 시 프록시 대상 메서드는 외부 서비스로 분리**하는 것이 안전
        - 단순히 어노테이션을 붙이는 것만으로 충분하지 않고 동작 방식에 대한 이해가 필요하다는 것을 깨달았다!
    </aside>
    
    <aside>
    
    - @Transactional 사용 시 불필요한 DB 커넥션 점유
        
        ---
        
        ## 문제 발생 배경
        
        티켓팅 시스템에서 예약 요청이 들어올 때, **모든 유저가 동일한 좌석을 예매하는 상황을 가정**하여 부하 테스트를 진행했습니다.
        
        - 핵심 전략은 DB 부하를 최소화하기 위해 **Redis에서 먼저 좌석 중복을 검증**하고,
            
            중복이 아닌 경우에만 DB에 예약 데이터를 생성하는 방식입니다.
            
        - 테스트 조건은 다음과 같습니다:
            - VUser 2000명
            - 총 50초간 테스트 진행
            - 모든 유저는 동일한 좌석 ID를 대상으로 예매 시도
        
        ## 문제 증상 요약
        
        ![image.png](attachment:80326072-c52f-453c-be91-084ab3af69a7:image.png)
        
        - 모든 요청이 Redis에서 중복으로 판별되어 **DB 작업은 수행되지 않았음에도**,
            
            DB 커넥션 풀(Pool)의 **Pending 커넥션 수가 200까지 증가**
            
        - 결국 DB 커넥션이 소진되며 다른 작업에 영향 발생 가능성 확인
        
        ## 원인 분석
        
         `@Transactional`이 DB 작업 전이라도 커넥션을 확보함
        
        Spring의 `@Transactional`은 단순히 트랜잭션 시작/커밋만 처리하는 것이 아니라,
        
        **트랜잭션이 시작되는 시점에서 Connection을 확보**하려고 합니다.
        
        ### 증명 테스트
        
        ```java
        @Transactional
        public void test() {
            // 아무 로직 없음
        }
        ```
        
        - 위와 같이 DB와 무관한 빈 메서드에 `@Transactional`을 붙인 후 동일한 조건으로 부하 테스트를 진행했음
        - 테스트 결과, DB 커넥션이 **동일하게 확보되는 현상 확인**
        
        ![image.png](attachment:c8faf94e-c24b-4406-8c13-a095f0dea266:image.png)
        
        👉 즉, 실제 DB 작업이 없더라도 @Transactional 어노테이션만 있어도
        
        Spring은 기본적으로 DB 커넥션을 점유하게 됩니다.
        
        ## 해결 방안: **트랜잭션 전 단계(선 검증 로직)와 DB 로직의 책임 분리**
        
        ### 구조 리팩터링
        
        - 기존에는 Redis 좌석 검증과 DB 예약 생성이 **하나의 `@Transactional` 메서드 내부에 공존**
        - 이를 **두 개의 서비스로 분리**하여, Redis 검증 로직은 트랜잭션 외부에서 수행되도록 개선
        - 리팩토링 전 코드
            
            하나의 트랜잭션에 모든 검증, 생서 로직이 모여있음
            
            ```java
            @Transactional
            	public ReservationResponse createReservation(AuthUser auth, ReservationCreateRequest reservationCreateRequest) {
            
            		//이미 예약된 좌석이 존재하는지 체크
            		ticketSeatService.checkDuplicateSeats(reservationCreateRequest.getSeatIds(),
            			reservationCreateRequest.getGameId());
            		//검증이 완료되면 좌석 점유
            		seatHoldRedisUtil.holdSeatAtomic(
            			reservationCreateRequest.getGameId(),
            			reservationCreateRequest.getSeatIds(),
            			String.valueOf(auth.getMemberId())
            		);
            
            		GameDto gameDto = null;
            		List<SeatDto> seats = new ArrayList<>();
            		try {
            			gameDto = gameClient.getGame(reservationCreateRequest.getGameId());
            			seats = seatClient.getSeatsByGameAndSection(
            				reservationCreateRequest.getGameId(),
            				reservationCreateRequest.getSectionId(),
            				reservationCreateRequest.getSeatIds()
            			);
            		} catch (Exception e) {
            			//경기, 좌석 정보를 가져오는도중 예외발생 시 선점했던 좌석을 해제
            			seatHoldRedisUtil.releaseSeatAtomic(reservationCreateRequest.getSeatIds(),
            				reservationCreateRequest.getGameId());
            			throw new ServerException(SEAT_NOT_FOUND);
            		}
            
            		int seatPrice = seats.stream().mapToInt(SeatDto::getPrice).sum();
            
            		//예약 데이터 생성
            		Reservation reservation = new Reservation(
            			auth.getMemberId(),
            			gameDto.getId(),
            			"WAITING_PAYMENT",
            			seatPrice
            		);
            		List<ReserveSeat> ReserveSeatList = ReserveSeat.from(reservationCreateRequest.getSectionId(), seats);
            		for (ReserveSeat reserveSeat : ReserveSeatList) {
            			reservation.addSeat(reserveSeat);
            		}
            		reservationRepository.save(reservation);
            
            		return ReservationResponse.from(reservation, gameDto, seats);
            	}
            ```
            
        - 리팩토링 후 코드
            
            트랜잭션이 필요한 작업(생성) 부분만 따로 서비스를 분리하여 예약로직 리팩토링
            
            ```java
            //예약 데이터 생성 서비스 분리
            public class ReservationCreateService {
            	private final ReservationRepository reservationRepository;
            	private final ReservationValidator reservationValidator;
            	private final SeatClient seatClient;
            
            	@Transactional
            	public ReservationResponse createReservation(AuthUser auth, ReservationCreateRequest reservationCreateRequest) {
            			//데이서 생성 로직
            	}
            }
            
            public class ReservationService {
            	public ReservationResponse processReserve(AuthUser auth, ReservationCreateRequest reservationCreateRequest) {
            			seatHoldRedisUtil.holdSeatAtomic(
            				reservationCreateRequest.getGameId(),
            				reservationCreateRequest.getSeatIds(),
            				String.valueOf(auth.getMemberId())
            			);
            			try {
            				ReservationResponse reservation = reservationCreateService.createReservation(
            					auth,
            					reservationCreateRequest
            				);
            				return reservation;
            			} catch (Exception e) {
            				seatHoldRedisUtil.releaseSeatAtomic(reservationCreateRequest.getSeatIds(),reservationCreateRequest.getGameId());
            				throw e;
            			}
            		}
            }
            	
            	
            ```
            
        
        ## 결과
        
        ![image.png](attachment:ad876269-a7ba-4aa8-b6ff-a229e161132e:image.png)
        
        - DB 커넥션 소모 없음
        
        ## 배운점
        
        - `@Transactional`은 **DB 쿼리 실행 여부와 무관하게 Connection을 점유**한다.
        - **트랜잭션이 반드시 필요한 로직과 그렇지 않은 로직은 책임을 분리**해야 한다.
        - Redis를 통한 선 필터링을 적용하더라도, `@Transactional`이 함께 적용되어 있다면 **불필요한 커넥션 소모로 병목이 발생**할 수 있다.
        - 대량 요청 트래픽을 처리하는 시스템에서는 작은 설정 하나가 병목 지점이 될 수 있으므로, **JDBC 커넥션 사용 시점에 대한 명확한 이해가 필수**이다.
    
    ## 원인 분석: `@Transactional`이 DB 작업 전이라도 커넥션을 확보함
    
    Spring의 `@Transactional`은 단순히 트랜잭션 시작/커밋만 처리하는 것이 아니라,
    
    **트랜잭션이 시작되는 시점에서 Connection을 확보**하려고 합니다.
    
    </aside>
    
    <aside>
    
    - **이벤트 기반 캐시 동기화 구조 도입 배경과 리팩토링 과정**
        
        ---
        
        ### 1. 문제 요약
        
        경매 입찰 시스템에서 **DB 입찰가와 Redis 캐시 입찰가 간 불일치로 인해 사용자 입찰이 반복적으로 실패하는 이슈**가 발생했습니다.
        
        특히 다음과 같은 상황이 문제의 핵심이었습니다:
        
        - DB 입찰가는 `2000원`, Redis 입찰가는 `2100원`으로 조회
        - 사용자는 분명히 입찰 조건을 만족했지만 계속해서 실패 처리됨
        
        결국 이는 **데이터 정합성 문제**였고, 캐시와 트랜잭션의 비동기적 처리 순서에서 발생한 이슈로 파악되었습니다.
        
        ---
        
        ### 2. 문제 발생 배경
        
        기존 구조에서는 입찰 등록 시 다음과 같은 흐름으로 처리되었습니다:
        
        1. 사용자의 입찰 요청 수신
        2. DB에 입찰 내역 저장 (비관적 락 적용)
        3. **입찰가를 Redis에 업데이트**
        
        이 구조는 한 가지 중요한 문제를 내포하고 있었습니다.
        
        > "DB 트랜잭션이 아직 커밋되지 않았는데, Redis에는 입찰가가 먼저 반영되는 상황"
        > 
        
        즉, **트랜잭션 커밋 전에 Redis에 입찰가를 먼저 업데이트해버렸고**,
        
        이후 트랜잭션이 롤백되거나 타임아웃이 발생하면 DB와 Redis 간 입찰가가 어긋나게 되는 것입니다.
        
        ### 🔍 왜 DB는 2000원인데 Redis는 2100원이었을까?
        
        경매 입찰 기능에서는 Redis에 저장된 **현재 입찰가**가 기준이 됩니다. 사용자가 입찰을 시도하면 다음 두 가지 비교가 동시에 일어납니다:
        
        - Redis의 입찰가와 동일한 금액인가? ✅
        - DB에 저장된 입찰가와 동일한 금액인가? ✅
        
        하지만 당시에는 다음과 같은 문제가 있었습니다:
        
        1. **입찰 요청 처리 중 트랜잭션이 커밋되기 전에**, Redis의 입찰가가 먼저 갱신되었습니다.
        2. 하지만 DB 트랜잭션이 중간에 실패하거나 롤백되면…
            - Redis는 이미 2100원으로 올라간 상태
            - DB는 여전히 2000원
        3. 이 상태에서 다른 사용자가 입찰을 시도하면:
            - Redis 기준으로는 2100원을 요구하지만
            - DB는 아직 2000원이므로 **불일치 상태가 발생하고, 입찰 실패**
        
        즉, **Redis와 DB의 상태가 일시적으로 분리되었고**, 비즈니스 로직은 Redis를 우선 기준으로 판단하다 보니 최신화 되지 않은 데이터로 간주되어 실패가 반복된 것입니다.
        
        ---
        
        ### 3. 원인 분석: 트랜잭션 커밋 전에 Redis 업데이트
        
        | **순번** | **작업** | **설명** |
        | --- | --- | --- |
        | **1** | 입찰 요청 수신 | 사용자가 입찰가를 제시 |
        | **2** | DB 저장 시도 | `@Transactional` 안에서 입찰가, 입찰자 등 저장 |
        | **3** | **Redis 입찰가 업데이트** | DB 저장 성공 여부와 관계없이 실행됨 ❌ |
        | **4** | 커넥션 타임아웃 또는 롤백 | DB에는 반영 안 됨, Redis에는 반영됨 ❌ |
        | **5** | 사용자 재입찰 시도 | Redis 입찰가 기준으로 최신화 되지 않아 예외 처리됨 ❌ |
        
        즉, **트랜잭션이 실제로 완료되기 전에 Redis에 값이 먼저 반영되는 구조**가 가장 핵심적인 문제였습니다.
        
        ---
        
        ### 4. 개선 전략: **이벤트 기반 처리 + 트랜잭션 후처리 적용**
        
        ### ✅ 개선 전 구조
        
        ```
        Bid 요청 → DB 저장 → Redis 즉시 업데이트 → 응답
        
        ```
        
        - 문제: 트랜잭션 커밋 여부와 관계없이 Redis가 업데이트됨
        
        ### ✅ 개선 후 구조 (도입한 구조)
        
        ```
        Bid 요청 → DB 저장 → ApplicationEvent 발행 → 응답
                                          ↓
                             (트랜잭션 커밋 이후 실행)
                            Redis 입찰가 업데이트 (BidEventListener)
        
        ```
        
        - 핵심: `@TransactionalEventListener(phase = AFTER_COMMIT)`를 통해 **트랜잭션이 성공한 경우에만 Redis를 갱신**
        
        ```java
        @EventListener
        @TransactionalEventListener(phase = TransactionPhase.AFTER_COMMIT)
        public void updateBidCache(BidUpdateEvent event) {
            auctionBidRedisUtil.setBidPoint(event.getAuctionId(), event.getBidPoint());
        }
        
        ```
        
        ---
        
        ### 5. 테스트 결과
        
        **기존 구조에서의 문제 재현**
        
        - 인위적으로 트랜잭션 예외 발생 (e.g. `throw new RuntimeException()`)
        - Redis 입찰가는 업데이트 되었으나, DB는 반영되지 않음
        - 이후 입찰자는 계속 409 오류 (입찰가 낮음) 발생
        
        **이벤트 구조 도입 이후**
        
        - 트랜잭션이 실패할 경우 Redis는 그대로 유지
        - 트랜잭션이 성공한 경우에만 Redis 갱신
        - DB와 Redis 간의 정합성 유지
        
        ---
        
        ### 6. 배운 점 및 적용 효과
        
        | 항목 | 개선 전 | 개선 후 |
        | --- | --- | --- |
        | Redis 값 반영 시점 | 트랜잭션 도중 | 트랜잭션 커밋 이후 |
        | 데이터 불일치 가능성 | 매우 높음 | 거의 없음 |
        | 입찰 오류 발생률 | 높음 | 없음 |
        | 구조 안정성 | 취약 | 안정적 |
        
        ---
        
        ### 7. 향후 개선 사항
        
        - **Redis 갱신 실패 대응**: 이벤트 리스너 내부에서 Redis 장애 시 재시도 전략 필요
        - **분산 락 연계**: 입찰 갱신 시점에 레디스 락을 함께 적용해 경쟁 조건 해소
        - **모니터링 추가**: 입찰 시 Redis/DB 간 불일치 감지 로깅 추가 예정
    </aside>
    
    <aside>
    
    - .
        
        ---
        
        ## 1. 리팩토링 전
        
        - 리팩토링 전 코드
            
            ```java
            public interface KeyValueRepository {
            
                //Key-Value
                String getValue(String key);
                Set<String> getKeys(String key);
            
                void setValue(String key, String value);
                void setValue(String key, String number, Duration duration);
                void setValue(String key, String value, long ttl, TimeUnit timeUnit);
            
                void removeValue(String key);
            
                long incrementValue(String key);
            
                void expireValue(String key, long ttl, TimeUnit timeUnit);
            
                Boolean hasKey(String key);
            
                List<String> opsForListRange(String key, long start, long end);
                void opsForListLeftPush(String key, String value);
            
                Set<String> opsForZSet(String key, long start, long end);
                void opsForZSetAdd(String key, String value, long score);
                Long opsForZSetCard(String key);
                Set<String> opsForZSetReverseRange(String key, long start, long end);
                void opsForZSetRemoveRange(String key, long start, long end);
            
            }
            
            ```
            
        
        ---
        
        ## 2. 현재 코드의 문제점
        
        ### ⚠️ **SRP 위반!**
        
        - 하나의 Repository에 너무 많은 메서드가 있고, 이들의 공통점은 Key-value 저장소를 사용한다는 점뿐입니다.
        - 새로운 팀원이 합류한다면, 어떤 메서드를 어디서 찾아서 사용해야 할지 헷갈립니다.
        
        ---
        
        ## 3. 리팩토링 과정
        
        ♻️ CQRS 아이디어를 차용해 볼까요?
        
        - 읽기 작업과 쓰기 작업으로 분리합시다!
            - 읽기(Query) 작업을 담는 KeyValueQueryRepository
                
                ```java
                public interface KeyValueQueryRepository {
                
                    //읽기 전용
                    String getValue(String key);
                    Set<String> getKeys(String key);
                    Boolean hasKey(String key);
                    List<String> getListRange(String key, long start, long end);
                    Set<String> getZSetRange(String key, long start, long end);
                    Long getZSetCard(String key);
                    Set<String> getZSetReverseRange(String key, long start, long end);
                }
                
                ```
                
            - 쓰기(Command) 작업을 담는 KeyValueCommandRepository
                
                ```java
                public interface KeyValueCommandRepository {
                
                    //쓰기 전용
                    void setValue(String key, String value);
                    void setValue(String key, String value, Duration duration);
                    void setValue(String key, String value, long ttl, TimeUnit timeUnit);
                    void removeValue(String key);
                    long incrementValue(String key);
                    void expireValue(String key, long ttl, TimeUnit timeUnit);
                    void pushToListLeft(String key, String value);
                    void addToZSet(String key, String value, long score);
                    void removeFromZSetRange(String key, long start, long end);
                }
                
                ```
                
        
        ---
        
        ## 4. 리팩토링 결과
        
        ✅ 로직을 나누어 협업하기 쉬워졌어요!
        
        ✅ 유지 보수하기 편해졌어요!
        
        ✅ 패턴이 명확하여 다른 Key-Value 저장소로 이전하기 쉬워졌어요!
        
    </aside>
    
    <aside>
    
    - .
        
        ---
        
        ## 1. 리팩토링 전
        
        - 리팩토링 전 코드
            
            ```java
            public interface KeyValueRepository {
            
                //Key-Value
                String getValue(String key);
                Set<String> getKeys(String key);
            
                void setValue(String key, String value);
                void setValue(String key, String number, Duration duration);
                void setValue(String key, String value, long ttl, TimeUnit timeUnit);
            
                void removeValue(String key);
            
                long incrementValue(String key);
            
                void expireValue(String key, long ttl, TimeUnit timeUnit);
            
                Boolean hasKey(String key);
            
                List<String> opsForListRange(String key, long start, long end);
                void opsForListLeftPush(String key, String value);
            
                Set<String> opsForZSet(String key, long start, long end);
                void opsForZSetAdd(String key, String value, long score);
                Long opsForZSetCard(String key);
                Set<String> opsForZSetReverseRange(String key, long start, long end);
                void opsForZSetRemoveRange(String key, long start, long end);
            
            }
            
            ```
            
        
        ---
        
        ## 2. 현재 코드의 문제점
        
        ### ⚠️ **SRP 위반!**
        
        - 하나의 Repository에 너무 많은 메서드가 있고, 이들의 공통점은 Key-value 저장소를 사용한다는 점뿐입니다.
        - 새로운 팀원이 합류한다면, 어떤 메서드를 어디서 찾아서 사용해야 할지 헷갈립니다.
        
        ---
        
        ## 3. 리팩토링 과정
        
        ♻️ CQRS 아이디어를 차용해 볼까요?
        
        - 읽기 작업과 쓰기 작업으로 분리합시다!
            - 읽기(Query) 작업을 담는 KeyValueQueryRepository
                
                ```java
                public interface KeyValueQueryRepository {
                
                    //읽기 전용
                    String getValue(String key);
                    Set<String> getKeys(String key);
                    Boolean hasKey(String key);
                    List<String> getListRange(String key, long start, long end);
                    Set<String> getZSetRange(String key, long start, long end);
                    Long getZSetCard(String key);
                    Set<String> getZSetReverseRange(String key, long start, long end);
                }
                
                ```
                
            - 쓰기(Command) 작업을 담는 KeyValueCommandRepository
                
                ```java
                public interface KeyValueCommandRepository {
                
                    //쓰기 전용
                    void setValue(String key, String value);
                    void setValue(String key, String value, Duration duration);
                    void setValue(String key, String value, long ttl, TimeUnit timeUnit);
                    void removeValue(String key);
                    long incrementValue(String key);
                    void expireValue(String key, long ttl, TimeUnit timeUnit);
                    void pushToListLeft(String key, String value);
                    void addToZSet(String key, String value, long score);
                    void removeFromZSetRange(String key, long start, long end);
                }
                
                ```
                
        
        ---
        
        ## 4. 리팩토링 결과
        
        ✅ 로직을 나누어 협업하기 쉬워졌어요!
        
        ✅ 유지 보수하기 편해졌어요!
        
        ✅ 패턴이 명확하여 다른 Key-Value 저장소로 이전하기 쉬워졌어요!
        
    </aside>
    
    <aside>
    
    - .
        
        ---
        
        ## 1. 리팩토링 전
        
        - 리팩토링 전 코드
            
            ```java
            public interface KeyValueRepository {
            
                //Key-Value
                String getValue(String key);
                Set<String> getKeys(String key);
            
                void setValue(String key, String value);
                void setValue(String key, String number, Duration duration);
                void setValue(String key, String value, long ttl, TimeUnit timeUnit);
            
                void removeValue(String key);
            
                long incrementValue(String key);
            
                void expireValue(String key, long ttl, TimeUnit timeUnit);
            
                Boolean hasKey(String key);
            
                List<String> opsForListRange(String key, long start, long end);
                void opsForListLeftPush(String key, String value);
            
                Set<String> opsForZSet(String key, long start, long end);
                void opsForZSetAdd(String key, String value, long score);
                Long opsForZSetCard(String key);
                Set<String> opsForZSetReverseRange(String key, long start, long end);
                void opsForZSetRemoveRange(String key, long start, long end);
            
            }
            
            ```
            
        
        ---
        
        ## 2. 현재 코드의 문제점
        
        ### ⚠️ **SRP 위반!**
        
        - 하나의 Repository에 너무 많은 메서드가 있고, 이들의 공통점은 Key-value 저장소를 사용한다는 점뿐입니다.
        - 새로운 팀원이 합류한다면, 어떤 메서드를 어디서 찾아서 사용해야 할지 헷갈립니다.
        
        ---
        
        ## 3. 리팩토링 과정
        
        ♻️ CQRS 아이디어를 차용해 볼까요?
        
        - 읽기 작업과 쓰기 작업으로 분리합시다!
            - 읽기(Query) 작업을 담는 KeyValueQueryRepository
                
                ```java
                public interface KeyValueQueryRepository {
                
                    //읽기 전용
                    String getValue(String key);
                    Set<String> getKeys(String key);
                    Boolean hasKey(String key);
                    List<String> getListRange(String key, long start, long end);
                    Set<String> getZSetRange(String key, long start, long end);
                    Long getZSetCard(String key);
                    Set<String> getZSetReverseRange(String key, long start, long end);
                }
                
                ```
                
            - 쓰기(Command) 작업을 담는 KeyValueCommandRepository
                
                ```java
                public interface KeyValueCommandRepository {
                
                    //쓰기 전용
                    void setValue(String key, String value);
                    void setValue(String key, String value, Duration duration);
                    void setValue(String key, String value, long ttl, TimeUnit timeUnit);
                    void removeValue(String key);
                    long incrementValue(String key);
                    void expireValue(String key, long ttl, TimeUnit timeUnit);
                    void pushToListLeft(String key, String value);
                    void addToZSet(String key, String value, long score);
                    void removeFromZSetRange(String key, long start, long end);
                }
                
                ```
                
        
        ---
        
        ## 4. 리팩토링 결과
        
        ✅ 로직을 나누어 협업하기 쉬워졌어요!
        
        ✅ 유지 보수하기 편해졌어요!
        
        ✅ 패턴이 명확하여 다른 Key-Value 저장소로 이전하기 쉬워졌어요!
        
    </aside>
    
    <aside>
    
    - .
        
        ---
        
        ## 1. 리팩토링 전
        
        - 리팩토링 전 코드
            
            ```java
            public interface KeyValueRepository {
            
                //Key-Value
                String getValue(String key);
                Set<String> getKeys(String key);
            
                void setValue(String key, String value);
                void setValue(String key, String number, Duration duration);
                void setValue(String key, String value, long ttl, TimeUnit timeUnit);
            
                void removeValue(String key);
            
                long incrementValue(String key);
            
                void expireValue(String key, long ttl, TimeUnit timeUnit);
            
                Boolean hasKey(String key);
            
                List<String> opsForListRange(String key, long start, long end);
                void opsForListLeftPush(String key, String value);
            
                Set<String> opsForZSet(String key, long start, long end);
                void opsForZSetAdd(String key, String value, long score);
                Long opsForZSetCard(String key);
                Set<String> opsForZSetReverseRange(String key, long start, long end);
                void opsForZSetRemoveRange(String key, long start, long end);
            
            }
            
            ```
            
        
        ---
        
        ## 2. 현재 코드의 문제점
        
        ### ⚠️ **SRP 위반!**
        
        - 하나의 Repository에 너무 많은 메서드가 있고, 이들의 공통점은 Key-value 저장소를 사용한다는 점뿐입니다.
        - 새로운 팀원이 합류한다면, 어떤 메서드를 어디서 찾아서 사용해야 할지 헷갈립니다.
        
        ---
        
        ## 3. 리팩토링 과정
        
        ♻️ CQRS 아이디어를 차용해 볼까요?
        
        - 읽기 작업과 쓰기 작업으로 분리합시다!
            - 읽기(Query) 작업을 담는 KeyValueQueryRepository
                
                ```java
                public interface KeyValueQueryRepository {
                
                    //읽기 전용
                    String getValue(String key);
                    Set<String> getKeys(String key);
                    Boolean hasKey(String key);
                    List<String> getListRange(String key, long start, long end);
                    Set<String> getZSetRange(String key, long start, long end);
                    Long getZSetCard(String key);
                    Set<String> getZSetReverseRange(String key, long start, long end);
                }
                
                ```
                
            - 쓰기(Command) 작업을 담는 KeyValueCommandRepository
                
                ```java
                public interface KeyValueCommandRepository {
                
                    //쓰기 전용
                    void setValue(String key, String value);
                    void setValue(String key, String value, Duration duration);
                    void setValue(String key, String value, long ttl, TimeUnit timeUnit);
                    void removeValue(String key);
                    long incrementValue(String key);
                    void expireValue(String key, long ttl, TimeUnit timeUnit);
                    void pushToListLeft(String key, String value);
                    void addToZSet(String key, String value, long score);
                    void removeFromZSetRange(String key, long start, long end);
                }
                
                ```
                
        
        ---
        
        ## 4. 리팩토링 결과
        
        ✅ 로직을 나누어 협업하기 쉬워졌어요!
        
        ✅ 유지 보수하기 편해졌어요!
        
        ✅ 패턴이 명확하여 다른 Key-Value 저장소로 이전하기 쉬워졌어요!
        
    </aside>
    
</aside>
