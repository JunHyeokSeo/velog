<h2 id="문제-정의-필터-위치와-entrypoint-미작동-이슈">문제 정의: 필터 위치와 EntryPoint 미작동 이슈</h2>
<p>Spring Security의 필터 체인에서는 <code>ExceptionTranslationFilter</code>가 인증 및 인가 예외를 <code>AuthenticationEntryPoint</code>나 <code>AccessDeniedHandler</code>로 위임하는 핵심 역할을 한다.
JWT 인증 필터를 <code>ExceptionTranslationFilter</code> <strong>뒤</strong>에 두면, JWT 검증 실패 시 발생하는 <code>AuthenticationException</code>을 <code>EntryPoint</code>가 정상적으로 처리할 수 있다. 하지만 이 배치는 권장 아키텍처와 맞지 않는다. 
JWT는 기본적으로 <strong>사전 인증(Pre-Authentication)</strong> 단계에서 처리되어야 하기 때문에, <code>UsernamePasswordAuthenticationFilter</code> <strong>앞</strong>에 두는 것이 올바르다.</p>
<p>문제는 이렇게 올바른 위치에 두었을 때 발생한다.
<code>JwtAuthenticationFilter</code>가 <code>AuthenticationException</code>을 던져도, 해당 시점에서는 아직 <code>ExceptionTranslationFilter</code>가 동작하지 않으므로 <code>EntryPoint</code>가 호출되지 않는다. 결과적으로 클라이언트는 표준화된 인증 실패 응답을 받을 수 없다.</p>
<hr />
<h2 id="구현-전략-entrypoint-수동-호출">구현 전략: EntryPoint 수동 호출</h2>
<p>이 문제를 해결하기 위해, JWT 필터 내부에서 <code>AuthenticationException</code>을 잡아 직접 <code>entryPoint.commence(request, response, e)</code>를 호출하도록 했다.</p>
<pre><code class="language-java">} catch (AuthenticationException e) {
    log.warn("인증 실패: {}", e.getMessage());
    SecurityContextHolder.clearContext();
    entryPoint.commence(request, response, e);
    return;
} catch (Exception e) {
    log.error("JWT 필터 처리 실패 - URI={}, message={}", request.getRequestURI(), e.getMessage(), e);
    SecurityContextHolder.clearContext();
    AuthenticationException ae = new AuthenticationServiceException(
            ErrorCode.UNEXPECTED_AUTH_ERROR.getMessage(), e);
    entryPoint.commence(request, response, ae);
    return;
}</code></pre>
<p>이 로직을 통해 다음과 같은 보장이 가능하다. - JWT 검증 단계에서 발생한 모든 인증 오류는 EntryPoint로 수렴한다.</p>
<ul>
<li>SecurityContext가 남지 않도록 항상 <code>clearContext()</code>로 초기화한다.</li>
<li>예상치 못한 런타임 예외도 <code>AuthenticationServiceException</code>으로 감싸 인증 실패로 처리한다.</li>
</ul>
<hr />
<h2 id="근거와-공식-문서">근거와 공식 문서</h2>
<ul>
<li><a href="https://docs.spring.io/spring-security/reference/servlet/authentication/architecture.html#servlet-authentication-exception-translation">Spring Security Reference - Exception Handling</a>: Spring Security는 <code>ExceptionTranslationFilter</code>를 통해 인증/인가 예외를 처리한다고 명시한다. 따라서 JWT 필터가 이보다 앞에 위치한다면, 예외는 <code>EntryPoint</code>로 자동 위임되지 않는다.</li>
<li>Spring Security 권장 아키텍처: JWT는 세션 기반 로그인 인증 이후가 아닌, <strong>사전 인증 필터</strong>로 배치해야 함. 따라서 <code>UsernamePasswordAuthenticationFilter</code>보다 앞에 두는 것이 맞다.</li>
</ul>
<hr />
<h2 id="내가-했던-고민과-결론">내가 했던 고민과 결론</h2>
<ul>
<li>처음에는 JWT 필터를 <code>ExceptionTranslationFilter</code> 뒤에 두어 EntryPoint 호출이 자동으로 동작하는 것을 확인했다. 하지만 이는 <strong>권장 아키텍처 위반</strong>이라 장기적으로 유지보수 리스크가 크다고 판단했다.</li>
<li>결국 필터를 <code>UsernamePasswordAuthenticationFilter</code> 앞에 두고, 예외 발생 시 EntryPoint를 직접 호출하는 전략을 선택했다.</li>
<li>이 접근은 <strong>권장 위치를 준수</strong>하면서도 <strong>일관된 인증 실패 응답</strong>을 보장할 수 있다.</li>
</ul>