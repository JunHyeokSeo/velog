<h2 id="문제-상황">문제 상황</h2>
<ul>
<li>Spring Cloud Config를 사용하는 <code>eureka-server</code> 애플리케이션을 실행했을 때, <strong>포트 8080 충돌로 서버 실행 실패</strong> 발생</li>
<li><code>application.yml</code>에서 <code>server.port=8761</code>로 명시했음에도, 실제 실행 시에는 <code>8080</code> 포트를 사용하려 시도</li>
<li>해당 포트는 이미 Config Server에서 사용 중이었음</li>
</ul>
<h2 id="환경-및-설정">환경 및 설정</h2>
<pre><code class="language-yaml"># application.yml
spring:
  application:
    name: eureka-server
  profiles:
    active: local
  config:
    import: optional:configserver:http://localhost:8080

# eureka-server-local.yml (Config Server에 존재)
server:
  port: 8761</code></pre>
<h2 id="원인-분석">원인 분석</h2>
<h3 id="1-config-server로부터-설정을-받아오기-전에-spring-context-초기화가-진행됨">[1] Config Server로부터 설정을 받아오기 전에 Spring Context 초기화가 진행됨</h3>
<ul>
<li><code>optional:</code> 설정으로 인해 config server 접속 실패 시 무시하고 application.yml 기준으로 초기화가 진행됨</li>
<li>이때 <code>server.port</code> 값이 application.yml에 명시되어 있지 않거나 미정의된 경우, <strong>기본값인 8080</strong> 포트를 사용하려고 시도</li>
<li>해당 포트는 이미 Config Server가 사용 중이므로 <strong>PortInUseException 발생</strong></li>
</ul>
<h3 id="2-config-server는-응답하지만-설정-적용-전에-context가-초기화되는-시점-차이">[2] Config Server는 응답하지만, 설정 적용 전에 context가 초기화되는 시점 차이</h3>
<ul>
<li>config server에서 값을 받아왔더라도 <code>optional:</code>로 인해 강제성이 없으므로,
Spring Context가 먼저 바인딩될 수 있음</li>
<li>이 경우 config 값을 무시한 채 application.yml의 값으로 서버가 먼저 실행되며 <strong>의도한 포트와 다르게 설정됨</strong></li>
</ul>
<h2 id="해결-방법">해결 방법</h2>
<h3 id="방법-optional-제거">방법: <code>optional:</code> 제거</h3>
<pre><code class="language-yaml">spring:
  config:
    import: configserver:http://localhost:8080</code></pre>
<ul>
<li><code>optional:</code> 제거 시 config server가 <strong>반드시 존재해야 하며</strong>, 응답 실패 시 애플리케이션은 실행되지 않음</li>
<li>덕분에 Spring Context 초기화 이전에 설정값이 제대로 바인딩됨</li>
<li>따라서 <code>server.port: 8761</code> 설정도 반영되어 충돌 없이 정상 실행됨</li>
</ul>
<h2 id="결론">결론</h2>
<ul>
<li><strong>Spring Boot 3.4</strong> 이상에서는 <code>bootstrap.yml</code> 없이도 <code>application.yml</code>만으로 config server 연동 가능</li>
<li>단, <code>optional:</code>을 제거해야 context 초기화 전에 config 값을 정확히 바인딩할 수 있음</li>
<li>이는 <strong>동기적 설정 바인딩을 강제</strong>하는 방식이며, 서버 실행 시 일관된 설정을 보장하는 방법임</li>
</ul>