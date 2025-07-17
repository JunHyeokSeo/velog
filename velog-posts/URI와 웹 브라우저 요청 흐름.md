<h1 id="uriuniform-resource-identifier">URI(Uniform Resource Identifier)</h1>
<aside>
💡 URI = URL + URN

</aside>

<blockquote>
<p>URN은 거의 사용하지 않으므로, URI = URL로 간주한다</p>
</blockquote>
<h2 id="url-문법">URL 문법</h2>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/5314878b-dc62-4d0a-9ff8-21a6c6e233e4/image.png" /></p>
<h3 id="scheme">scheme</h3>
<p>주로 프로토콜을 명시한다</p>
<h3 id="host">host</h3>
<p>호스트명을 의미하고 도메인명 또는 IP 주소를 직접 사용할 수 있다</p>
<h3 id="port">port</h3>
<p>접속 포트</p>
<p>프로토콜 사용 시 일반적으로 생략한다(생략 시 HTTP: 80, HTTPS: 443)</p>
<h3 id="path">path</h3>
<p>리소스 경로로 계층적 구조를 갖는다</p>
<h3 id="query">query</h3>
<p>key=value 형태</p>
<p>?로 시작, &amp;로 추가 가능</p>
<p>query parameter, query string 등으로 불린다</p>
<h3 id="fragment">fragment</h3>
<p>HTML 내부 북마크 등에 사용되며 서버에 전송하는 정보는 아니다</p>
<h1 id="웹-브라우저-요청-흐름">웹 브라우저 요청 흐름</h1>
<ol>
<li>URL 입력</li>
<li>DNS 조회 및 HTTP 요청 메시지 생성</li>
<li>패킷 생성<ol>
<li>웹 브라우저가 HTTP 메시지 생성</li>
<li>SOCKET 라이브러리를 통해 전달 - TCP/IP 연결(3 way handshake)</li>
<li>TCP/IP 패킷 생성, HTTP 메시지 포함</li>
</ol>
</li>
<li>요청 패킷 전달/도착</li>
<li>응답 패킷 전달/도착</li>
<li>웹 브라우저 HTML 렌더링</li>
</ol>