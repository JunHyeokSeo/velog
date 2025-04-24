<h2 id="현재-상황-및-문제-개요">현재 상황 및 문제 개요</h2>
<p>Spring Cloud Config Server를 통해 <code>item-service</code>의 설정 파일(<code>item-service.yaml</code>)을 내려받아 Spring Boot 애플리케이션에서 사용하려 했으나, 설정이 주입되지 않아 다음과 같은 오류가 반복적으로 발생:</p>
<pre><code>Failed to configure a DataSource: 'url' attribute is not specified and no embedded datasource could be configured.
Reason: Failed to determine a suitable driver class</code></pre><h2 id="문제-원인">문제 원인</h2>
<ul>
<li><code>item-service</code>는 <code>bootstrap.yaml</code>을 통해 Config Server에 접근하도록 설정되어 있었음.</li>
<li>하지만 Spring Boot 3.4 이상에서는 <code>bootstrap.yaml</code>이 <strong>기본적으로 로딩되지 않음</strong>.</li>
<li>따라서 <code>Config Server</code>와 연결은 시도되지 않았고, <code>spring.datasource.*</code> 설정이 비어 있어 애플리케이션 시작에 실패.</li>
</ul>
<h2 id="bootstrapyaml의-역할"><code>bootstrap.yaml</code>의 역할</h2>
<ul>
<li>Spring Cloud Config에서 외부 설정을 <strong>애플리케이션 컨텍스트가 초기화되기 전</strong>에 가져오기 위한 메커니즘.</li>
<li><code>application.yaml</code>보다 우선순위가 높은 설정 파일.</li>
<li>설정 서버 주소, 서비스 이름 등 config client 초기화 설정을 담음.</li>
</ul>
<h2 id="요즘은-왜-bootstrapyaml을-안-쓰는가">요즘은 왜 <code>bootstrap.yaml</code>을 안 쓰는가?</h2>
<ul>
<li>Spring Boot 2.4 이후부터 <code>spring.config.import=configserver:</code> 방식이 도입됨.</li>
<li>Spring Cloud 2023.x/2024.x 버전에서는 <code>bootstrap.yaml</code>은 <strong>기본 로딩 대상에서 제외</strong>됨.</li>
<li>대신 <code>application.yaml</code>에서 configserver를 직접 import 하는 방식이 <strong>공식 권장 방식</strong>이 됨.</li>
<li>더 명확한 설정 흐름과 유연한 환경 구성 가능.</li>
</ul>
<h2 id="어떻게-해결했는가">어떻게 해결했는가?</h2>
<ol>
<li><code>bootstrap.yaml</code> 삭제 또는 무시</li>
<li><code>application.yaml</code>에 아래와 같이 수정:</li>
</ol>
<pre><code class="language-yaml">spring:
  application:
    name: item-service
  config:
    import: configserver:http://localhost:8080

server:
  port: 5000</code></pre>
<ol start="3">
<li><code>spring-cloud-starter-bootstrap</code> 의존성은 제거 (더 이상 필요 없음)</li>
<li><code>config-server</code>를 먼저 실행한 후, <code>item-service</code>를 실행</li>
</ol>
<p>→ 설정이 정상 주입되며 <code>DataSource</code>, <code>JpaRepository</code>, <code>Swagger</code>, <code>Logback</code> 설정 등 모두 문제없이 동작</p>
<hr />
<h2 id="참고-로그-키워드">참고 로그 키워드</h2>
<ul>
<li><code>Fetching config from server at ...</code></li>
<li><code>Located environment: name=item-service</code></li>
<li><code>HikariPool-1 started</code></li>
</ul>