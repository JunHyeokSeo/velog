<h2 id="문제-상황">문제 상황</h2>
<ul>
<li>Swagger UI 접근 시 500 오류 발생</li>
<li><code>/v3/api-docs</code> 호출 시 다음과 같은 에러 응답:</li>
</ul>
<pre><code># build.gradle
implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.0.3'

# Error log
NoSuchMethodError: 'void org.springframework.web.method.ControllerAdviceBean.&lt;init&gt;(java.lang.Object)'</code></pre><h2 id="원인-분석">원인 분석</h2>
<ul>
<li><code>@RestControllerAdvice</code> 클래스 등록 시 Swagger 문서 생성 과정에서 <code>ControllerAdviceBean</code> 생성자 호출</li>
<li><code>spring-web</code>의 올바른 버전(6.2.2)을 로딩 중이었음에도 런타임 충돌 발생</li>
<li>이 오류는 Springdoc OpenAPI와 Spring Boot의 <strong>버전 불일치</strong>로 발생하는 문제로 확인됨</li>
</ul>
<h2 id="해결-방법">해결 방법</h2>
<ol>
<li><code>ControllerAdviceBean</code> 클래스 로딩 경로 확인 (<code>6.2.2.jar</code>) → 정상</li>
<li><code>printConstructors()</code>로 생성자 존재 확인 → <code>(Object)</code>가 없음</li>
<li>최종적으로 <strong>Springdoc 버전이 낮아서 Spring 6을 완전히 지원하지 못함</strong>을 확인</li>
<li><code>springdoc-openapi-starter-webmvc-ui</code>를 <code>2.8.6</code>으로 업그레이드 후 오류 해결</li>
</ol>
<h2 id="결론">결론</h2>
<ul>
<li>Spring Boot 3.x 이상에서는 반드시 Springdoc 2.8.x 이상을 사용해야 함</li>
<li>Swagger 관련 오류가 발생하면 <code>/v3/api-docs</code>를 먼저 호출해 원인 파악</li>
<li><code>@ControllerAdvice</code>도 Swagger 문서 대상이 될 수 있으므로 관련 충돌 여부를 반드시 검토해야 함</li>
</ul>
<h2 id="출처">출처</h2>
<p><a href="https://stackoverflow.com/questions/78075398/trouble-configuring-springdoc-openapi-with-spring-boot-3-2-2/78487627?utm_source=chatgpt.com">https://stackoverflow.com/questions/78075398/trouble-configuring-springdoc-openapi-with-spring-boot-3-2-2/78487627?utm_source=chatgpt.com</a></p>