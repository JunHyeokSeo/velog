<h2 id="배경">배경</h2>
<p>Spring Security는 인증 정보를 <code>SecurityContextHolder</code>를 통해 관리한다.<br />이때 내부적으로 <code>ThreadLocal</code>을 이용해 각 스레드마다 독립적인 인증 정보를 보관한다.
<strong>HTTP 요청마다 스레드를 새로 생성하는 것이 아니라</strong>, 재사용되는 스레드 풀에서 스레드를 꺼내 사용한다.
따라서 <code>ThreadLocal</code>에 저장된 인증 정보가 요청 처리 후 명시적으로 초기화되지 않으면, 다음 요청에 영향을 줄 수 있다.</p>
<hr />
<h2 id="발생-가능한-시나리오">발생 가능한 시나리오</h2>
<h3 id="a-사용자-요청-처리">[A 사용자 요청 처리]</h3>
<ol>
<li>A 사용자의 로그인 요청이 들어온다.</li>
<li><code>Thread-42</code>가 할당되고, 해당 스레드의 ThreadLocal에 A의 인증 객체가 저장된다.</li>
<li><code>clearContext()</code>를 호출하지 않고 요청 처리가 끝나면, A의 인증 정보는 스레드에 그대로 남아있게 된다.</li>
</ol>
<h3 id="b-사용자-요청-처리">[B 사용자 요청 처리]</h3>
<ol start="4">
<li>B 사용자의 요청이 들어오고, <code>Thread-42</code>가 재사용 된다.</li>
<li>이 스레드는 아직 A의 인증 정보를 보유하고 있다.</li>
<li>인증 설정 전에 <code>clearContext()</code>를 호출하지 않으면, B 사용자 요청에서 A의 인증 정보가 사용된다.</li>
</ol>
<hr />
<h2 id="예시">예시</h2>
<p>아래와 같이 인증 정보를 설정하기 전에 기존의 컨텍스트를 초기화하여 충돌을 방지한다.</p>
<pre><code class="language-java">public class OAuth2AuthenticationSuccessHandler implements AuthenticationSuccessHandler {

    @Override
    public void onAuthenticationSuccess(HttpServletRequest request,
                                        HttpServletResponse response,
                                        Authentication authentication) throws IOException, ServletException {
        // 기존 컨텍스트 제거
        SecurityContextHolder.clearContext();

        // 새로운 인증 정보 설정
        SecurityContext context = SecurityContextHolder.createEmptyContext();
        context.setAuthentication(authentication);
        SecurityContextHolder.setContext(context);

        // 이후 로직 진행
        ...
    }
}</code></pre>
<hr />
<h2 id="요약">요약</h2>
<p>스레드는 재사용되며, <code>ThreadLocal</code>도 초기화되지 않으면 그대로 유지된다.
인증 정보를 수동으로 설정하는 경우, 반드시 <code>SecurityContextHolder.clearContext()</code> 호출 후 설정해야 한다.
<code>Spring Security</code>는 필터 체인에서 이를 자동 처리하지만, 커스텀 로직에서는 수동 처리가 필수이다.</p>