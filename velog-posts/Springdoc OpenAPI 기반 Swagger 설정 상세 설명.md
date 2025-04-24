<h2 id="1-컨트롤러의-swagger-관련-어노테이션-설명">1. 컨트롤러의 Swagger 관련 어노테이션 설명</h2>
<h3 id="openapidefinition">@OpenAPIDefinition</h3>
<ul>
<li>전체 OpenAPI 문서의 메타정보를 설정한다.</li>
<li><code>info</code> 속성을 통해 문서 제목, 설명, 버전 정보를 제공한다.</li>
<li>이 설정은 Swagger UI의 메인 페이지에서 확인할 수 있는 API 명세의 전반적인 설명 부분에 반영된다.<pre><code class="language-java">@OpenAPIDefinition(info = @Info(title = &quot;물품 처리요청 API&quot;, description = &quot;물품 처리요청 API&quot;, version = &quot;v1&quot;))</code></pre>
<h3 id="operation">@Operation</h3>
</li>
<li>하나의 API 엔드포인트(메서드)에 대한 요약 정보 및 설명을 제공한다.</li>
<li><code>summary</code>: 간단한 제목.</li>
<li><code>description</code>: 상세 설명.</li>
<li><code>tags</code>: API를 그룹으로 묶어 Swagger UI에서 구분해서 볼 수 있게 한다.<pre><code class="language-java">@Operation(summary = &quot;물품등록 요청&quot;, description = &quot;물품 등록을 진행할 수 있다.&quot;, tags = {&quot;addItem&quot;})</code></pre>
<h3 id="apiresponses--apiresponse">@ApiResponses / @ApiResponse</h3>
</li>
<li>각 HTTP 응답 코드에 따른 설명을 추가한다.</li>
<li>Swagger UI에서 사용자가 요청 후 예상 가능한 응답의 형태와 설명을 확인할 수 있게 한다.<pre><code class="language-java">@ApiResponses({
  @ApiResponse(responseCode = &quot;200&quot;, description = &quot;SUCCESS&quot;),
  @ApiResponse(responseCode = &quot;501&quot;, description = &quot;API EXCEPTION&quot;)
})</code></pre>
</li>
</ul>
<hr />
<h2 id="2-dto-클래스의-swagger-관련-어노테이션-설명">2. DTO 클래스의 Swagger 관련 어노테이션 설명</h2>
<h3 id="schema">@Schema</h3>
<ul>
<li>필드에 대한 Swagger 문서상의 메타데이터를 제공.</li>
<li><code>description</code>: 필드 설명.</li>
<li><code>example</code>: 예시 데이터.</li>
<li><code>hidden</code>: 해당 필드를 Swagger 문서에서 숨김 처리할 때 사용.<pre><code class="language-java">@Schema(description = &quot;물품ID&quot;, example = &quot;TESTID&quot;)</code></pre>
</li>
</ul>
<hr />
<h2 id="3-buildgradle의-swagger-관련-설정">3. build.gradle의 Swagger 관련 설정</h2>
<pre><code class="language-groovy">implementation 'org.springdoc:springdoc-openapi-starter-webmvc-ui:2.8.6'</code></pre>
<ul>
<li>Springdoc 라이브러리로 Spring Boot에서 OpenAPI 3.0 기반 Swagger UI를 제공.</li>
<li>이 의존성 하나로 Swagger 문서 생성과 UI 제공이 가능함.</li>
</ul>
<hr />
<h2 id="4-applicationyaml-설정-설명">4. application.yaml 설정 설명</h2>
<pre><code class="language-yaml">springdoc:
  default-consumes-media-type: application/json
  default-produces-media-type: application/json
  swagger-ui:
    operations-sorter: alpha
    tags-sorter: alpha
    path: /swagger-ui.html
    disable-swagger-default-url: true
    display-query-params-without-oauth2: true</code></pre>
<ul>
<li><code>default-consumes-media-type</code>: Swagger에서 API의 기본 요청 미디어 타입 설정 (생략 시 이 값 사용).</li>
<li><code>default-produces-media-type</code>: Swagger에서 API의 기본 응답 미디어 타입 설정.</li>
<li><code>operations-sorter</code>: 알파벳 순서로 정렬.</li>
<li><code>tags-sorter</code>: 태그도 알파벳 순으로 정렬.</li>
<li><code>path</code>: Swagger UI의 접속 URL 지정.</li>
<li><code>disable-swagger-default-url</code>: 기본 Swagger 문서 주소 비활성화.</li>
<li><code>display-query-params-without-oauth2</code>: 인증 없이 URL 쿼리 매개변수 표시 허용.</li>
</ul>
<hr />
<h2 id="5-swagger-uihtml-vs-swagger-uiindexhtml의-차이">5. /swagger-ui.html vs /swagger-ui/index.html의 차이</h2>
<h3 id="path-swagger-uihtml로-설정했는데-실제-접근은-httplocalhost8080swagger-uiindexhtml인-이유"><code>path: /swagger-ui.html</code>로 설정했는데 실제 접근은 <code>http://localhost:8080/swagger-ui/index.html</code>인 이유</h3>
<ul>
<li><code>springdoc-openapi</code> 2.x 버전부터 Swagger UI는 내부적으로 <code>swagger-ui/index.html</code>을 기준으로 동작함.</li>
<li><code>/swagger-ui.html</code>은 단순히 <code>index.html</code>로 리다이렉트하는 엔드포인트 역할만 수행한다.</li>
<li>즉, 브라우저에서 <code>/swagger-ui.html</code>로 접근하면 실제로는 <code>/swagger-ui/index.html</code>이 렌더링됨.</li>
</ul>
<hr />
<h2 id="참고-사항">참고 사항</h2>
<ul>
<li>Springfox (2.x 시대 Swagger)와 달리 Springdoc은 OpenAPI 3.0 표준을 완전히 따르며 설정이 단순하고 유지보수성이 뛰어남.</li>
</ul>