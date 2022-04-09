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
