<h2 id="작업-개요">작업 개요</h2>
<p>Spring Cloud Config 기반 MSA 환경에서 암호화된 설정 값을 안전하게 관리하고,<br />Actuator를 활용해 설정 변경 시 HTTP 요청만으로 서비스의 설정을 갱신할 수 있도록 기능을 구현하였습니다.</p>
<hr />
<h2 id="주요-변경-사항">주요 변경 사항</h2>
<h3 id="1-비밀번호-암호화-기능-구현">1. 비밀번호 암호화 기능 구현</h3>
<ul>
<li><code>spring-security-rsa:1.0.13.RELEASE</code> 의존성 추가</li>
<li>Config Server에서 <code>/encrypt</code> 엔드포인트 활성화</li>
<li><code>encrypt.key</code> 기반 대칭키 설정으로 민감정보 암호화 처리</li>
</ul>
<pre><code class="language-yaml">encrypt:
  key: mK8@7pLr#2gTq!s9YeWdZxC1nMbQvA3h</code></pre>
<p>암호화 요청 예시:</p>
<pre><code class="language-bash">curl -X POST http://localhost:8888/encrypt \
     -H &quot;Content-Type: text/plain&quot; \
     -d 'my-plaintext-password'</code></pre>
<p>응답 예시 (실제 응답):</p>
<pre><code>AQB2QG6zgn...&lt;생략&gt;...9FQ==</code></pre><blockquote>
<p>이 응답에는 <code>{cipher}</code>가 포함되지 않으며,<br />설정 파일에 등록할 때 <strong>수동으로 <code>{cipher}</code> 접두사를 붙여야</strong> Config Server가 암호화된 값으로 인식함.</p>
</blockquote>
<pre><code class="language-yaml">spring:
  datasource:
    password: &quot;{cipher}AQB2QG6zgn...9FQ==&quot;</code></pre>
<hr />
<h3 id="2-actuator-설정-변경-및-refresh-적용">2. Actuator 설정 변경 및 Refresh 적용</h3>
<ul>
<li><code>spring-boot-starter-actuator</code> 의존성 추가</li>
<li>클라이언트 서비스(<code>item-service</code>)에서 <code>/actuator/refresh</code> 엔드포인트 노출</li>
</ul>
<pre><code class="language-yaml">#item-service/src/main/resources/application.yml
management:
  endpoints:
    web:
      exposure:
        include: refresh, health, beans</code></pre>
<blockquote>
<p><strong><code>management.endpoints.web.exposure.include</code></strong>
<code>Spring Boot Actuator</code>에서 제공하는 <code>HTTP 엔드포인트</code> 중 어떤 것을 외부에 노출할지를 정의
<code>refresh</code>: 클라이언트 설정을 다시 로드하도록 트리거하는 <code>/actuator/refresh</code> 엔드포인트 활성화
<code>health</code>: 헬스체크 정보를 확인할 수 있는 <code>/actuator/health</code> 엔드포인트 활성화
<code>beans</code>: 현재 생성된 모든 스프링 빈 정보를 확인할 수 있는 <code>/actuator/beans</code> 엔드포인트 활성화</p>
<p>기본적으로 <code>Spring Boot Actuator</code>는 거의 모든 엔드포인트를 비활성화시켜 두므로,
이렇게 명시적으로 <code>exposure.include</code>에 나열해줘야 외부에서 HTTP로 접근이 가능함.</p>
</blockquote>
<ul>
<li>설정 변경 후 <code>/actuator/refresh</code> 호출을 통해 설정 반영</li>
</ul>
<pre><code class="language-bash">curl -X POST http://localhost:5000/actuator/refresh</code></pre>
<p>응답 예시:</p>
<pre><code class="language-json">[
  &quot;spring.datasource.password&quot;
]</code></pre>
<blockquote>
<p>변경된 속성이 반영되면 이름이 배열로 반환되고, 변경이 없으면 빈 배열([])이 반환됨</p>
</blockquote>
<hr />
<h2 id="주의사항">주의사항</h2>
<ul>
<li><code>logging.file.name</code> 등 로깅 관련 설정은 <code>/refresh</code> 대상이 아니며, <strong>애플리케이션 재시작이 필요</strong></li>
<li>암호화된 설정값에는 반드시 <code>{cipher}</code> 접두사를 수동으로 붙여야 함</li>
<li>Config Server는 설정 변경을 감지하려면 재시작이 필요함 (<code>native</code> 모드 기준)</li>
</ul>
<hr />
<h1 id="spring-cloud-config-server-프로필-종류-및-native-방식-설명">Spring Cloud Config Server 프로필 종류 및 native 방식 설명</h1>
<h2 id="프로필이란">프로필이란?</h2>
<p>Spring에서는 <code>spring.profiles.active</code> 값을 통해 설정을 <strong>실행 환경에 따라 선택</strong>할 수 있습니다.<br />Spring Cloud Config에서는 이 값을 통해 설정 파일의 <strong>로딩 방식(source)</strong> 을 정의합니다.</p>
<hr />
<h2 id="native-프로필">native 프로필</h2>
<h3 id="정의">정의</h3>
<ul>
<li><code>native</code>는 설정 파일을 외부 Git이 아닌 <strong>로컬 파일 시스템(classpath 또는 디렉토리 경로)</strong> 에서 직접 읽는 방식입니다.</li>
</ul>
<h3 id="설정-예시">설정 예시</h3>
<pre><code class="language-yaml">#config-server/src/main/resources/application.yaml
spring:
  profiles:
    active: native
  cloud:
    config:
      server:
        native:
          search-locations: classpath:/config</code></pre>
