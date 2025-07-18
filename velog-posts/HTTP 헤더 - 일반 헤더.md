<h1 id="http-헤더">HTTP 헤더</h1>
<h2 id="header-field">header-field</h2>
<aside>
💡 `header-field` = `field-name` `:`OWS `field-vlaue` OWS

<p>OWS: 띄어쓰기 허용</p>
</aside>

<h2 id="http-헤더---과거rfc2616">HTTP 헤더 - 과거(RFC2616)</h2>
<h3 id="헤더-분류">헤더 분류</h3>
<blockquote>
<p><code>General 헤더:</code> 메시지 전체에 적용되는 정보, <code>예) Connection: close</code>
<code>Request 헤더:</code> 요청 정보, <code>예) User-Agent: Mozilla/5.0 (Macintosh; ..)</code>
<code>Response 헤더:</code> 응답 정보, <code>예) Server: Apache</code>
<code>Entity 헤더:</code> 엔티티 바디 정보, <code>예) Content-Type: text/html, Content-Length: 3423</code></p>
</blockquote>
<h3 id="message-body">message body</h3>
<p>message body는 entity body를 전달하는데 사용</p>
<p>entity body는 요청이나 응답에서 전달할 실제 데이터</p>
<p>entity header는 entity body의 데이터를 해석할 수 있는 정보 제공</p>
<p><img alt="" src="https://velog.velcdn.com/images/sjhgd107/post/70bfeca8-c3b1-4620-a128-28ed6c456ac4/image.png" /></p>
<h2 id="http-헤더---현재rfc7230-5">HTTP 헤더 - 현재(RFC7230-5)</h2>
<h3 id="변화">변화</h3>
<blockquote>
<p>Entity → Representation</p>
<p>Representation = Representation Metadata + Representation Data</p>
</blockquote>
<h3 id="message-body-1">message body</h3>
<aside>
💡 메시지 본문 = 페이로드(payload)

</aside>

<p>메시지 본문(message body)을 통해 표현 데이터 전달</p>
<p><code>표현</code>은 요청이나 응답에서 전달할 <code>실제 데이터</code></p>
<p>표현 헤더는 표현 데이터를 해석할 수 있는 정보 제공</p>
<ul>
<li>데이터 유형(html, json), 데이터 길이, 압축 정보 등등</li>
</ul>
<h2 id="표현">표현</h2>
<aside>
💡 왜 표현인가? resource를 HTTP로 전달할 때 다양한 형태로 `표현` 할 수 있다

<p>HTTP를 통한 값 전달을 <code>표현</code>이라 정의한 것</p>
</aside>

<h3 id="표현-헤더">표현 헤더</h3>
<aside>
💡 표현 헤더는 전송/응답 둘 다 사용

</aside>

<blockquote>
<p><code>Content-Type</code>: 표현 데이터의 형식</p>
<p><code>Content-Encoding</code>: 표현 데이터의 압축 방식</p>
<p><code>Content-Language</code>: 표현 데이터의 자연 언어</p>
<p><code>Content-Length</code>: 표현 데이터의 길이</p>
</blockquote>
<h2 id="협상">협상</h2>
<aside>
💡 클라이언트가 선호하는 표현을 요청하는 것

</aside>

<h3 id="협상-헤더">협상 헤더</h3>
<aside>
💡 협상 헤더는 요청시에만 사용

</aside>

