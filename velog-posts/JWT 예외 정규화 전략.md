<h2 id="문제-정의-jwt-라이브러리-예외와-spring-security-예외의-불일치">문제 정의: JWT 라이브러리 예외와 Spring Security 예외의 불일치</h2>
<p>JWT 기반 인증 로직에서는 JJWT(io.jsonwebtoken) 라이브러리를 사용하는
경우가 많다. 하지만 JJWT가 발생시키는 예외는 Spring Security의 표준 예외
계층(<code>AuthenticationException</code> 하위 클래스들)과 다르다.
예를 들어 <code>JJWT</code>는 <code>ExpiredJwtException</code>, <code>UnsupportedJwtException</code>,
<code>MalformedJwtException</code> 등을 던지지만, Spring Security는 이들을 직접
처리하지 못한다. 이 경우 EntryPoint나 AccessDeniedHandler에서 적절한
응답을 내려줄 수 없고, 결과적으로 클라이언트가 받는 응답은 불일치하거나
일관성이 떨어질 수 있다.</p>
<p>따라서 <strong>JWT 라이브러리 예외를 Spring Security 표준 예외로 변환하는
계층</strong>이 필요하다.</p>
<hr />
<h2 id="구현-전략-jwtvalidator에서의-예외-변환">구현 전략: JwtValidator에서의 예외 변환</h2>
<p><code>JwtValidator</code> 클래스는 토큰의 유효성을 검증하고, JJWT 예외를 Spring
Security 예외로 변환하는 역할을 수행한다.</p>
<pre><code class="language-java">try {
    Jws&lt;Claims&gt; parsedToken = Jwts.parserBuilder()
                                  .setSigningKey(jwtTokenProvider.getSecretKey())
                                  .setAllowedClockSkewSeconds(60)
                                  .build()
                                  .parseClaimsJws(token);
} catch (ExpiredJwtException e) {
    throw new CredentialsExpiredException(ErrorCode.EXPIRED_TOKEN.getMessage(), e);
} catch (UnsupportedJwtException e) {
    throw new BadCredentialsException(ErrorCode.UNSUPPORTED_TOKEN.getMessage(), e);
} catch (MalformedJwtException e) {
    throw new BadCredentialsException(ErrorCode.MALFORMED_TOKEN.getMessage(), e);
} catch (SecurityException e) {
    throw new BadCredentialsException(ErrorCode.INVALID_SIGNATURE.getMessage(), e);
} catch (IllegalArgumentException e) {
    throw new BadCredentialsException(ErrorCode.INVALID_TOKEN.getMessage(), e);
}</code></pre>
<p>이처럼 JWT 예외를 Spring Security 예외(<code>AuthenticationException</code>의 하위
클래스)로 매핑하면, 이후 필터 체인에서 표준화된 방식으로 처리할 수 있다.</p>
<hr />
<h2 id="entrypoint에서의-errorcode-매핑">EntryPoint에서의 ErrorCode 매핑</h2>
<p><code>CustomAuthenticationEntryPoint</code>는 <code>AuthenticationException</code>을 받아
적절한 <code>ErrorCode</code>로 변환해 클라이언트에 JSON 응답을 내려준다.</p>
<pre><code class="language-java">@Slf4j
public class CustomAuthenticationEntryPoint implements AuthenticationEntryPoint {

    @Override
    public void commence(HttpServletRequest request, HttpServletResponse response, AuthenticationException authException) throws IOException {
        ErrorCode errorCode = resolveErrorCode(authException);

        log.warn("[AuthenticationEntryPoint] 인증 실패 - URI={}, code={}, message={}",
                request.getRequestURI(), errorCode.name(), errorCode.getMessage());

        ApiResponse.writeErrorResponse(response, errorCode);
    }

    private ErrorCode resolveErrorCode(AuthenticationException ex) {
        // 스프링 시큐리티 예외
        if (ex instanceof CredentialsExpiredException)         return EXPIRED_TOKEN;
        if (ex instanceof UnsupportedAlgorithmAuthenticationException) return UNSUPPORTED_ALGORITHM;
        if (ex instanceof org.springframework.security.authentication.AuthenticationServiceException) return UNEXPECTED_AUTH_ERROR;

        // BadCredentials인 경우에만 cause 한 번 훑기
        if (ex instanceof org.springframework.security.authentication.BadCredentialsException) {
            Throwable c = ex.getCause();
            if (c instanceof io.jsonwebtoken.ExpiredJwtException)        return EXPIRED_TOKEN;
            if (c instanceof io.jsonwebtoken.UnsupportedJwtException)    return UNSUPPORTED_TOKEN;
            if (c instanceof io.jsonwebtoken.MalformedJwtException)      return MALFORMED_TOKEN;
            if (c instanceof io.jsonwebtoken.security.SecurityException
                        || c instanceof java.security.SignatureException
                        || c instanceof SecurityException)                          return INVALID_SIGNATURE;
            // 그 밖의 BadCredentials
            return INVALID_TOKEN;
        }

        return UNAUTHORIZED;
    }
}</code></pre>
<p>이 로직은 다음과 같은 흐름을 보장한다.</p>
<ol>
<li>JWT 예외 → Spring Security 예외 (JwtValidator 단계)</li>
<li>Spring Security 예외 → ErrorCode (EntryPoint 단계)</li>
<li>ErrorCode → JSON 응답 (ApiResponse 단계)</li>
</ol>