<blockquote>
<p><strong><code>spring.profiles.active: native</code></strong>
<code>Spring Boot</code> 애플리케이션의 <code>활성 프로파일</code>을 <code>native</code>로 지정
이 설정은 <code>Spring Cloud Config Server</code>에서 <code>설정 파일</code>을 <code>로컬 파일 시스템에서 직접 읽는 모드</code>를 사용하게 함
즉, <code>Git</code>이나 <code>Vault</code> 같은 외부 저장소가 아닌, 프로젝트 내부의 <code>resources/config</code> 디렉토리에 있는 설정 파일을 사용하게 됨</p>
</blockquote>
<blockquote>
<p><strong><code>spring.cloud.config.server.native.search-locations</code></strong>
<code>Config Server</code>가 설정 파일을 찾을 위치를 지정
<code>classpath:/config</code>는 <code>src/main/resources/config</code> 디렉토리를 의미
이 경로에 <code>item-service.yml</code>, <code>application.yml</code> 같은 설정 파일이 위치해야 <code>Config Server</code>가 이를 클라이언트에 전달할 수 있음</p>
</blockquote>
<h3 id="특징">특징</h3>
<ul>
<li><strong>Git 없이도 설정 관리가 가능</strong></li>
<li><code>resources/config</code>에 존재하는 <code>item-service.yml</code> 등 파일을 그대로 읽어 옴</li>
<li>설정을 바꿔도 <strong>Config Server 재시작 없이는 변경을 감지하지 않음</strong></li>
<li><code>/refresh</code>는 클라이언트의 빈을 재로딩하지만, 서버가 변경된 설정을 반환하지 않으면 효과 없음</li>
</ul>
<hr />
<h2 id="관련-개념-정리">관련 개념 정리</h2>
<h3 id="config-server">Config Server</h3>
<p>Spring Cloud에서 설정 파일을 중앙 집중형으로 관리하는 서버 구성요소.<br />클라이언트 서비스는 이를 통해 설정을 가져오며, 설정 변경도 반영 가능.</p>
<h3 id="refreshscope">@RefreshScope</h3>
<p>Spring Bean에 설정값이 변경될 때 이를 동적으로 반영하게 하는 어노테이션.<br />이 어노테이션이 없으면 <code>/refresh</code> 호출에도 설정이 반영되지 않음.</p>
<h3 id="actuatorrefresh">/actuator/refresh</h3>
<p>Spring Boot Actuator가 제공하는 HTTP 엔드포인트.<br />클라이언트가 현재 로딩된 설정을 다시 읽고 필요한 Bean을 재생성하도록 트리거함.</p>
<hr />
<h2 id="요약-흐름">요약 흐름</h2>
<pre><code class="language-plaintext">[1] resources/config/item-service.yml 수정
[2] Config Server 재시작 (필수)
[3] 클라이언트에서 /actuator/refresh 호출
[4] @RefreshScope가 붙은 Bean에 새로운 설정 반영</code></pre>
<hr />
<h2 id="결론">결론</h2>
<ul>
<li><strong>로컬에서 native 모드로 설정 관리 시 Config Server 재시작은 반드시 필요</strong></li>
<li><code>/refresh</code>는 클라이언트가 변경사항을 반영할 수 있게 해주지만, 변경사항을 Config Server가 먼저 인식해야 함</li>
<li>운영환경에선 Git 프로필을 쓰는 것이 일반적이지만, 로컬 개발에서는 native가 가장 간단한 방식</li>
</ul>