<blockquote>
<p><code>Accept</code>: 클라이언트가 선호하는 미디어 타입 전달</p>
<p><code>Accept-Charset</code>: 클라이언트가 선호하는 문자 인코딩 </p>
<p><code>Accept-Encoding</code>: 클라이언트가 선호하는 압축 인코딩 </p>
<p><code>Accept-Language</code>: 클라이언트가 선호하는 자연 언어</p>
</blockquote>
<h3 id="협상과-우선순위">협상과 우선순위</h3>
<ol>
<li><p>Quality Values(q) 값 사용</p>
<p> 0~1 범위의 값, 클수록 높은 우선순위를 갖는다</p>
<p> 생략하면 1이 된다</p>
<p> <code>Accept-Language: ko-KR,ko;q=0.9,en-US;q=0.8,en;q=0.7</code></p>
</li>
<li><p>구체적인 것이 우선한다</p>
<p> <code>Accept: text/*, text/plain, text/plain;format=flowed, */*</code></p>
</li>
<li><p>구체적인 것을 기준으로 미디어 타입을 맞춘다
<code>Accept: text/*;q=0.3, text/html;q=0.7, text/html;level=1</code></p>
<p> <code>text/plain</code> Type이 있다고 가정하여 Quality를 매칭</p>
<p> ⇒ <code>text/plain</code>은 <code>text/*</code>에 부합되므로 0.3의 가중치를 가짐</p>
</li>
</ol>
<h2 id="전송방식">전송방식</h2>
<h3 id="방식">방식</h3>
<p>단순 전송</p>
<p>압축 전송</p>
<p>분할 전송</p>
<p>범위 전송</p>
<h3 id="단순-전송">단순 전송</h3>
<p>content-length를 알 수 있을 때 사용</p>
<p>한번에 전송하고 수신한다</p>
<h3 id="압축-전송">압축 전송</h3>
<p>압축 후 encoding 정보를 헤더에 추가하여 전송</p>
<p>크기가 절반 이하로 줄어든다</p>
<h3 id="분할-전송">분할 전송</h3>
<p><code>Transfer-Encoding</code> 헤더에 <code>chunked</code> 사용</p>
<p>전송할 Byte를 보내고, 실제 Date를 보내는 형태로 전송한다</p>
<p>분할하여 전송하고 수신측은 바로 처리한다</p>
<p>content-length를 사용해서는 안된다</p>
<h3 id="범위-전송">범위 전송</h3>
<p>중간에 전송이 끊긴 경우 처음부터 다시 송신하는 것은 비효율적이다</p>
<p>⇒ 못 받은 부분에 대해서만 전송을 요청한다 (범위를 지정하여 요청 가능)</p>
<p><code>Content-Range</code> 헤더 사용</p>
<h2 id="일반-정보">일반 정보</h2>
<blockquote>
<p><code>From</code>: 유저 에이전트의 이메일 정보
<code>Referer</code>: 이전 웹 페이지 주소
<code>User-Agent</code>: 유저 에이전트 애플리케이션 정보
<code>Server</code>: 요청을 처리하는 오리진 서버의 소프트웨어 정보
<code>Date</code>: 메시지가 생성된 날짜</p>
</blockquote>
<h2 id="특별한-정보">특별한 정보</h2>
<blockquote>
<p><code>Host</code>: 요청한 호스트 정보(도메인)
<code>Location</code>: 페이지 리다이렉션
<code>Allow</code>: 허용 가능한 HTTP 메서드
<code>Retry-After</code>: 유저 에이전트가 다음 요청을 하기까지 기다려야 하는 시간</p>
</blockquote>
<h3 id="host">Host</h3>
<p>요청에서 사용되며 필수값이다</p>
<p>하나의 서버가 여러 도메인을 처리해야할 때,</p>
<p>하나의 IP 주소에 여러 도메인이 적용되어 있을 때 사용한다</p>
<h2 id="인증">인증</h2>
<blockquote>
<p><code>Authorization</code>: 클라이언트 인증 정보를 서버에 전달</p>
<p><code>WWW-Authenticate</code>: 리소스 접근시 필요한 인증 방법 정의</p>
</blockquote>
<h3 id="authorization">Authorization</h3>
<p>인증 매커니즘마다 헤더의 값이 달라짐</p>
<p>HTTP Authorization 헤더는 인증 매커니즘과는 관계없이 헤더를 제공 ⇒ 인증과 관련된 값 삽입</p>
<h3 id="www-authenticate">WWW-Authenticate</h3>
<p>401 Unauthorized 응답과 함께 사용</p>
<p>헤더의 값을 참고하여 제대로 된 인증정보를 만들라는 의미</p>
<h2 id="쿠키">쿠키</h2>
<blockquote>
<p><code>Set-Cookie</code>: 서버에서 클라이언트로 쿠키 전달(응답)
<code>Cookie</code>: 클라이언트가 서버에서 받은 쿠키를 저장하고, HTTP 요청시 서버로 전달</p>
</blockquote>
<h3 id="쿠키-미사용">쿠키 미사용</h3>
<p>사용자는 본인의 정보를 서버가 기억하길 원한다</p>
<p>다만 HTTP는 무상태 프로토콜로 이전 요청을 기억하지 못한다</p>
<p>⇒ 모든 요청에 사용자 정보를 포함하자니 구현이 번거로울 뿐만 아니라 보안에 취약하다</p>
<h3 id="사용처">사용처</h3>
<p>사용자 로그인 세션 관리</p>
<p>광고 정보 트래킹</p>
<h3 id="특징">특징</h3>
<p>쿠키 정보는 항상 서버에 전송된다<code>(지정한 서버에 대해)</code></p>
<p>네트워크 트래픽 추가 유발</p>
<p>최소한의 정보만 사용(세션ID, 인증 토큰)</p>
<p>서버에 전송하지 않고, 웹 브라우저 내부에 데이터를 저장하고 싶다면 웹 스토리지를 참고하자</p>
<h3 id="유의사항">유의사항</h3>
<p>쿠키에 사용자 정보를 직접적으로 포함하는 것은 보안상 문제가 될 수 있다</p>
<ol>
<li>서버측에서 session을 생성하고 session ID를 보관한다</li>
<li>session ID 값이 들어있는 쿠키를 클라이언트에게 응답과 함께 보낸다</li>
<li>클라이언트가 쿠키를 보내면 서버는 쿠키에 포함된 session ID를 기반으로 사용자를 파악한다</li>
</ol>
<h3 id="생명주기">생명주기</h3>
<blockquote>
<p>Set-Cookie: <code>expires</code>=Sat, 26-Dec-2020 04:39:21 GMT</p>
<ul>
<li>GMT로 만료일 지정<code>(만료일이 되면 쿠키 삭제)</code></li>
</ul>
<p>Set-Cookie: <code>max-age</code>=3600 (3600초)</p>
<ul>
<li>초단위<code>(0이나 음수를 지정하면 쿠키 삭제)</code></li>
</ul>
<p><code>세션 쿠키</code>: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지 </p>
<p><code>영속 쿠키</code>: 만료 날짜를 입력하면 해당 날짜까지 유지</p>
</blockquote>
<h3 id="도메인">도메인</h3>
<blockquote>
<p><strong><code>명시</code> : <code>명시한 문서 기준 도메인 + 서브 도메인 포함</code></strong></p>
<ul>
<li><code>domain = example.org</code>를 <code>지정</code>해서 쿠키 생성<ul>
<li>example.org는 물론이고</li>
<li>dev.example.org도 쿠키 접근</li>
</ul>
</li>
</ul>
<p><strong><code>생략</code> : <code>현재 문서 기준 도메인만 적용</code></strong></p>
<ul>
<li>example.org 에서 쿠키를 생성하고 domain 지정을 <code>생략</code><ul>
<li>example.org 에서만 쿠키 접근</li>
<li>dev.example.org는 쿠키 미접근</li>
</ul>
</li>
</ul>
</blockquote>
<h3 id="경로path">경로(Path)</h3>
<p>지정한 경로를 포함한 하위 경로 페이지만 쿠키 접근</p>
<p>일반적으로 <code>path=/</code> 루트로 지정</p>
<h3 id="보안">보안</h3>
<blockquote>
<p><strong><code>Secure</code></strong></p>
<ul>
<li>쿠키는 http, https를 구분하지 않고 전송</li>
<li>Secure를 적용하면 https인 경우에만 전송</li>
</ul>
<p><strong><code>HttpOnly</code></strong></p>
<ul>
<li>XSS 공격 방지</li>
<li>자바스크립트에서 접근 불가(document.cookie)</li>
<li>HTTP 전송에만 사용</li>
</ul>
<p><strong><code>SameSite</code></strong></p>
<ul>
<li>XSRF 공격 방지</li>
<li>요청 도메인과 쿠키에 설정된 도메인이 같은 경우만 쿠키 전송</li>
</ul>
</blockquote>