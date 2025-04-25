<h2 id="문제-요약">문제 요약</h2>
<ul>
<li><code>spring.cloud.config.uri</code>와 <code>spring.cloud.loadbalancer.ribbon.enabled</code>는 Spring Boot 3.4 이상에서 더 이상 권장되지 않음</li>
<li><code>bootstrap.yml</code>은 자동 로딩되지 않으며, 모든 설정은 <code>application.yml</code>에 통합 필요</li>
<li>Ribbon은 Spring Cloud에서 deprecated 되었으며, 해당 설정은 무시됨</li>
</ul>
<h2 id="기존-설정-bootstrapyml-기준">기존 설정 (bootstrap.yml 기준)</h2>
<pre><code class="language-yaml">spring:
  application:
    name: eureka-server
  profiles:
    active: local
  cloud:
    config:
      uri: http://localhost:8080
    loadbalancer:
      ribbon:
        enabled: false</code></pre>
<h2 id="변경된-설정-spring-boot-34-대응-applicationyml">변경된 설정 (Spring Boot 3.4+ 대응 application.yml)</h2>
<pre><code class="language-yaml">spring:
  application:
    name: eureka-server
  profiles:
    active: local
  config:
    import: optional:configserver:http://localhost:8080</code></pre>
<h2 id="핵심-변경-사항">핵심 변경 사항</h2>
<ul>
<li><code>spring.cloud.config.uri</code> → <code>spring.config.import</code>로 대체</li>
<li><code>ribbon.enabled</code> 제거 (더 이상 필요 없음)</li>
<li><code>application.yml</code>만 사용하고, <code>bootstrap.yml</code>은 제거</li>
</ul>
<h2 id="결론">결론</h2>
<p>Spring Boot 3.4+에서는 <code>spring.config.import</code>를 사용하여 Config Server 설정을 연결하고,<br />더 이상 사용되지 않는 Ribbon 관련 설정은 제거해야 한다.</p>