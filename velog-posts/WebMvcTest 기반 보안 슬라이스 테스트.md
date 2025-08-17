<h2 id="문제-정의-보안-필터-체인-테스트의-무거움">문제 정의: 보안 필터 체인 테스트의 무거움</h2>
<p>Spring Security의 전체 필터 체인을 통합 테스트하려면 <code>@SpringBootTest</code>와 실제 서버 구동이 필요하다. 그러나 이는 실행 속도가 느리고, 단위별로 예외 응답을 세밀하게 검증하기 어렵다. 
특히 JWT 필터와 EntryPoint, AccessDeniedHandler 등 보안 관련 구성 요소는 <strong>슬라이스 단위에서 집중적으로 검증</strong>하는 것이 효과적이다.</p>
<hr />
<h2 id="구현-전략-webmvctest-기반-슬라이스-테스트">구현 전략: WebMvcTest 기반 슬라이스 테스트</h2>
<p>나는 <code>@WebMvcTest</code>를 활용해 <strong>컨트롤러 + 보안 필터 체인만 로딩</strong>하는 테스트 환경을 구성했다.
이 방식은 애플리케이션 전체 컨텍스트를 띄우지 않고, 필요한 보안 관련 Bean만 가져와 빠르고 가벼운 테스트를 가능하게 한다.</p>
<pre><code class="language-java">@WebMvcTest
@Import({
    SecurityConfig.class,
    SecurityExceptionHandlingWebTest.MethodSecurityTestConfig.class,
    SecurityExceptionHandlingWebTest.SecurityTestStubs.class,
    SecurityExceptionHandlingWebTest.TestMvcControllers.class
})
class SecurityExceptionHandlingWebTest {
    @Resource MockMvc mockMvc;

    @MockitoBean JwtTokenResolver tokenResolver;
    @MockitoBean JwtValidator jwtValidator;
    @MockitoBean JwtAuthenticationFactory authFactory;

    @Test
    @DisplayName("만료 토큰 → 401 EXPIRED_TOKEN")
    void expired_token_should_401() throws Exception {
        String token = "expired";
        when(tokenResolver.resolve(any(HttpServletRequest.class))).thenReturn(token);
        doThrow(new CredentialsExpiredException(ErrorCode.EXPIRED_TOKEN.getMessage()))
                .when(jwtValidator).validate(token);

        mockMvc.perform(get("/secure/user"))
                .andExpect(status().isUnauthorized())
                .andExpect(jsonPath("$.error.code").value("EXPIRED_TOKEN"));
    }
}</code></pre>
<p>이 구조에서는 다음 요소들을 명시적으로 분리했다. - <code>@MockitoBean</code>을 통해 JWT 의존성(JwtTokenResolver, JwtValidator, JwtAuthenticationFactory)을 스텁 처리</p>
<ul>
<li><code>@EnableMethodSecurity</code>로 메서드 단위 권한 검증(@PreAuthorize)을 활성화</li>
<li><code>TestMvcControllers</code>를 통해 보호된 엔드포인트와 permitAll 엔드포인트를 제공</li>
</ul>
<hr />
<h2 id="근거와-공식-문서">근거와 공식 문서</h2>
<ul>
<li><a href="https://docs.spring.io/spring-security/reference/servlet/test/mockmvc.html">Spring Security Testing Reference</a>:
공식 문서에서도 <code>MockMvc</code>를 활용한 테스트를 권장한다. 이를 통해 <code>AuthenticationEntryPoint</code>, <code>AccessDeniedHandler</code> 동작을 직접 검증할 수 있다.</li>
<li><code>@WebMvcTest</code>는 컨트롤러 계층과 관련된 빈만 로드하여, 보안 필터 검증을 빠르게 실행하는 데 적합하다.</li>
</ul>
<hr />
<h2 id="내가-했던-고민과-결론">내가 했던 고민과 결론</h2>
<ul>
<li>보안 예외를 전부 통합 테스트로만 검증하려 했을 때 실행 속도가 너무 느리고, 테스트 케이스 단위로 집중 분석이 어렵다는 문제가 있었다.</li>
<li>그래서 WebMvcTest 기반 슬라이스 테스트로 전환했고, JWT 토큰 검증 로직을 스텁 처리하여 <strong>필터 동작만 집중적으로 테스트</strong>할 수 있었다.</li>
<li>이 방법으로 만료 토큰, 서명 불일치, 알고리즘 불일치 등 다양한 케이스를 빠르게 회귀 검증할 수 있었고, 보안 응답(JSON 에러 코드)이 항상 기대와 일치하는지 확인할 수 있었다.</li>
</ul>