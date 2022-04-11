# login_session
# 로그인 처리하기 - 쿠키 사용
**쿠키**
- 로그인의 상태를 유지하기 위해 서버에서 로그인 성공 시 HTTP 응답에 쿠키를 담아서 브라우저에 전달.
- 앞으로 브라우저는 해당 쿠키를 지속해서 전송

![image](https://user-images.githubusercontent.com/96407257/162555172-dd5923fb-d7b5-4c22-ab81-e5a1f2c94a39.png)

![image](https://user-images.githubusercontent.com/96407257/162555181-65b2c565-77b1-4fc7-a35c-f8fde51f17c5.png)

- 쿠키는 **영속 쿠키**와 **세션 쿠키**가 존재
- 영속 쿠키 : 만료 날짜를 입력하면 해당 날짜까지 유지
- 세션 쿠키 : 만료 날짜를 생략하면 브라우저 종료 시 까지만 유지

**쿠키 생성 로직**

    Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));
    response.addCookie(idCookie);
    
- 로그인 성공 시 쿠키를 생성하고 HttpServletResponse에 담으며, 쿠키 이름은 memberId, 값은 회원의 id
- 웹 브라우저 종료 전까지 회원의 id를 서버에 계속 전송

**homeController**

    @GetMapping("/")
    public String homeLoginV1(@CookieValue(name = "memberId", required = false) Long memberId, Model model) {

        if (memberId == null) {
            return "home";
        }

        Member loginMember = memberRepository.findById(memberId);
        if (loginMember == null) {
            return "home";
        }

        model.addAttribute("member", loginMember);
        return "loginHome";
    }
    
- @CookieValue 에노테이션을 사용하여 편리하게 쿠키 조회
- 로그인 하지 않는 사용자도 홈에 접근이 가능하므로 required = false를 사용

**로직 분석**
- 로그인 쿠키(memberId)가 없는 사용자는 기존 home으로 보내고, 로으인 쿠키가 있어도 회원이 없으면 home 으로 전송
- 로그인 쿠키(memberId)가 있는 사용자는 로그인 사용자 전용 홈 화면인 loginHome으로 전송
- 홈 화면에 회원 관련 정보 출력을 위해 member 데이터도 모델에 담아서 전송

**로그아웃 기능**
- 세션 쿠키이므로 웹 브라우저 종료 시 로그아웃
- 서버에서 해당 쿠키의 종료 날짜를 0으로 지정

# 쿠키와 보안 문제
**보안 문제**
- 쿠키 값을 임의로 변경 가능
  - 클라이언트가 쿠키를 강제로 변경하면 다른 사용자로 변경
  - 실제 웹브라우저 개발자모드에서 쿠키 변경 시 다른 사용자의 이름 확인 가능
- 쿠키에 보관된 정보 확인 가능
  - 쿠키에 개인정보, 신용카드 정보 등이 웹 브라우저에도 보관되고, 네트워크 요청마다 계속 클라이언트에서 서버로 전달
  - 쿠키의 정보가 로컬 PC, 네트워크 전송 구간에서 유출 가능
- 해커가 쿠키를 한번 훔쳐가면 평생 사용 가능
  - 쿠키가 매번 리셋되지 않아 보안이 취약

**대안**
- 쿠키에 중요 값을 노출하지 않고, 사용자 별로 예측 불가능한 임의의 토큰(랜덤 값)을 노출, 서버에서 토큰과 사용자 id를 매핑하여 인식, 서버에서 토큰 관리
- 토큰은 임의의 값을 넣어도 찾을 수 없도록 설계
- 해커가 토큰을 털어가도 시간이 지나면 사용 불가능하도록 만료시간을 짧게 설계

# 로그인 처리하기 - 세션 동작 방식
- 보안 문제 해결을 위해서 중요한 정보를 모두 서버에 저장
- 클라이언트와 서버는 추정 불가능한 임의의 식별자 값으로 연결

