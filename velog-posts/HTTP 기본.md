<h2 id="모든-것이-http">모든 것이 HTTP</h2>
<p>HTTP 메세지에 모든 것을 전송한다</p>
<ul>
<li>HTML, TEXT</li>
<li>Image, 음성, 영상, 파일</li>
<li>Json, XML</li>
<li>거의 모든 형태의 데이터 전송 가능</li>
<li>서버 간 데이터 전송 시 대부분 HTTP 사용</li>
</ul>
<h2 id="기반-프로토콜">기반 프로토콜</h2>
<aside>
💡 현재 HTTP/1.1 주로 사용, 2, 3도 점점 증가

</aside>

<p>TCP - HTTP/1.1, HTTP/2</p>
<p>UDP- HTTP/3</p>
<h1 id="http-특징">HTTP 특징</h1>
<h2 id="클라이언트-서버-구조">클라이언트-서버 구조</h2>
<aside>
💡 클라이언트는 UI, 사용성 담당

<p>서버는 Logic, Data 담당</p>
<p>⇒ 역할의 구분으로 독립적 발전이 가능했다</p>
</aside>

<p>Request/Response 구조</p>
<p>클라이언트는 서버에 요청을 보내고, 응답을 대기</p>
<p>서버가 요청에 대한 결과를 만들어서 응답한다</p>
<h2 id="무-상태stateless-프로토콜">무 상태(Stateless) 프로토콜</h2>
<aside>
💡 상태유지(Stateful): 서버가 바뀔 때, 상태 정보를 다른 서버에게 미리 알려줘야 한다

<p>무상태(Stateless): 응답 서버를 쉽게 바꿀 수 있다 ⇒ <code>무한한 서버 증설 가능</code></p>
</aside>

<p>서버가 클라이언트 상태를 보존하지 않는다</p>
<p>장점: 서버 확장성 높음, 수평 확장에 유리하다</p>
<p>단점: 클라이언트가 추가 데이터 전송</p>
<blockquote>
<p><strong>Stateless 실무 한계</strong></p>
<p>모든 것을 무상태로 설계할 수 없는 경우도 존재한다</p>
<p>로그인 했다는 상태를 서버에 유지해야 하는 경우가 그렇다</p>
<p>일반적으로 브라우저 쿠키와 서버 세션 등을 사용해서 상태를 유지한다</p>
<p><code>상태 유지는 최소한만 사용해야 한다!</code></p>
</blockquote>
<h2 id="비연결성">비연결성</h2>
<aside>
💡 연결을 유지하는 모델은 데이터 전송이 끝난 이후에도 서버 자원을 소모한다

<p>비연결성 모델은 최소한의 자원을 사용한다</p>
</aside>

<p>HTTP는 기본이 연결을 유지하지 않는 모델이다</p>
<p>1시간 동안 수천명이 서비스를 사용해도 실제 서버에서 동시에 처리하는 요청은 수십개 이하로 매우 작다</p>
<p>서버 자원을 매우 효율적으로 사용할 수 있다</p>
<blockquote>
<p><strong>한계와 극복</strong></p>
<p>TCP/IP 연결을 새로 맺어야 하므로 느리다</p>
<p>HTTP 지속 연결(Presistent Connections)로 문제를 해결했다</p>
<p>HTTP/2, /3에서 더 많은 최적화가 이루어졌다</p>
</blockquote>
<h2 id="송수신-간-http-메시지-활용">송/수신 간 HTTP 메시지 활용</h2>
<aside>
💡 HTTP 요청 메시지와 응답 메시지는 시작 라인이 다르다

</aside>

<h3 id="http-시작-라인start-line">HTTP 시작 라인(start-line)</h3>
<p><strong>요청 메시지</strong></p>
<p><code>method</code> SP(공백) <code>request-target</code> SP(공백) <code>HTTP-version</code> CRLF(개행)</p>
<ul>
<li><code>method:</code> 서버가 수행해야 할 동작 지정</li>
<li><code>request-target:</code> 절대경로 + 쿼리</li>
</ul>
<p>응답 메시지</p>
<p><code>HTTP-version</code> SP(공백) <code>status-code</code> SP(공백) <code>reason-phrase</code> CRLF(개행)</p>
<ul>
<li><code>status-code:</code> 요청 성공, 실패를 나타냄</li>
<li><code>reason-phrase:</code>사람이 이해할 수 있는 짧은 상태 코드 설명 글</li>
</ul>
<h3 id="http-헤더">HTTP 헤더</h3>
<p><code>field-name</code> <code>:</code> OWS <code>field-value</code> OWS</p>
<p>OWS: 띄어쓰기 허용(사용해도 되고 하지 않아도 된다)</p>
<p>field-name은 대소문자를 구분하지 않는다</p>
<p>HTTP 전송에 필요한 모든 부가정보를 담는다</p>
<h3 id="http-바디">HTTP 바디</h3>
<p>실제 전송할 데이터
HTML 문서, 이미지, 영상, JSON 등등 <code>byte로 표현할 수 있는 모든 데이터</code> 전송 가능</p>
<h3 id="단순함-확장-가능">단순함, 확장 가능</h3>