**세션 동작 방식**
**로그인**
![image](https://user-images.githubusercontent.com/96407257/162619285-2f9a0cae-c43c-4c60-afda-d399daef7385.png)
- 사용자가 loginId, password 정보를 전달하면 서버에서 해당 사용자를 확인
- 
**세션 생성**
![image](https://user-images.githubusercontent.com/96407257/162619300-1cca84dd-7e20-406a-a9fe-8837424f87e8.png)
- 세션 ID를 생성하는데, 추정 불가능하도록 설계
- UUID는 추정이 불가능
- 생성된 세션 ID와 세션에 보관할 값을 서버의 세션 저장소에 보관
- 
**세션id를 응답 쿠키로 전달**
![image](https://user-images.githubusercontent.com/96407257/162619355-ead9b85e-a74f-4a25-bd38-a61b7c41629d.png)
- 클라이언트와 서버는 쿠키로 연결
  - 서버는 클라이언트에 mySessionId라는 이름으로 세션ID만 쿠키에 담아서 전달
  - 클라이언트는 쿠키 저장소에 mySessionId쿠키를 보관
- 회원과 관련된 정보는 전혀 클라이언트에 전달 X
- 오직 추정 불간으한 세션ID만 쿠키를 통해 클라이언트로 전달

**클라이언트의 세션id 쿠키 전달**
![image](https://user-images.githubusercontent.com/96407257/162619419-ba99fb0b-dc55-4bc5-92b9-0f1f961b5086.png)
- 클라이언트는 요청 시 항상 mySessionId 쿠키를 전달
- 서버에서는 클라이언트가 전달한 mySessionId 쿠키 정보로 세션 저장소를 조회해서 로그인 시 보관한 세션 정보를 사용

# 로그인 처리하기 - 세션 직접 만들기
- 세션 생성
  - sessionId 생성(임의의 추정 불가능한 랜덤 값)
  - 세션 저장소에 sessionId와 보관할 값 저장
  - sessionId로 응답 쿠키를 생성해서 클라이언트에 전달
- 세션 조회
  - 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 값 조회
- 세션 만료
  - 클라이언트가 요청한 sessionId 쿠키의 값으로, 세션 저장소에 보관한 sessionId와 값 제거

**SessionManager - 세션 관리**
    
    @Component
    public class SessionManager {

        public static final String SESSION_COOKIE_NAME = "mySessionId";
        private Map<String, Object> sessionStore = new ConcurrentHashMap<>();

        //세션 생성
        public void createSession(Object value, HttpServletResponse response) {

            String sessionId = UUID.randomUUID().toString();
            sessionStore.put(sessionId, value);

            Cookie mySessionCookie = new Cookie(SESSION_COOKIE_NAME, sessionId);
            response.addCookie(mySessionCookie);
        }

        //세션 조회
        public Object getSession(HttpServletRequest request) {
            Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
            if (sessionCookie == null) {
                return null;
            }
            return sessionStore.get(sessionCookie.getValue());
        }

        //세션 만료
        public void expire(HttpServletRequest request) {
            Cookie sessionCookie = findCookie(request, SESSION_COOKIE_NAME);
            if (sessionCookie != null) {
                sessionStore.remove(sessionCookie.getValue());
            }
        }

        private Cookie findCookie(HttpServletRequest request, String cookieName) {
            if (request.getCookies() == null) {
                return null;
            }
            return Arrays.stream(request.getCookies())
                    .filter(cookie -> cookie.getName().equals(cookieName))
                    .findAny()
                    .orElse(null);
        }
    }
    
- @Component : 스프링 빈으로 자동 등록
- ConcurrentHashMap : HashMap은 동시 요청에 안전하지 X, 동시 요청에 안전한 ConcurrentHashMap 사용

# 로그인 처리하기 - 직접 만든 세션 적용
**LoginController - loginV2()**

    @PostMapping("/login")
    public String loginV2(@Validated @ModelAttribute LoginForm form, BindingResult bindingResult,
                        HttpServletResponse response) {
        if (bindingResult.hasErrors()) {
            return "login/loginForm";
        }

        Member loginMember = loginService.login(form.getLoginId(), form.getPassword());
        log.info("login? {}", loginMember);

        if (loginMember == null) {
            bindingResult.reject("loginFail", "아이디 또는 비밀번호가 맞지 않습니다.");
            return "login/loginForm";
        }

        //로그인 성공 처리
        sessionManager.createSession(loginMember, response);
        return "redirect:/";
    }
    
- private final SessionManager sessionManager; 주입
- sessionManager.createSession(loginMember, response); : 로그인 성공 시 세션을 등록, 세션에 loginMember 를 저장해두고, 쿠키도 함께 발행

**LoginController - logoutV2()**

    @PostMapping("/logout")
    public String logoutV2(HttpServletRequest request) {
        sessionManager.expire(request);
        return "redirect:/";
    }
    
- 로그 아웃 시 해당 세션의 정보를 제거

**HomeController - homeLoginV2()**

    @GetMapping("/")
    public String homeLoginV2(HttpServletRequest request, Model model) {

        Member member = (Member) sessionManager.getSession(request);
        if (member == null) {
            return "home";
        }

        model.addAttribute("member", member);
        return "loginHome";
    }
    
- private fianl SessionManager sessionManager; 주입
- 세션 관리자에서 저장된 회원 정보를 조회
- 회원 정보가 없을 시, 쿠키나 세션이 없는 것으로 로그인 되지 않는 것으로 처리
